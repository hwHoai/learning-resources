# 🏗️ Nguyên Tắc Thiết Kế Smart Contract

> Hướng dẫn chi tiết về các nguyên tắc thiết kế smart contract an toàn, hiệu quả và dễ bảo trì. Mỗi nguyên tắc được giải thích theo cấu trúc: **Pain Point → Orient → Solution → Example**.

---

## 📋 Mục lục

**Cơ bản:**

- [Cấu trúc Code Smart Contract](#-cấu-trúc-code-smart-contract)

**12 Nguyên tắc thiết kế:**

1. [Single Responsibility Principle (SRP)](#1-single-responsibility-principle-srp)
2. [Composition over Deep Inheritance](#2-composition-over-deep-inheritance)
3. [Checks-Effects-Interactions (CEI)](#3-checks-effects-interactions-cei)
4. [Pull over Push Pattern](#4-pull-over-push-pattern)
5. [Use Battle-Tested Libraries](#5-use-battle-tested-libraries)
6. [Minimize Privileges](#6-minimize-privileges)
7. [Reentrancy Protection](#7-reentrancy-protection)
8. [Explicit Visibility & Error Handling](#8-explicit-visibility--error-handling)
9. [Bounded Loops & Gas Limits](#9-bounded-loops--gas-limits)
10. [Avoid tx.origin & Validate Inputs](#10-avoid-txorigin--validate-inputs)
11. [Events & Invariant Checks](#11-events--invariant-checks)
12. [Testing & Quality Assurance](#12-testing--quality-assurance)

---

## 🏗️ Cấu trúc Code Smart Contract

Trước khi đi vào các nguyên tắc thiết kế, hãy hiểu rõ cách tổ chức code trong một smart contract. Một cấu trúc rõ ràng giúp code dễ đọc, dễ bảo trì và giảm thiểu lỗi.

### 📐 Cấu trúc chuẩn của một Contract

Smart contract nên được tổ chức theo thứ tự logic từ trên xuống dưới:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// 1️⃣ IMPORTS
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

// 2️⃣ INTERFACES
interface IToken {
    function transfer(address to, uint256 amount) external returns (bool);
}

// 3️⃣ LIBRARIES
library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        return a + b;
    }
}

// 4️⃣ CONTRACT
contract MyContract is Ownable, ReentrancyGuard {

    // ============================================
    // A. TYPE DECLARATIONS
    // ============================================
    enum Status { Pending, Active, Completed }

    struct User {
        address wallet;
        uint256 balance;
        Status status;
    }

    // ============================================
    // B. STATE VARIABLES
    // ============================================

    // B.1. Immutable (Không đổi sau deploy)
    uint256 public immutable MAX_SUPPLY;
    address public immutable TREASURY;

    // B.2. Constants (Hằng số)
    uint256 public constant FEE_DENOMINATOR = 10000;  // 100% = 10000
    uint256 public constant MAX_FEE = 1000;           // 10%

    // B.3. State variables (Có thể thay đổi)
    mapping(address => User) public users;
    address[] public userList;
    uint256 public totalSupply;
    bool public paused;

    // ============================================
    // C. CUSTOM ERRORS
    // ============================================
    error InvalidAmount();
    error InsufficientBalance(uint256 available, uint256 required);
    error Unauthorized(address caller);
    error ContractPaused();

    // ============================================
    // D. EVENTS
    // ============================================
    event UserRegistered(address indexed user, uint256 timestamp);
    event BalanceUpdated(address indexed user, uint256 oldBalance, uint256 newBalance);
    event FeeChanged(uint256 oldFee, uint256 newFee);

    // ============================================
    // E. MODIFIERS
    // ============================================
    modifier whenNotPaused() {
        if (paused) revert ContractPaused();
        _;
    }

    modifier validAddress(address _addr) {
        require(_addr != address(0), "Invalid address");
        require(_addr != address(this), "Cannot be contract address");
        _;
    }

    modifier validAmount(uint256 _amount) {
        if (_amount == 0) revert InvalidAmount();
        _;
    }

    // ============================================
    // F. CONSTRUCTOR
    // ============================================
    constructor(uint256 _maxSupply, address _treasury) Ownable(msg.sender) {
        require(_treasury != address(0), "Invalid treasury");
        MAX_SUPPLY = _maxSupply;
        TREASURY = _treasury;
    }

    // ============================================
    // G. RECEIVE & FALLBACK
    // ============================================
    receive() external payable {
        // Nhận ETH trực tiếp
        users[msg.sender].balance += msg.value;
    }

    fallback() external payable {
        // Xử lý call không khớp function signature
        revert("Fallback not allowed");
    }

    // ============================================
    // H. EXTERNAL FUNCTIONS (Public API)
    // ============================================

    /// @notice Đăng ký user mới
    /// @param _wallet Địa chỉ ví của user
    function registerUser(address _wallet)
        external
        validAddress(_wallet)
        whenNotPaused
    {
        require(users[_wallet].wallet == address(0), "User exists");

        users[_wallet] = User({
            wallet: _wallet,
            balance: 0,
            status: Status.Pending
        });

        userList.push(_wallet);

        emit UserRegistered(_wallet, block.timestamp);
    }

    /// @notice Deposit tokens vào contract
    /// @param amount Số lượng token cần deposit
    function deposit(uint256 amount)
        external
        nonReentrant
        whenNotPaused
        validAmount(amount)
    {
        User storage user = users[msg.sender];
        require(user.wallet != address(0), "User not registered");

        // Checks-Effects-Interactions pattern
        uint256 oldBalance = user.balance;
        user.balance += amount;
        totalSupply += amount;

        emit BalanceUpdated(msg.sender, oldBalance, user.balance);

        // Interaction cuối cùng
        // (giả sử có token transfer)
    }

    /// @notice Rút tokens từ contract
    /// @param amount Số lượng token cần rút
    function withdraw(uint256 amount)
        external
        nonReentrant
        whenNotPaused
        validAmount(amount)
    {
        User storage user = users[msg.sender];

        if (user.balance < amount) {
            revert InsufficientBalance(user.balance, amount);
        }

        // Checks-Effects-Interactions
        uint256 oldBalance = user.balance;
        user.balance -= amount;
        totalSupply -= amount;

        emit BalanceUpdated(msg.sender, oldBalance, user.balance);

        // Transfer ETH
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }

    // ============================================
    // I. PUBLIC FUNCTIONS
    // ============================================

    /// @notice Lấy thông tin user
    /// @param _user Địa chỉ user cần query
    /// @return User struct
    function getUserInfo(address _user) public view returns (User memory) {
        return users[_user];
    }

    /// @notice Tính phí cho một giao dịch
    /// @param amount Số tiền giao dịch
    /// @return Phí phải trả
    function calculateFee(uint256 amount) public pure returns (uint256) {
        return (amount * MAX_FEE) / FEE_DENOMINATOR;
    }

    // ============================================
    // J. INTERNAL FUNCTIONS (Helpers)
    // ============================================

    /// @dev Update status của user
    function _updateUserStatus(address _user, Status _status) internal {
        users[_user].status = _status;
    }

    /// @dev Validate transaction amount
    function _validateTransaction(uint256 amount) internal pure returns (bool) {
        return amount > 0 && amount <= type(uint256).max;
    }

    // ============================================
    // K. PRIVATE FUNCTIONS (Contract-only)
    // ============================================

    /// @dev Internal accounting logic
    function _processAccounting(address user, uint256 amount) private {
        // Logic chỉ contract này dùng
    }

    // ============================================
    // L. ADMIN FUNCTIONS (Restricted)
    // ============================================

    /// @notice Pause contract trong trường hợp khẩn cấp
    function pause() external onlyOwner {
        paused = true;
    }

    /// @notice Unpause contract
    function unpause() external onlyOwner {
        paused = false;
    }

    /// @notice Emergency withdraw (chỉ owner)
    function emergencyWithdraw() external onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }
}
```

---

### 🔍 Giải thích chi tiết từng phần

#### **1️⃣ Imports**

```solidity
import "@openzeppelin/contracts/access/Ownable.sol";
```

- Import các thư viện bên ngoài
- Đặt ở đầu file để dễ theo dõi dependencies
- Sử dụng thư viện đã kiểm chứng (OpenZeppelin)

---

#### **2️⃣ Interfaces**

```solidity
interface IToken {
    function transfer(address to, uint256 amount) external returns (bool);
}
```

- Định nghĩa các function signatures mà contract khác phải implement
- Giúp contract tương tác với nhau mà không cần biết implementation
- **Tại sao cần?** Giúp code linh hoạt, dễ test (mock), dễ upgrade

---

#### **3️⃣ Libraries**

```solidity
library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        return a + b;
    }
}
```

- Chứa logic tái sử dụng
- Không có state variables
- Tiết kiệm gas khi deploy nhiều contract dùng chung

---

#### **A. Type Declarations**

```solidity
enum Status { Pending, Active, Completed }

struct User {
    address wallet;
    uint256 balance;
    Status status;
}
```

- **Enum**: Định nghĩa tập hợp các giá trị có thể
- **Struct**: Nhóm nhiều biến liên quan thành một kiểu dữ liệu
- **Tại sao đặt đầu?** Để các phần khác có thể reference đến

---

#### **B. State Variables**

**B.1. Immutable**

```solidity
uint256 public immutable MAX_SUPPLY;
```

- Chỉ set một lần trong `constructor`
- Không thể thay đổi sau deploy
- **Khi nào dùng?** Config quan trọng không nên thay đổi (max supply, treasury address)

**B.2. Constants**

```solidity
uint256 public constant FEE_DENOMINATOR = 10000;
```

- Giá trị cố định, không bao giờ thay đổi
- Tiết kiệm gas hơn immutable (hardcoded vào bytecode)
- **Khi nào dùng?** Các hằng số toán học (100%, max fee)

**B.3. State Variables**

```solidity
mapping(address => User) public users;
uint256 public totalSupply;
```

- Có thể thay đổi trong quá trình runtime
- **Thứ tự:** Quan trọng → ít quan trọng

---

#### **C. Custom Errors**

```solidity
error InsufficientBalance(uint256 available, uint256 required);
```

- Tiết kiệm gas hơn `require` string (40%)
- Dễ debug hơn (có parameters)
- **So sánh:**

  ```solidity
  // ❌ Tốn gas
  require(balance >= amount, "Insufficient balance");

  // ✅ Tiết kiệm gas
  if (balance < amount) revert InsufficientBalance(balance, amount);
  ```

---

#### **D. Events**

```solidity
event BalanceUpdated(address indexed user, uint256 oldBalance, uint256 newBalance);
```

- Log các hành động quan trọng
- Frontend có thể lắng nghe real-time
- `indexed`: Có thể filter (tối đa 3 params)
- **Khi nào emit?** Sau khi state thay đổi thành công

---

#### **E. Modifiers**

```solidity
modifier whenNotPaused() {
    if (paused) revert ContractPaused();
    _;  // ← Vị trí thực thi function gốc
}
```

- Tái sử dụng logic kiểm tra điều kiện
- Đặt trước function signature
- **Thứ tự thực thi:** modifier1 → modifier2 → function → modifier2 → modifier1

**Ví dụ:**

```solidity
function deposit() external nonReentrant whenNotPaused validAmount(100) {
    // Thứ tự: nonReentrant → whenNotPaused → validAmount → deposit logic
}
```

---

#### **F. Constructor**

```solidity
constructor(uint256 _maxSupply, address _treasury) Ownable(msg.sender) {
    MAX_SUPPLY = _maxSupply;
    TREASURY = _treasury;
}
```

- Chỉ chạy một lần khi deploy
- Set immutable variables
- Initialize state
- **Lưu ý:** Không nên có logic phức tạp (tốn gas deploy)

---

#### **G. Receive & Fallback**

```solidity
receive() external payable {
    // Nhận ETH trực tiếp (không có data)
}

fallback() external payable {
    // Xử lý call không khớp function nào
}
```

- `receive()`: Được gọi khi có ETH gửi đến mà không có function call
- `fallback()`: Được gọi khi function signature không tồn tại
- **Khi nào cần?** Contract cần nhận ETH trực tiếp

---

#### **H. External Functions (Public API)**

```solidity
function deposit(uint256 amount) external nonReentrant whenNotPaused {
    // User gọi từ bên ngoài
}
```

- Interface mà user/contract khác tương tác
- Luôn có visibility modifier rõ ràng
- Nên có NatSpec comment (`///`)
- **Best practice:** External > Public (tiết kiệm gas cho call từ ngoài)

---

#### **I. Public Functions**

```solidity
function getUserInfo(address _user) public view returns (User memory) {
    return users[_user];
}
```

- Có thể gọi từ bên ngoài VÀ từ bên trong
- Thường là view/pure functions (không thay đổi state)
- **Khi nào dùng?** Cần gọi từ cả external và internal

---

#### **J. Internal Functions**

```solidity
function _updateUserStatus(address _user, Status _status) internal {
    users[_user].status = _status;
}
```

- Chỉ contract này và contract con (inheritance) gọi được
- Thường đặt prefix `_` (convention)
- **Khi nào dùng?** Logic helper dùng chung nhiều function

---

#### **K. Private Functions**

```solidity
function _processAccounting(address user, uint256 amount) private {
    // Chỉ contract này gọi được
}
```

- Chỉ contract này gọi được (contract con KHÔNG được)
- Tiết kiệm gas nhất (không cần thông qua inheritance)
- **Khi nào dùng?** Logic nhạy cảm, chỉ contract này dùng

---

#### **L. Admin Functions**

```solidity
function pause() external onlyOwner {
    paused = true;
}
```

- Các function đặc quyền (owner/admin only)
- Đặt cuối cùng để dễ phân biệt với user functions
- **Luôn có access control:** `onlyOwner`, `onlyRole(ADMIN_ROLE)`

---

### 📊 Tóm tắt Visibility & Function Types

| Visibility | Gọi từ bên ngoài | Gọi từ bên trong       | Contract con | Gas cost   |
| ---------- | ---------------- | ---------------------- | ------------ | ---------- |
| `external` | ✅               | ❌ (qua `this.func()`) | ❌           | Thấp nhất  |
| `public`   | ✅               | ✅                     | ✅           | Trung bình |
| `internal` | ❌               | ✅                     | ✅           | Thấp       |
| `private`  | ❌               | ✅                     | ❌           | Thấp nhất  |

**Function Types:**

- `view`: Đọc state, không thay đổi
- `pure`: Không đọc cũng không thay đổi state
- `payable`: Có thể nhận ETH
- Không modifier: Thay đổi state, không nhận ETH

---

### 🎯 Checklist Cấu trúc Code

Khi viết contract, hãy tự hỏi:

- [ ] ✅ Imports có đầy đủ không?
- [ ] ✅ Enum/Struct có đặt đầu không?
- [ ] ✅ State variables có thứ tự hợp lý không? (immutable → constant → state)
- [ ] ✅ Custom errors có tiết kiệm gas không?
- [ ] ✅ Events có `indexed` đúng không?
- [ ] ✅ Modifiers có tái sử dụng được không?
- [ ] ✅ Functions có NatSpec comment không?
- [ ] ✅ Visibility có khai báo rõ không?
- [ ] ✅ External functions có ở trên public không?
- [ ] ✅ Admin functions có access control không?

---

### 💡 Naming Conventions

```solidity
// ✅ GOOD: Đặt tên rõ ràng
uint256 public constant MAX_SUPPLY = 1000000;    // UPPER_SNAKE_CASE cho constants
uint256 public immutable deployedAt;             // camelCase cho immutable
mapping(address => uint) private _balances;      // prefix _ cho private
address[] public userList;                       // camelCase cho variables

function getUserBalance(address user) external view returns (uint256) {
    return _balances[user];                      // camelCase cho functions
}

function _internalHelper() internal {            // prefix _ cho internal
    // ...
}

event TokensMinted(address indexed to, uint256 amount);  // PascalCase cho events
error InsufficientFunds(uint256 available);              // PascalCase cho errors
```

---

**Bây giờ bạn đã hiểu cấu trúc cơ bản của một smart contract. Hãy áp dụng nó cùng với 12 nguyên tắc dưới đây để viết code an toàn và chuyên nghiệp!** 🚀

---

## 1. Single Responsibility Principle (SRP)

### 🔴 Pain Point (Nỗi đau)

Khi một smart contract cố gắng làm quá nhiều việc cùng lúc, nó trở thành một "God Contract" - một khối code khổng lồ, khó hiểu, khó test và cực kỳ nguy hiểm khi có lỗi.

**Ví dụ vấn đề:**

```solidity
// ❌ TỒI: Contract làm quá nhiều việc
contract GodContract {
    // Token logic
    mapping(address => uint) balances;

    // Staking logic
    mapping(address => uint) stakedAmount;
    mapping(address => uint) stakingTimestamp;

    // Governance logic
    mapping(address => bool) voters;
    mapping(uint => Proposal) proposals;

    // NFT logic
    mapping(uint => address) nftOwners;

    // Lending logic
    mapping(address => uint) borrowed;

    // ... 500+ dòng code hỗn loạn
}
```

**Hậu quả:**

- 🐛 Một bug nhỏ ở phần staking có thể ảnh hưởng đến toàn bộ hệ thống
- 🧪 Không thể test riêng từng chức năng
- 🔄 Upgrade một tính năng phải deploy lại toàn bộ
- 📖 Developer mới không thể hiểu code
- ⛽ Gas cost cực cao vì contract size lớn

---

### 🧭 Orient (Định hướng)

Mỗi contract chỉ nên có **một lý do duy nhất để thay đổi**. Nếu bạn phải sửa contract vì nhiều lý do khác nhau (fix bug token, thêm tính năng staking, sửa governance), thì contract đó đang vi phạm SRP.

**Nguyên tắc vàng:**

> "Một contract, một trách nhiệm, một mục đích"

---

### ✅ Solution (Giải pháp)

Tách contract lớn thành nhiều contract nhỏ, mỗi contract chịu trách nhiệm cho một domain cụ thể.

**Cấu trúc đề xuất:**

```
MyDApp/
├── Token.sol           # Chỉ quản lý token logic
├── Staking.sol         # Chỉ quản lý staking
├── Governance.sol      # Chỉ quản lý voting
├── Treasury.sol        # Chỉ quản lý funds
└── Facade.sol          # Điều phối các contract
```

---

### 💡 Example (Ví dụ & Khi nào dùng)

#### ✅ Code tốt: Tách biệt trách nhiệm

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// ============================================
// 1. Contract chỉ quản lý TOKEN
// ============================================
contract MyToken {
    mapping(address => uint256) private _balances;
    uint256 private _totalSupply;

    event Transfer(address indexed from, address indexed to, uint256 value);

    function balanceOf(address account) external view returns (uint256) {
        return _balances[account];
    }

    function transfer(address to, uint256 amount) external returns (bool) {
        _transfer(msg.sender, to, amount);
        return true;
    }

    function _transfer(address from, address to, uint256 amount) internal {
        require(_balances[from] >= amount, "Insufficient balance");
        _balances[from] -= amount;
        _balances[to] += amount;
        emit Transfer(from, to, amount);
    }
}

// ============================================
// 2. Contract chỉ quản lý STAKING
// ============================================
contract Staking {
    MyToken public token;

    mapping(address => uint256) public stakedAmount;
    mapping(address => uint256) public stakingTimestamp;

    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount, uint256 reward);

    constructor(address _token) {
        token = MyToken(_token);
    }

    function stake(uint256 amount) external {
        require(amount > 0, "Cannot stake 0");
        require(token.transferFrom(msg.sender, address(this), amount), "Transfer failed");

        stakedAmount[msg.sender] += amount;
        stakingTimestamp[msg.sender] = block.timestamp;

        emit Staked(msg.sender, amount);
    }

    function calculateReward(address user) public view returns (uint256) {
        uint256 stakedTime = block.timestamp - stakingTimestamp[user];
        // Ví dụ: 10% APR
        return (stakedAmount[user] * stakedTime * 10) / (365 days * 100);
    }

    function unstake() external {
        uint256 amount = stakedAmount[msg.sender];
        require(amount > 0, "No staked amount");

        uint256 reward = calculateReward(msg.sender);

        stakedAmount[msg.sender] = 0;
        stakingTimestamp[msg.sender] = 0;

        require(token.transfer(msg.sender, amount + reward), "Transfer failed");

        emit Unstaked(msg.sender, amount, reward);
    }
}

// ============================================
// 3. Contract điều phối (Facade)
// ============================================
contract MyDAppFacade {
    MyToken public token;
    Staking public staking;

    constructor(address _token, address _staking) {
        token = MyToken(_token);
        staking = Staking(_staking);
    }

    // Người dùng tương tác qua Facade
    function getUserInfo(address user) external view returns (
        uint256 balance,
        uint256 staked,
        uint256 pendingReward
    ) {
        balance = token.balanceOf(user);
        staked = staking.stakedAmount(user);
        pendingReward = staking.calculateReward(user);
    }
}
```

---

#### 🎯 Khi nào áp dụng SRP?

| Tình huống                     | Nên làm                       |
| ------------------------------ | ----------------------------- |
| **Contract > 300 dòng**        | Tách thành nhiều contract nhỏ |
| **Nhiều loại state variables** | Mỗi loại một contract riêng   |
| **Logic không liên quan**      | Tách thành module độc lập     |
| **Cần upgrade một phần**       | Tách để upgrade riêng lẻ      |
| **Team lớn phát triển**        | Mỗi người một contract        |

---

#### ✨ Lợi ích thực tế

```solidity
// ✅ Dễ test
contract StakingTest {
    function testStaking() public {
        // Chỉ cần mock MyToken
        // Không phải lo về Governance, NFT, etc.
        MyToken mockToken = new MyToken();
        Staking staking = new Staking(address(mockToken));

        // Test logic staking một cách độc lập
        staking.stake(100);
        assertEq(staking.stakedAmount(address(this)), 100);
    }
}

// ✅ Dễ upgrade
contract StakingV2 {
    // Nâng cấp staking logic mà không ảnh hưởng Token contract
    // Token contract vẫn giữ nguyên address và state
}

// ✅ Dễ audit
// Auditor chỉ cần focus vào Staking.sol (200 dòng)
// Thay vì phải đọc GodContract (2000 dòng)
```

---

## 2. Composition over Deep Inheritance

### 🔴 Pain Point (Nỗi đau)

Kế thừa sâu (deep inheritance) trong Solidity tạo ra một "cây gia phả" phức tạp, khó theo dõi và dễ gây lỗi không mong muốn.

**Ví dụ vấn đề:**

```solidity
// ❌ TỒI: Kế thừa quá sâu
contract A {
    uint public x;
    function setX(uint _x) public { x = _x; }
}

contract B is A {
    uint public y;
    function setY(uint _y) public { y = _y; }
}

contract C is B {
    uint public z;
    function setZ(uint _z) public { z = _z; }
}

contract D is C {
    uint public w;
    // ??? x, y, z, w đều ở đây nhưng không rõ từ đâu
    // ??? Constructor nào được gọi trước?
    // ??? Override function nào ghi đè function nào?
}
```

**Hậu quả:**

- 🔀 Storage layout phức tạp, dễ conflict
- 🐛 Bug ẩn từ contract cha xa xôi
- 📚 Phải đọc 4-5 file để hiểu một hàm
- ⛽ Gas cao vì nhiều lớp kế thừa
- 🔒 Khó kiểm soát quyền truy cập

---

### 🧭 Orient (Định hướng)

Thay vì xây dựng một "tháp" kế thừa cao chọc trời, hãy xây dựng nhiều "khối" nhỏ độc lập và ghép chúng lại. Đây là tư duy **"Lego"**: nhiều mảnh nhỏ có thể tháo lắp linh hoạt.

**Nguyên tắc vàng:**

> "Hợp thành (Composition) linh hoạt hơn kế thừa (Inheritance)"

---

### ✅ Solution (Giải pháp)

Sử dụng ba công cụ thay thế kế thừa:

1. **Libraries**: Chứa logic tái sử dụng
2. **Interfaces**: Định nghĩa contract chuẩn
3. **Contract References**: Tham chiếu đến contract khác

---

### 💡 Example (Ví dụ & Khi nào dùng)

#### ❌ Code tồi: Kế thừa sâu

```solidity
// File: Ownable.sol
contract Ownable {
    address public owner;
    constructor() { owner = msg.sender; }
    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }
}

// File: Pausable.sol
contract Pausable is Ownable {
    bool public paused;
    modifier whenNotPaused() {
        require(!paused);
        _;
    }
    function pause() public onlyOwner { paused = true; }
}

// File: TokenBase.sol
contract TokenBase is Pausable {
    mapping(address => uint) public balances;
    function _transfer(address to, uint amount) internal whenNotPaused {
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}

// File: MyToken.sol
contract MyToken is TokenBase {
    // Kế thừa từ 3 lớp: Ownable -> Pausable -> TokenBase
    // Phải hiểu tất cả 3 contract cha để biết MyToken làm gì
    string public name = "MyToken";
}
```

---

#### ✅ Code tốt: Composition với Libraries & Interfaces

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// ============================================
// 1. LIBRARY: Logic tái sử dụng
// ============================================
library SafeMath {
    function add(uint a, uint b) internal pure returns (uint) {
        uint c = a + b;
        require(c >= a, "Overflow");
        return c;
    }

    function sub(uint a, uint b) internal pure returns (uint) {
        require(b <= a, "Underflow");
        return a - b;
    }
}

// ============================================
// 2. INTERFACE: Định nghĩa chuẩn
// ============================================
interface IAccessControl {
    function isOwner(address account) external view returns (bool);
    function isAdmin(address account) external view returns (bool);
}

interface IPausable {
    function isPaused() external view returns (bool);
    function pause() external;
    function unpause() external;
}

interface IToken {
    function balanceOf(address account) external view returns (uint);
    function transfer(address to, uint amount) external returns (bool);
}

// ============================================
// 3. MODULE: Contract nhỏ, độc lập
// ============================================
contract AccessControl is IAccessControl {
    address public owner;
    mapping(address => bool) public admins;

    constructor() {
        owner = msg.sender;
    }

    function isOwner(address account) external view returns (bool) {
        return account == owner;
    }

    function isAdmin(address account) external view returns (bool) {
        return admins[account];
    }

    function addAdmin(address admin) external {
        require(msg.sender == owner, "Not owner");
        admins[admin] = true;
    }
}

contract PausableModule is IPausable {
    bool private _paused;
    IAccessControl private _accessControl;

    constructor(address accessControl) {
        _accessControl = IAccessControl(accessControl);
    }

    function isPaused() external view returns (bool) {
        return _paused;
    }

    function pause() external {
        require(_accessControl.isOwner(msg.sender), "Not authorized");
        _paused = true;
    }

    function unpause() external {
        require(_accessControl.isOwner(msg.sender), "Not authorized");
        _paused = false;
    }
}

// ============================================
// 4. MAIN CONTRACT: Hợp thành các module
// ============================================
contract MyToken is IToken {
    using SafeMath for uint256;

    // Dependencies (Composition)
    IAccessControl private accessControl;
    IPausable private pausable;

    mapping(address => uint256) private _balances;
    uint256 private _totalSupply;

    event Transfer(address indexed from, address indexed to, uint256 value);

    constructor(address _accessControl, address _pausable) {
        accessControl = IAccessControl(_accessControl);
        pausable = IPausable(_pausable);
        _totalSupply = 1000000 * 10**18;
        _balances[msg.sender] = _totalSupply;
    }

    function balanceOf(address account) external view returns (uint256) {
        return _balances[account];
    }

    function transfer(address to, uint256 amount) external returns (bool) {
        require(!pausable.isPaused(), "Paused");
        require(to != address(0), "Invalid recipient");

        _balances[msg.sender] = _balances[msg.sender].sub(amount);
        _balances[to] = _balances[to].add(amount);

        emit Transfer(msg.sender, to, amount);
        return true;
    }

    // Admin function
    function mint(address to, uint256 amount) external {
        require(accessControl.isAdmin(msg.sender), "Not admin");
        require(!pausable.isPaused(), "Paused");

        _totalSupply = _totalSupply.add(amount);
        _balances[to] = _balances[to].add(amount);

        emit Transfer(address(0), to, amount);
    }
}
```

---

#### 🎯 So sánh Inheritance vs Composition

| Tiêu chí           | Inheritance (Kế thừa)        | Composition (Hợp thành)      |
| ------------------ | ---------------------------- | ---------------------------- |
| **Độ phức tạp**    | ❌ Cao (phải đọc nhiều file) | ✅ Thấp (mỗi module độc lập) |
| **Testing**        | ❌ Khó (phụ thuộc lẫn nhau)  | ✅ Dễ (mock interface)       |
| **Upgrade**        | ❌ Phải deploy lại tất cả    | ✅ Chỉ thay contract con     |
| **Reusability**    | ❌ Cứng nhắc                 | ✅ Linh hoạt                 |
| **Gas cost**       | ❌ Cao hơn                   | ✅ Tối ưu hơn                |
| **Storage layout** | ❌ Dễ conflict               | ✅ Tách biệt rõ ràng         |

---

#### 🧪 Testing với Composition

```solidity
// ✅ Dễ dàng mock các dependency
contract MyTokenTest {
    function testTransferWhenPaused() public {
        // Mock AccessControl
        MockAccessControl accessControl = new MockAccessControl();

        // Mock Pausable (set paused = true)
        MockPausable pausable = new MockPausable();
        pausable.setPaused(true);

        // Test MyToken với mock objects
        MyToken token = new MyToken(
            address(accessControl),
            address(pausable)
        );

        // Expect revert vì paused
        vm.expectRevert("Paused");
        token.transfer(address(1), 100);
    }
}

contract MockPausable is IPausable {
    bool public paused;
    function isPaused() external view returns (bool) { return paused; }
    function pause() external { paused = true; }
    function unpause() external { paused = false; }
    function setPaused(bool _paused) external { paused = _paused; }
}
```

---

#### 📐 Kiến trúc đề xuất: Facade Pattern

```
                    ┌─────────────┐
                    │   Facade    │ ← User tương tác
                    │  (Main UI)  │
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
     ┌────▼────┐     ┌────▼────┐     ┌────▼────┐
     │ Module  │     │ Module  │     │ Module  │
     │  Token  │     │ Staking │     │  Access │
     └─────────┘     └─────────┘     └─────────┘
```

---

#### 🎯 Khi nào dùng Composition?

| Tình huống               | Giải pháp                                |
| ------------------------ | ---------------------------------------- |
| **Logic tái sử dụng**    | Dùng Library                             |
| **Cần upgrade**          | Dùng Interface + Contract References     |
| **Nhiều implementation** | Dùng Interface (ví dụ: nhiều loại token) |
| **Tách quyền hạn**       | Dùng AccessControl module riêng          |
| **Testing phức tạp**     | Dùng Composition để mock dễ dàng         |

---

## 3. Checks-Effects-Interactions (CEI)

### 🔴 Pain Point (Nỗi đau)

**Lỗ hổng Re-entrancy** - một trong những lỗ hổng nguy hiểm nhất trong smart contract. Khi contract của bạn gọi một contract khác, contract đó có thể gọi lại (re-enter) vào contract của bạn **trước khi** hàm gốc thực thi xong.

**Ví dụ tấn công thực tế (The DAO Hack - $60M):**

```solidity
// ❌ Contract BỊ TẤN CÔNG
contract VulnerableBank {
    mapping(address => uint) public balances;

    function withdraw(uint amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // ⚠️ NGUY HIỂM: Gửi ETH TRƯỚC khi update state
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");

        // ❌ Dòng này không bao giờ được thực thi trong lần gọi lại
        balances[msg.sender] -= amount;
    }

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }
}

// 💀 Contract TẤN CÔNG
contract Attacker {
    VulnerableBank public bank;
    uint public count;

    constructor(address _bank) {
        bank = VulnerableBank(_bank);
    }

    // Bước 1: Deposit 1 ETH
    function attack() external payable {
        require(msg.value == 1 ether);
        bank.deposit{value: 1 ether}();
        bank.withdraw(1 ether);
    }

    // Bước 2: Receive được gọi khi nhận ETH
    receive() external payable {
        count++;
        if (count < 10) {
            // ⚡ Gọi lại withdraw() trước khi balance được trừ
            bank.withdraw(1 ether);
        }
    }
}
```

**Kịch bản tấn công:**

```
1. Attacker deposit 1 ETH vào bank
2. Attacker gọi withdraw(1 ETH)
3. Bank gửi 1 ETH cho Attacker → Kích hoạt receive()
4. receive() gọi lại withdraw(1 ETH) (balance vẫn là 1 ETH!)
5. Bank lại gửi 1 ETH → receive() lại được gọi
6. Lặp lại 10 lần → Attacker rút được 10 ETH chỉ với 1 ETH ban đầu
7. Bank bị hút cạn tiền
```

---

### 🧭 Orient (Định hướng)

Luôn hoàn thành **tất cả công việc nội bộ** trước khi tương tác với bất kỳ contract nào bên ngoài. Hãy coi việc gọi một contract khác như việc "mở cửa cho kẻ lạ vào nhà" - bạn phải dọn dẹp, khóa két tiền trước.

**Nguyên tắc vàng:**

> "Kiểm tra → Thay đổi → Tương tác"

---

### ✅ Solution (Giải pháp)

Tuân thủ nghiêm ngặt thứ tự CEI:

1. **Checks**: Kiểm tra tất cả điều kiện (`require`, `if`)
2. **Effects**: Cập nhật tất cả state variables
3. **Interactions**: Gọi external contract hoặc gửi ETH

---

### 💡 Example (Ví dụ & Khi nào dùng)

#### ✅ Code an toàn: Tuân thủ CEI Pattern

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SecureBank {
    mapping(address => uint256) public balances;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    // ============================================
    // ✅ HÀM AN TOÀN: Tuân thủ CEI
    // ============================================
    function withdraw(uint256 amount) external {
        // 1️⃣ CHECKS: Kiểm tra tất cả điều kiện
        uint256 userBalance = balances[msg.sender];
        require(userBalance >= amount, "Insufficient balance");
        require(amount > 0, "Amount must be > 0");

        // 2️⃣ EFFECTS: Cập nhật state TRƯỚC
        balances[msg.sender] = userBalance - amount;

        // 3️⃣ INTERACTIONS: Gửi ETH SAU
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");

        emit Withdrawn(msg.sender, amount);
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");

        // CEI: Effect trước, event sau
        balances[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    // ============================================
    // ✅ ALTERNATIVE: Sử dụng ReentrancyGuard
    // ============================================
    bool private locked;

    modifier nonReentrant() {
        require(!locked, "ReentrancyGuard: reentrant call");
        locked = true;
        _;
        locked = false;
    }

    function withdrawWithGuard(uint256 amount) external nonReentrant {
        require(balances[msg.sender] >= amount, "Insufficient balance");

        balances[msg.sender] -= amount;

        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");

        emit Withdrawn(msg.sender, amount);
    }
}
```

---

#### 🔍 Phân tích chi tiết: Tại sao CEI hoạt động?

```solidity
// ❌ TRƯỚC: Balance chưa được update
balances[msg.sender] = 100 ETH  // ← Attacker có thể exploit
(bool success, ) = msg.sender.call{value: amount}("");  // ← Re-enter tại đây
balances[msg.sender] -= amount;  // ← Không bao giờ chạy trong re-enter

// ✅ SAU: Balance đã được update
balances[msg.sender] = 100 ETH
balances[msg.sender] -= amount;  // ← Update TRƯỚC = 0 ETH
(bool success, ) = msg.sender.call{value: amount}("");  // ← Re-enter
// Nếu re-enter, balance = 0 → require fail → An toàn
```

---

#### 🧪 Test Re-entrancy Attack

```solidity
contract ReentrancyTest {
    SecureBank public bank;

    function testReentrancyProtection() public {
        bank = new SecureBank();

        // Attacker deposit 1 ETH
        bank.deposit{value: 1 ether}();

        uint256 bankBalanceBefore = address(bank).balance;
        uint256 attackerBalanceBefore = address(this).balance;

        // Attacker thử tấn công
        bank.withdraw(1 ether);

        // Kiểm tra: Attacker chỉ nhận được 1 ETH (không phải 10 ETH)
        assertEq(address(this).balance, attackerBalanceBefore + 1 ether);
        assertEq(address(bank).balance, bankBalanceBefore - 1 ether);
    }

    receive() external payable {
        // Thử re-enter
        if (address(bank).balance > 0) {
            bank.withdraw(1 ether);  // ← Sẽ FAIL vì balance = 0
        }
    }
}
```

---

#### 🛡️ Các cách bảo vệ khác

```solidity
// CÁCH 1: OpenZeppelin ReentrancyGuard
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract BankWithOZ is ReentrancyGuard {
    function withdraw(uint amount) external nonReentrant {
        // ... logic
    }
}

// CÁCH 2: Pull over Push (Xem phần 4)
contract BankWithPull {
    mapping(address => uint) public withdrawable;

    function requestWithdraw(uint amount) external {
        require(balances[msg.sender] >= amount);
        balances[msg.sender] -= amount;
        withdrawable[msg.sender] += amount;  // Ghi nhận
    }

    function claimWithdraw() external {
        uint amount = withdrawable[msg.sender];
        withdrawable[msg.sender] = 0;
        payable(msg.sender).transfer(amount);  // User tự rút
    }
}

// CÁCH 3: Checks-Effects-Interactions + Transfer (gas limit 2300)
function withdrawSafe(uint amount) external {
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount;
    payable(msg.sender).transfer(amount);  // transfer() chỉ cho 2300 gas
    // ⚠️ Nhưng không khuyến khích vì 2300 gas có thể thay đổi trong future
}
```

---

#### 🎯 Checklist áp dụng CEI

| Bước                | Câu hỏi                          | Hành động                                 |
| ------------------- | -------------------------------- | ----------------------------------------- |
| **1. Checks**       | Đã kiểm tra tất cả input?        | `require(balances[msg.sender] >= amount)` |
|                     | Đã kiểm tra điều kiện nghiệp vụ? | `require(amount > 0)`                     |
|                     | Đã kiểm tra quyền?               | `require(msg.sender == owner)`            |
| **2. Effects**      | Đã update tất cả state?          | `balances[msg.sender] -= amount`          |
|                     | Đã emit event?                   | `emit Withdrawn(msg.sender, amount)`      |
| **3. Interactions** | Đã gửi ETH hoặc gọi external?    | `msg.sender.call{value: amount}("")`      |

---

#### ⚠️ Các lỗi phổ biến

```solidity
// ❌ LỖI 1: Emit event sau interaction
function withdrawBad1(uint amount) external {
    balances[msg.sender] -= amount;
    (bool success, ) = msg.sender.call{value: amount}("");  // ← Interaction
    emit Withdrawn(msg.sender, amount);  // ← Event có thể không được emit nếu revert
}

// ❌ LỖI 2: Update state trong nhiều bước
function withdrawBad2(uint amount) external {
    balances[msg.sender] -= amount;  // ← Effect 1
    (bool success, ) = msg.sender.call{value: amount}("");  // ← Interaction
    totalWithdrawn += amount;  // ← Effect 2 (quá muộn!)
}

// ❌ LỖI 3: Kiểm tra sau interaction
function withdrawBad3(uint amount) external {
    (bool success, ) = msg.sender.call{value: amount}("");  // ← Interaction
    require(success, "Failed");  // ← Check (quá muộn!)
    balances[msg.sender] -= amount;
}

// ✅ ĐÚNG: CEI nghiêm ngặt
function withdrawGood(uint amount) external {
    // 1. Checks
    require(balances[msg.sender] >= amount);
    require(amount > 0);

    // 2. Effects
    balances[msg.sender] -= amount;
    totalWithdrawn += amount;
    emit Withdrawn(msg.sender, amount);

    // 3. Interactions
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed");
}
```

---

#### 🎯 Khi nào áp dụng CEI?

| Tình huống                | Bắt buộc CEI               |
| ------------------------- | -------------------------- |
| **Gửi ETH**               | ✅ Bắt buộc                |
| **Gọi external contract** | ✅ Bắt buộc                |
| **Delegate call**         | ✅ Bắt buộc                |
| **Chỉ đọc data**          | ❌ Không cần (view/pure)   |
| **Internal function**     | ⚠️ Nên làm (best practice) |

---

## 📚 Tổng kết 3 Nguyên tắc Đầu tiên

### 🎯 Quick Reference

| Nguyên tắc      | Mục đích                      | Khi nào dùng            |
| --------------- | ----------------------------- | ----------------------- |
| **SRP**         | Một contract, một trách nhiệm | Mọi project             |
| **Composition** | Ghép module thay vì kế thừa   | Contract phức tạp       |
| **CEI**         | Chống re-entrancy             | Có gửi ETH/gọi external |

---

### ✅ Checklist thiết kế contract

```solidity
// ✅ Template contract an toàn
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureContract is ReentrancyGuard {
    // 1. SRP: Contract chỉ làm MỘT việc
    // Ví dụ: Chỉ quản lý staking

    // 2. Composition: Sử dụng interface
    IAccessControl public accessControl;
    IToken public token;

    constructor(address _accessControl, address _token) {
        accessControl = IAccessControl(_accessControl);
        token = IToken(_token);
    }

    // 3. CEI Pattern trong mọi hàm
    function stake(uint amount) external nonReentrant {
        // 1. Checks
        require(amount > 0, "Invalid amount");
        require(token.balanceOf(msg.sender) >= amount, "Insufficient balance");

        // 2. Effects
        stakedAmount[msg.sender] += amount;
        totalStaked += amount;
        emit Staked(msg.sender, amount);

        // 3. Interactions
        require(token.transferFrom(msg.sender, address(this), amount), "Transfer failed");
    }
}
```

---

## 4. Pull over Push Pattern

### 🔴 Pain Point (Nỗi đau)

Khi một contract "đẩy" (push) ETH đến nhiều địa chỉ cùng lúc, một địa chỉ nhận bị lỗi hoặc cố tình gây lỗi có thể làm **toàn bộ giao dịch thất bại** (Denial of Service). Hơn nữa, việc đẩy tiền trực tiếp làm tăng nguy cơ re-entrancy.

**Ví dụ vấn đề:**

```solidity
// ❌ TỒI: Push payments (Đẩy tiền)
contract VulnerableAuction {
    address[] public bidders;
    mapping(address => uint) public bids;

    function endAuction() public {
        // Hoàn tiền cho tất cả người thua
        for (uint i = 0; i < bidders.length; i++) {
            address bidder = bidders[i];
            if (bidder != highestBidder) {
                // ⚠️ NGUY HIỂM: Nếu một người fail, tất cả fail
                payable(bidder).transfer(bids[bidder]);
            }
        }
    }
}

// 💀 Attacker có thể DoS
contract MaliciousReceiver {
    // Reject mọi ETH → endAuction() sẽ fail mãi mãi
    receive() external payable {
        revert("I don't want money!");
    }
}
```

**Hậu quả:**

- 🚫 Một user ác ý chặn toàn bộ hệ thống
- ⛽ Gas cost cao khi phải gửi cho nhiều người
- 🔄 Phải retry nhiều lần nếu fail
- 🐛 Khó debug vì không biết address nào gây lỗi

---

### 🧭 Orient (Định hướng)

Thay vì contract chủ động đi "phát tiền" (push), hãy biến nó thành một "ngân hàng" thụ động. Contract chỉ **ghi nhận số tiền mỗi người được hưởng**. Người dùng phải tự chịu trách nhiệm đến và **"kéo" (pull)** tiền của họ về.

**Nguyên tắc vàng:**

> "Người nhận tự rút, contract không tự đẩy"

---

### ✅ Solution (Giải pháp)

1. Contract không trực tiếp gửi tiền đi
2. Cập nhật `mapping` để ghi nhận số tiền người dùng được nhận
3. Cung cấp hàm `withdraw()` hoặc `claim()` để user tự rút

---

### 💡 Example (Ví dụ & Khi nào dùng)

#### ✅ Code an toàn: Pull Payment Pattern

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// ============================================
// ✅ AUCTION với Pull Payment
// ============================================
contract SecureAuction {
    address public highestBidder;
    uint256 public highestBid;

    // Pull pattern: Ghi nhận số tiền người dùng có thể rút
    mapping(address => uint256) public pendingReturns;

    bool public ended;

    event BidPlaced(address indexed bidder, uint256 amount);
    event AuctionEnded(address winner, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    // ============================================
    // Đặt giá thầu
    // ============================================
    function bid() external payable {
        require(!ended, "Auction ended");
        require(msg.value > highestBid, "Bid too low");

        // Nếu có người trả giá cao trước đó
        if (highestBidder != address(0)) {
            // ✅ KHÔNG gửi tiền ngay, chỉ GHI NHẬN
            pendingReturns[highestBidder] += highestBid;
        }

        highestBidder = msg.sender;
        highestBid = msg.value;

        emit BidPlaced(msg.sender, msg.value);
    }

    // ============================================
    // Người dùng TỰ RÚT tiền
    // ============================================
    function withdraw() external {
        uint256 amount = pendingReturns[msg.sender];
        require(amount > 0, "No funds to withdraw");

        // CEI pattern
        pendingReturns[msg.sender] = 0;

        (bool success, ) = payable(msg.sender).call{value: amount}("");
        require(success, "Withdraw failed");

        emit Withdrawn(msg.sender, amount);
    }

    // ============================================
    // Kết thúc đấu giá
    // ============================================
    function endAuction() external {
        require(!ended, "Already ended");

        ended = true;
        emit AuctionEnded(highestBidder, highestBid);

        // ✅ KHÔNG cần loop gửi tiền cho mọi người
        // Họ tự rút qua withdraw()
    }

    // ============================================
    // Helper: Check số tiền có thể rút
    // ============================================
    function checkPendingReturn(address user) external view returns (uint256) {
        return pendingReturns[user];
    }
}
```

---

#### 🔍 So sánh Push vs Pull

```solidity
// ❌ PUSH: Contract chủ động gửi tiền
contract PushPayment {
    function distributeRewards(address[] memory users, uint[] memory amounts) public {
        for (uint i = 0; i < users.length; i++) {
            // ⚠️ Nếu users[5] fail → tất cả fail
            payable(users[i]).transfer(amounts[i]);
        }
    }
}

// ✅ PULL: User tự rút tiền
contract PullPayment {
    mapping(address => uint) public rewards;

    function recordRewards(address[] memory users, uint[] memory amounts) public {
        for (uint i = 0; i < users.length; i++) {
            // ✅ Chỉ ghi nhận, không gửi
            rewards[users[i]] += amounts[i];
        }
    }

    function claimReward() public {
        uint amount = rewards[msg.sender];
        require(amount > 0, "No reward");

        rewards[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

---

#### 🎯 Use Case: Staking Rewards

```solidity
contract StakingWithPull {
    struct Stake {
        uint256 amount;
        uint256 timestamp;
    }

    mapping(address => Stake) public stakes;
    mapping(address => uint256) public claimableRewards;

    // ============================================
    // Stake tokens
    // ============================================
    function stake(uint256 amount) external {
        require(amount > 0, "Invalid amount");

        // Tính reward cũ trước khi stake thêm
        if (stakes[msg.sender].amount > 0) {
            uint256 pending = calculateReward(msg.sender);
            claimableRewards[msg.sender] += pending;
        }

        stakes[msg.sender].amount += amount;
        stakes[msg.sender].timestamp = block.timestamp;
    }

    // ============================================
    // Tính reward (không tốn gas)
    // ============================================
    function calculateReward(address user) public view returns (uint256) {
        Stake memory userStake = stakes[user];
        if (userStake.amount == 0) return 0;

        uint256 stakingDuration = block.timestamp - userStake.timestamp;
        // 10% APR
        return (userStake.amount * stakingDuration * 10) / (365 days * 100);
    }

    // ============================================
    // User tự claim reward (Pull pattern)
    // ============================================
    function claimRewards() external {
        uint256 pending = calculateReward(msg.sender);
        uint256 total = claimableRewards[msg.sender] + pending;

        require(total > 0, "No rewards");

        // Update state
        claimableRewards[msg.sender] = 0;
        stakes[msg.sender].timestamp = block.timestamp;

        // Transfer
        payable(msg.sender).transfer(total);
    }
}
```

---

#### 🛡️ OpenZeppelin PullPayment

```solidity
import "@openzeppelin/contracts/security/PullPayment.sol";

contract MyContract is PullPayment {
    function asyncSend(address recipient, uint256 amount) internal {
        // Ghi nhận payment thay vì gửi ngay
        _asyncTransfer(recipient, amount);
    }

    // User gọi hàm này để rút tiền
    function withdrawPayments(address payable payee) public override {
        super.withdrawPayments(payee);
    }
}
```

---

#### 🎯 Khi nào dùng Pull Pattern?

| Tình huống                    | Nên dùng Pull         |
| ----------------------------- | --------------------- |
| **Phân phối cho nhiều người** | ✅ Airdrop, dividends |
| **Hoàn tiền auction**         | ✅ Refund bidders     |
| **Trả thưởng staking**        | ✅ Claim rewards      |
| **Lottery payouts**           | ✅ Winners claim      |
| **Gửi cho 1 người đáng tin**  | ❌ Có thể dùng Push   |

---

## 5. Use Battle-Tested Libraries

### 🔴 Pain Point (Nỗi đau)

Tự viết lại các logic cơ bản như kiểm soát quyền (`Ownable`), chống re-entrancy (`ReentrancyGuard`), hoặc tính toán số học an toàn rất dễ mắc lỗi. Những lỗi này đã được cộng đồng phát hiện và sửa chữa **hàng trăm lần**.

**Ví dụ vấn đề:**

```solidity
// ❌ TỒI: Tự viết Ownable
contract MyOwnable {
    address owner;

    // 🐛 Bug 1: Không có event
    // 🐛 Bug 2: Không có renounceOwnership
    // 🐛 Bug 3: Không check address(0)
    function setOwner(address newOwner) public {
        require(msg.sender == owner);
        owner = newOwner;  // ⚠️ Có thể set thành address(0) → mất quyền mãi mãi
    }
}

// ❌ TỒI: Tự viết ReentrancyGuard
contract MyGuard {
    bool locked;

    modifier noReentrancy() {
        require(!locked);
        locked = true;
        _;
        locked = false;  // 🐛 Nếu hàm revert, locked không được reset
    }
}
```

---

### 🧭 Orient (Định hướng)

Đừng phát minh lại bánh xe. Hãy tận dụng các thư viện mã nguồn mở, **đã được kiểm toán** và sử dụng rộng rãi bởi hàng nghìn dự án.

**Nguyên tắc vàng:**

> "Dùng code đã test 10,000 lần, không dùng code tự viết test 10 lần"

---

### ✅ Solution (Giải pháp)

Sử dụng **OpenZeppelin Contracts** - thư viện chuẩn công nghiệp cho Solidity.

**Các module quan trọng:**

- `Ownable`: Quản lý owner
- `AccessControl`: Quản lý role-based permissions
- `ReentrancyGuard`: Chống re-entrancy
- `Pausable`: Tạm dừng contract khi khẩn cấp
- `ERC20/721/1155`: Token standards
- `SafeERC20`: Safe token transfers

---

### 💡 Example (Ví dụ & Khi nào dùng)

#### ✅ Sử dụng OpenZeppelin

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

// ============================================
// ✅ Contract sử dụng battle-tested libs
// ============================================
contract SecureVault is Ownable, ReentrancyGuard, Pausable {
    using SafeERC20 for IERC20;

    IERC20 public token;
    mapping(address => uint256) public balances;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    constructor(address _token) Ownable(msg.sender) {
        token = IERC20(_token);
    }

    // ============================================
    // Deposit với ReentrancyGuard
    // ============================================
    function deposit(uint256 amount) external nonReentrant whenNotPaused {
        require(amount > 0, "Invalid amount");

        // ✅ SafeERC20: Xử lý token không chuẩn
        token.safeTransferFrom(msg.sender, address(this), amount);

        balances[msg.sender] += amount;
        emit Deposited(msg.sender, amount);
    }

    // ============================================
    // Withdraw với CEI + ReentrancyGuard
    // ============================================
    function withdraw(uint256 amount) external nonReentrant whenNotPaused {
        require(balances[msg.sender] >= amount, "Insufficient balance");

        balances[msg.sender] -= amount;

        // ✅ SafeERC20: Tự động revert nếu fail
        token.safeTransfer(msg.sender, amount);

        emit Withdrawn(msg.sender, amount);
    }

    // ============================================
    // Emergency: Chỉ owner mới pause
    // ============================================
    function pause() external onlyOwner {
        _pause();
    }

    function unpause() external onlyOwner {
        _unpause();
    }

    // ============================================
    // Emergency: Rút token bị kẹt
    // ============================================
    function emergencyWithdraw(address tokenAddress, uint256 amount)
        external
        onlyOwner
    {
        IERC20(tokenAddress).safeTransfer(owner(), amount);
    }
}
```

---

#### 🎯 OpenZeppelin Modules chi tiết

```solidity
// ============================================
// 1. Ownable: Quản lý owner đơn giản
// ============================================
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyContract is Ownable {
    constructor() Ownable(msg.sender) {}

    function adminFunction() external onlyOwner {
        // Chỉ owner gọi được
    }

    function transferOwnership(address newOwner) public override onlyOwner {
        // ✅ Tự động check address(0)
        // ✅ Emit event
        super.transferOwnership(newOwner);
    }

    function renounceOwnership() public override onlyOwner {
        // ✅ Từ bỏ quyền owner (không thể undo)
        super.renounceOwnership();
    }
}

// ============================================
// 2. AccessControl: Role-based permissions
// ============================================
import "@openzeppelin/contracts/access/AccessControl.sol";

contract MyDAO is AccessControl {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(ADMIN_ROLE, msg.sender);
    }

    function mint(address to, uint amount) external onlyRole(MINTER_ROLE) {
        // Chỉ MINTER_ROLE gọi được
    }

    function addAdmin(address account) external onlyRole(ADMIN_ROLE) {
        grantRole(ADMIN_ROLE, account);
    }
}

// ============================================
// 3. Pausable: Emergency stop
// ============================================
import "@openzeppelin/contracts/security/Pausable.sol";

contract MyToken is Pausable {
    function transfer(address to, uint amount) external whenNotPaused {
        // Chỉ hoạt động khi không bị pause
    }

    function pause() external onlyOwner {
        _pause();
    }
}

// ============================================
// 4. SafeERC20: An toàn với token không chuẩn
// ============================================
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract TokenHandler {
    using SafeERC20 for IERC20;

    function handleToken(IERC20 token, address to, uint amount) external {
        // ✅ Tự động revert nếu transfer fail
        // ✅ Xử lý token không return bool
        token.safeTransfer(to, amount);
        token.safeTransferFrom(msg.sender, to, amount);
        token.safeApprove(to, amount);
    }
}
```

---

#### 🛡️ Tại sao SafeERC20 quan trọng?

```solidity
// ❌ VẤN ĐỀ: Một số token không tuân thủ ERC20
// Ví dụ: USDT không return bool trong transfer()

contract Unsafe {
    function transferToken(IERC20 token, address to, uint amount) external {
        // ⚠️ NGUY HIỂM: Nếu token không return bool, code fail
        bool success = token.transfer(to, amount);
        require(success, "Transfer failed");
    }
}

// ✅ GIẢI PHÁP: SafeERC20
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract Safe {
    using SafeERC20 for IERC20;

    function transferToken(IERC20 token, address to, uint amount) external {
        // ✅ Tự động xử lý mọi loại token
        token.safeTransfer(to, amount);
    }
}
```

---

#### 📦 Cài đặt OpenZeppelin

```bash
# Với Hardhat
npm install @openzeppelin/contracts

# Với Foundry
forge install OpenZeppelin/openzeppelin-contracts
```

```solidity
// hardhat.config.js
module.exports = {
  solidity: "0.8.20",
  settings: {
    optimizer: {
      enabled: true,
      runs: 200
    }
  }
};
```

---

#### 🎯 Khi nào dùng thư viện nào?

| Yêu cầu                               | Thư viện                              |
| ------------------------------------- | ------------------------------------- |
| **Quản lý owner**                     | `Ownable`                             |
| **Nhiều role**                        | `AccessControl`                       |
| **Chống re-entrancy**                 | `ReentrancyGuard`                     |
| **Emergency stop**                    | `Pausable`                            |
| **Tạo token**                         | `ERC20`, `ERC721`                     |
| **Xử lý token không chuẩn**           | `SafeERC20`                           |
| **Tính toán số học (Solidity < 0.8)** | `SafeMath`                            |
| **Upgradeable contract**              | `@openzeppelin/contracts-upgradeable` |

---

## 6. Minimize Privileges

### 🔴 Pain Point (Nỗi đau)

Khi contract có quá nhiều quyền không cần thiết, nó trở thành mục tiêu tấn công hấp dẫn. Nếu private key của admin bị đánh cắp, toàn bộ hệ thống sụp đổ.

**Ví dụ vấn đề:**

```solidity
// ❌ TỒI: Owner có quá nhiều quyền
contract DangerousToken {
    address public owner;
    mapping(address => uint) public balances;

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    // 🚨 Owner có thể rút toàn bộ balance của bất kỳ ai
    function adminSteal(address victim) external onlyOwner {
        balances[owner] += balances[victim];
        balances[victim] = 0;
    }

    // 🚨 Owner có thể mint không giới hạn → Phá giá token
    function adminMint(uint amount) external onlyOwner {
        balances[owner] += amount;
    }

    // 🚨 Owner có thể pause mãi mãi
    function adminPause() external onlyOwner {
        paused = true;  // Không có unpause
    }
}
```

**Hậu quả:**

- 🔑 Single point of failure (một private key bị mất = game over)
- 🎯 Mục tiêu tấn công hấp dẫn
- 😱 User không tin tưởng (rug pull risk)
- ⚖️ Vấn đề pháp lý (centralization)

---

### 🧭 Orient (Định hướng)

Áp dụng nguyên tắc **"Least Privilege"**: Mỗi account chỉ nên có **quyền tối thiểu** cần thiết để thực hiện công việc của nó.

**Nguyên tắc vàng:**

> "Ít quyền = Ít rủi ro"

---

### ✅ Solution (Giải pháp)

1. **Immutable > Mutable**: Làm constant nếu không cần thay đổi
2. **Timelock**: Admin thay đổi phải chờ một khoảng thời gian
3. **Multisig**: Cần nhiều chữ ký để thực hiện hành động quan trọng
4. **Role-based**: Tách quyền thành nhiều role nhỏ
5. **Cap limits**: Giới hạn số lượng có thể thay đổi

---

### 💡 Example (Ví dụ & Khi nào dùng)

#### ✅ Code an toàn: Minimize Privileges

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/AccessControl.sol";

// ============================================
// ✅ Token với quyền hạn hợp lý
// ============================================
contract SecureToken is AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

    mapping(address => uint256) public balances;
    uint256 public totalSupply;
    bool public paused;

    // ============================================
    // 1. IMMUTABLE: Các giá trị không đổi
    // ============================================
    uint256 public immutable MAX_SUPPLY = 1_000_000 * 10**18;  // Không thể thay đổi
    uint256 public immutable MINT_CAP = 10_000 * 10**18;      // Giới hạn mỗi lần mint

    // ============================================
    // 2. TIMELOCK: Thay đổi phải chờ
    // ============================================
    uint256 public constant TIMELOCK_DELAY = 2 days;

    struct PendingChange {
        uint256 newValue;
        uint256 executeTime;
    }

    PendingChange public pendingFeeChange;
    uint256 public fee;

    event FeeChangeProposed(uint256 newFee, uint256 executeTime);
    event FeeChanged(uint256 oldFee, uint256 newFee);

    // ============================================
    // 3. ROLE-BASED: Tách quyền
    // ============================================
    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        // Admin KHÔNG tự động có MINTER_ROLE
    }

    // ============================================
    // Mint với giới hạn
    // ============================================
    function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) {
        require(!paused, "Paused");
        require(amount <= MINT_CAP, "Exceeds mint cap");
        require(totalSupply + amount <= MAX_SUPPLY, "Exceeds max supply");

        balances[to] += amount;
        totalSupply += amount;
    }

    // ============================================
    // Pause với timelock tự động unpause
    // ============================================
    uint256 public pausedUntil;
    uint256 public constant MAX_PAUSE_DURATION = 7 days;

    function pause(uint256 duration) external onlyRole(PAUSER_ROLE) {
        require(duration <= MAX_PAUSE_DURATION, "Pause too long");

        paused = true;
        pausedUntil = block.timestamp + duration;
    }

    function checkPause() public {
        if (paused && block.timestamp >= pausedUntil) {
            paused = false;  // Tự động unpause
        }
    }

    // ============================================
    // Thay đổi fee với timelock
    // ============================================
    function proposeFeeChange(uint256 newFee) external onlyRole(DEFAULT_ADMIN_ROLE) {
        require(newFee <= 1000, "Fee too high");  // Max 10%

        pendingFeeChange = PendingChange({
            newValue: newFee,
            executeTime: block.timestamp + TIMELOCK_DELAY
        });

        emit FeeChangeProposed(newFee, pendingFeeChange.executeTime);
    }

    function executeFeeChange() external {
        require(pendingFeeChange.executeTime != 0, "No pending change");
        require(block.timestamp >= pendingFeeChange.executeTime, "Timelock not expired");

        uint256 oldFee = fee;
        fee = pendingFeeChange.newValue;

        delete pendingFeeChange;

        emit FeeChanged(oldFee, fee);
    }

    // ============================================
    // Transfer: Không thể bị admin can thiệp
    // ============================================
    function transfer(address to, uint256 amount) external {
        require(!paused || block.timestamp >= pausedUntil, "Paused");
        require(balances[msg.sender] >= amount, "Insufficient balance");

        balances[msg.sender] -= amount;
        balances[to] += amount;

        // ✅ Admin KHÔNG thể can thiệp vào transfer của user
    }
}
```

---

#### 🔐 Multisig Wallet Pattern

```solidity
// ============================================
// ✅ Multisig: Cần 3/5 chữ ký
// ============================================
contract Multisig {
    address[] public owners;
    uint256 public required;  // Số chữ ký cần thiết

    struct Transaction {
        address to;
        uint256 value;
        bytes data;
        bool executed;
        uint256 confirmations;
    }

    mapping(uint256 => Transaction) public transactions;
    mapping(uint256 => mapping(address => bool)) public confirmations;
    uint256 public transactionCount;

    constructor(address[] memory _owners, uint256 _required) {
        require(_owners.length >= _required, "Invalid setup");
        require(_required > 0, "Required must be > 0");

        owners = _owners;
        required = _required;
    }

    function submitTransaction(address to, uint256 value, bytes memory data)
        external
        returns (uint256)
    {
        require(isOwner(msg.sender), "Not owner");

        uint256 txId = transactionCount++;
        transactions[txId] = Transaction({
            to: to,
            value: value,
            data: data,
            executed: false,
            confirmations: 0
        });

        return txId;
    }

    function confirmTransaction(uint256 txId) external {
        require(isOwner(msg.sender), "Not owner");
        require(!confirmations[txId][msg.sender], "Already confirmed");

        confirmations[txId][msg.sender] = true;
        transactions[txId].confirmations++;

        if (transactions[txId].confirmations >= required) {
            executeTransaction(txId);
        }
    }

    function executeTransaction(uint256 txId) internal {
        Transaction storage txn = transactions[txId];
        require(!txn.executed, "Already executed");
        require(txn.confirmations >= required, "Not enough confirmations");

        txn.executed = true;
        (bool success, ) = txn.to.call{value: txn.value}(txn.data);
        require(success, "Execution failed");
    }

    function isOwner(address account) public view returns (bool) {
        for (uint i = 0; i < owners.length; i++) {
            if (owners[i] == account) return true;
        }
        return false;
    }
}
```

---

#### 🎯 Best Practices

```solidity
// ============================================
// ✅ 1. Dùng immutable cho config quan trọng
// ============================================
contract GoodConfig {
    address public immutable token;        // Không đổi sau deploy
    uint256 public immutable maxSupply;    // Không đổi
    uint256 public immutable startTime;    // Không đổi

    constructor(address _token, uint256 _maxSupply) {
        token = _token;
        maxSupply = _maxSupply;
        startTime = block.timestamp;
    }
}

// ============================================
// ✅ 2. Giới hạn quyền admin
// ============================================
contract LimitedAdmin {
    uint256 public fee;
    uint256 public constant MAX_FEE = 1000;  // Max 10%

    function setFee(uint256 newFee) external onlyOwner {
        require(newFee <= MAX_FEE, "Fee too high");
        fee = newFee;
    }
}

// ============================================
// ✅ 3. Emergency pause với thời gian giới hạn
// ============================================
contract TimedPause {
    bool public paused;
    uint256 public pausedAt;
    uint256 public constant MAX_PAUSE = 7 days;

    function pause() external onlyOwner {
        require(!paused, "Already paused");
        paused = true;
        pausedAt = block.timestamp;
    }

    function unpause() external {
        // Bất kỳ ai cũng có thể unpause sau MAX_PAUSE
        require(paused, "Not paused");
        require(
            msg.sender == owner || block.timestamp >= pausedAt + MAX_PAUSE,
            "Cannot unpause yet"
        );
        paused = false;
    }
}

// ============================================
// ✅ 4. Revoke privileges sau deploy
// ============================================
contract RevokeAdmin {
    address public admin;
    bool public adminRevoked;

    function revokeAdmin() external {
        require(msg.sender == admin, "Not admin");
        adminRevoked = true;
        admin = address(0);
    }

    modifier onlyAdmin() {
        require(!adminRevoked, "Admin revoked");
        require(msg.sender == admin, "Not admin");
        _;
    }
}
```

---

#### 🎯 Checklist giảm quyền

| Hành động           | Câu hỏi                | Best Practice              |
| ------------------- | ---------------------- | -------------------------- |
| **Thay đổi state**  | Có cần thay đổi không? | Dùng `immutable` nếu không |
| **Admin function**  | Có giới hạn không?     | Set cap, max value         |
| **Pause**           | Có tự động unpause?    | Set max pause duration     |
| **Critical change** | Có timelock không?     | Delay ít nhất 24h          |
| **High privilege**  | Có cần multisig không? | 3/5 hoặc 4/7 signatures    |
| **Mint/burn**       | Có cap không?          | Set max mint per tx        |

---

## 7. Reentrancy Protection

### 🔴 Pain Point (Nỗi đau)

Re-entrancy là một trong những lỗ hổng phổ biến và nguy hiểm nhất. Mặc dù đã có CEI pattern (phần 3), nhưng đôi khi logic phức tạp khiến việc tuân thủ CEI khó khăn.

**Ví dụ vấn đề:**

```solidity
// ❌ Phức tạp, khó áp dụng CEI
contract ComplexLogic {
    function complexWithdraw() external {
        // Nhiều state cần update
        // Nhiều check điều kiện
        // Gọi nhiều contract khác
        // → Khó đảm bảo CEI 100%
    }
}
```

---

### 🧭 Orient (Định hướng)

Sử dụng **ReentrancyGuard** như một lớp bảo vệ thứ hai, bổ sung cho CEI pattern.

**Nguyên tắc vàng:**

> "CEI + ReentrancyGuard = An toàn kép"

---

### ✅ Solution (Giải pháp)

```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract Safe is ReentrancyGuard {
    function withdraw() external nonReentrant {
        // Logic an toàn
    }
}
```

---

### 💡 Example (Ví dụ & Khi nào dùng)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureVault is ReentrancyGuard {
    mapping(address => uint256) public balances;

    // ✅ CEI + nonReentrant
    function withdraw(uint256 amount) external nonReentrant {
        // 1. Checks
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // 2. Effects
        balances[msg.sender] -= amount;

        // 3. Interactions
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

**Khi nào dùng:**

- ✅ Mọi hàm có gửi ETH
- ✅ Mọi hàm gọi external contract
- ✅ Mọi hàm có delegate call
- ✅ Logic phức tạp khó đảm bảo CEI

---

## 8. Explicit Visibility & Error Handling

### 🔴 Pain Point (Nỗi đau)

Không khai báo visibility rõ ràng hoặc dùng `require` string dài tốn gas và khó debug.

**Ví dụ vấn đề:**

```solidity
// ❌ TỒI
contract Bad {
    function transfer() {  // ⚠️ Không có visibility
        require(balance > 0, "Insufficient balance to transfer tokens");  // ⛽ Tốn gas
    }
}
```

---

### ✅ Solution (Giải pháp)

1. **Luôn khai báo visibility**: `public`, `external`, `internal`, `private`
2. **Dùng Custom Errors** (Solidity >= 0.8.4): Tiết kiệm gas, dễ debug

---

### 💡 Example (Ví dụ & Khi nào dùng)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// ============================================
// ✅ Custom Errors (Gas-efficient)
// ============================================
error InsufficientBalance(uint256 available, uint256 required);
error Unauthorized(address caller);
error InvalidAddress();

contract SecureToken {
    mapping(address => uint256) private _balances;
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    // ✅ Explicit visibility
    function transfer(address to, uint256 amount) external {
        if (to == address(0)) revert InvalidAddress();
        if (_balances[msg.sender] < amount) {
            revert InsufficientBalance(_balances[msg.sender], amount);
        }

        _balances[msg.sender] -= amount;
        _balances[to] += amount;
    }

    // ✅ Internal helper
    function _transfer(address from, address to, uint256 amount) internal {
        _balances[from] -= amount;
        _balances[to] += amount;
    }

    // ✅ Private for contract only
    function _calculateFee(uint256 amount) private pure returns (uint256) {
        return amount / 100;
    }
}
```

**So sánh gas:**

```solidity
// ❌ require string: ~50,000 gas
require(balance > 0, "Insufficient balance");

// ✅ custom error: ~30,000 gas (tiết kiệm 40%)
if (balance == 0) revert InsufficientBalance();
```

---

## 9. Bounded Loops & Gas Limits

### 🔴 Pain Point (Nỗi đau)

Vòng lặp không giới hạn trên array từ user input có thể gây **Denial of Service** do vượt quá gas limit.

**Ví dụ vấn đề:**

```solidity
// ❌ TỒI: Loop không giới hạn
contract Vulnerable {
    address[] public users;

    function distributeRewards() external {
        for (uint i = 0; i < users.length; i++) {  // ⚠️ Nếu users.length = 10,000?
            payable(users[i]).transfer(1 ether);
        }
    }
}
```

---

### ✅ Solution (Giải pháp)

1. **Giới hạn array size**: `require(array.length <= MAX_SIZE)`
2. **Pagination**: Chia nhỏ thành nhiều batch
3. **Pull pattern**: Thay vì loop, let user claim

---

### 💡 Example (Ví dụ & Khi nào dùng)

```solidity
// ✅ GOOD: Bounded loops
contract BoundedRewards {
    uint256 public constant MAX_BATCH_SIZE = 100;

    function distributeBatch(address[] calldata users, uint256[] calldata amounts)
        external
    {
        require(users.length <= MAX_BATCH_SIZE, "Batch too large");
        require(users.length == amounts.length, "Length mismatch");

        for (uint256 i = 0; i < users.length; i++) {
            payable(users[i]).transfer(amounts[i]);
        }
    }
}

// ✅ BETTER: Pull pattern (no loop needed)
contract PullRewards {
    mapping(address => uint256) public rewards;

    function claim() external {
        uint256 amount = rewards[msg.sender];
        rewards[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

---

## 10. Avoid tx.origin & Validate Inputs

### 🔴 Pain Point (Nỗi đau)

`tx.origin` dễ bị phishing. Input không validate gây lỗi.

---

### ✅ Solution (Giải pháp)

```solidity
// ✅ GOOD
contract Secure {
    function transfer(address to, uint256 amount) external {
        require(msg.sender == owner, "Not owner");  // ✅ Dùng msg.sender
        require(to != address(0), "Invalid address");  // ✅ Validate input
        require(amount > 0, "Invalid amount");

        // ... logic
    }
}
```

---

## 11. Events & Invariant Checks

### 🔴 Pain Point (Nỗi đau)

Không emit event = không trace được history. Không check invariants = bug ẩn.

---

### ✅ Solution (Giải pháp)

```solidity
contract WithEvents {
    uint256 public totalSupply;
    mapping(address => uint256) public balances;

    event Transfer(address indexed from, address indexed to, uint256 amount);

    function transfer(address to, uint256 amount) external {
        balances[msg.sender] -= amount;
        balances[to] += amount;

        emit Transfer(msg.sender, to, amount);

        // ✅ Invariant check
        assert(balances[msg.sender] + balances[to] <= totalSupply);
    }
}
```

---

## 12. Testing & Quality Assurance

### 🔴 Pain Point (Nỗi đau)

Deploy mà chưa test kỹ = mạo hiểm mất tiền.

---

### ✅ Solution (Giải pháp)

**Checklist trước khi deploy:**

```bash
# 1. Unit tests
npx hardhat test

# 2. Static analysis
slither .

# 3. Gas profiling
REPORT_GAS=true npx hardhat test

# 4. Fuzz testing
echidna . --contract MyContract

# 5. Coverage
npx hardhat coverage

# 6. Mainnet fork test
npx hardhat test --network hardhat
```

**Test template:**

```solidity
contract MyTokenTest {
    MyToken token;

    function setUp() public {
        token = new MyToken();
    }

    function testTransfer() public {
        token.transfer(address(1), 100);
        assertEq(token.balanceOf(address(1)), 100);
    }

    function testRevertInsufficientBalance() public {
        vm.expectRevert();
        token.transfer(address(1), 1000);
    }

    function testFuzz_Transfer(address to, uint256 amount) public {
        // Fuzz testing with random inputs
    }
}
```

---

## 📚 Tổng kết Toàn bộ Nguyên tắc

### 🎯 Checklist Thiết kế Contract An toàn

| #   | Nguyên tắc          | Kiểm tra                                    |
| --- | ------------------- | ------------------------------------------- |
| 1   | **SRP**             | ✅ Mỗi contract < 300 dòng, một trách nhiệm |
| 2   | **Composition**     | ✅ Dùng interface, không kế thừa sâu        |
| 3   | **CEI**             | ✅ Check → Effect → Interact                |
| 4   | **Pull Pattern**    | ✅ User tự rút, không tự đẩy                |
| 5   | **OpenZeppelin**    | ✅ Dùng `Ownable`, `ReentrancyGuard`        |
| 6   | **Min Privileges**  | ✅ Dùng `immutable`, timelock, multisig     |
| 7   | **ReentrancyGuard** | ✅ Thêm `nonReentrant`                      |
| 8   | **Visibility**      | ✅ Khai báo rõ, dùng custom errors          |
| 9   | **Bounded Loops**   | ✅ Giới hạn array, pagination               |
| 10  | **Validate**        | ✅ Dùng `msg.sender`, validate input        |
| 11  | **Events**          | ✅ Emit events, check invariants            |
| 12  | **Testing**         | ✅ Unit + Fuzz + Static analysis            |

---

### ✨ Template Contract Production-Ready

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

// ============================================
// ✅ PRODUCTION-READY CONTRACT TEMPLATE
// ============================================
contract ProductionContract is Ownable, ReentrancyGuard, Pausable {

    // 1. Custom Errors (Gas-efficient)
    error InvalidAmount();
    error InsufficientBalance(uint256 available, uint256 required);
    error InvalidAddress();

    // 2. Immutable variables
    uint256 public immutable MAX_SUPPLY;
    address public immutable TOKEN;

    // 3. State variables
    mapping(address => uint256) public balances;
    uint256 public totalSupply;

    // 4. Events
    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    // 5. Constructor
    constructor(address _token, uint256 _maxSupply) Ownable(msg.sender) {
        if (_token == address(0)) revert InvalidAddress();
        TOKEN = _token;
        MAX_SUPPLY = _maxSupply;
    }

    // 6. External functions (user-facing)
    function deposit(uint256 amount) external nonReentrant whenNotPaused {
        if (amount == 0) revert InvalidAmount();
        if (totalSupply + amount > MAX_SUPPLY) revert InvalidAmount();

        // CEI Pattern
        balances[msg.sender] += amount;
        totalSupply += amount;

        emit Deposited(msg.sender, amount);

        // Invariant check
        assert(totalSupply <= MAX_SUPPLY);
    }

    function withdraw(uint256 amount) external nonReentrant whenNotPaused {
        uint256 balance = balances[msg.sender];
        if (balance < amount) {
            revert InsufficientBalance(balance, amount);
        }

        // CEI Pattern
        balances[msg.sender] = balance - amount;
        totalSupply -= amount;

        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");

        emit Withdrawn(msg.sender, amount);
    }

    // 7. Admin functions (restricted)
    function pause() external onlyOwner {
        _pause();
    }

    function unpause() external onlyOwner {
        _unpause();
    }

    // 8. View functions (gas-free)
    function getBalance(address user) external view returns (uint256) {
        return balances[user];
    }

    // 9. Internal/private helpers
    function _internalLogic() internal {
        // Helper logic
    }
}
```

---

### 🚀 Deployment Checklist

**Trước khi deploy mainnet:**

- [ ] ✅ Unit tests pass 100%
- [ ] ✅ Fuzz testing done
- [ ] ✅ Slither analysis (no high/medium issues)
- [ ] ✅ Gas optimized
- [ ] ✅ Mainnet fork testing
- [ ] ✅ Code reviewed by team
- [ ] ✅ External audit (for high-value)
- [ ] ✅ Multisig setup
- [ ] ✅ Timelock deployed
- [ ] ✅ Emergency pause tested
- [ ] ✅ Documentation complete
- [ ] ✅ Bug bounty program ready

---

### 📖 Tài liệu tham khảo

**Security:**

- [Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [SWC Registry](https://swcregistry.io/)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts)

**Tools:**

- [Slither](https://github.com/crytic/slither) - Static analyzer
- [Echidna](https://github.com/crytic/echidna) - Fuzzer
- [Foundry](https://book.getfoundry.sh/) - Testing framework
- [Tenderly](https://tenderly.co/) - Monitoring & debugging

**Auditors:**

- [OpenZeppelin](https://www.openzeppelin.com/security-audits)
- [Trail of Bits](https://www.trailofbits.com/)
- [ConsenSys Diligence](https://consensys.net/diligence/)

---

## 🎓 Kết luận

Thiết kế smart contract an toàn không chỉ là viết code đúng cú pháp, mà là **tư duy phòng thủ** - luôn giả định mọi thứ có thể sai và chuẩn bị sẵn phương án.

**12 nguyên tắc này** là kim chỉ nam được rút ra từ hàng trăm lỗ hổng thực tế và hàng tỷ đô la tổn thất. Hãy áp dụng chúng một cách nghiêm túc.

**Remember:**

> "Code is law. Once deployed, there's no undo button."

---

**[⬆ Về đầu trang](#-nguyên-tắc-thiết-kế-smart-contract)**

---

© 2025 hwHoai | [github.com/hwHoai](https://github.com/hwHoai) | License: MIT

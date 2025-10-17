# 📚 Tổng hợp Kiến thức Lập trình Blockchain & dApp

> Tài liệu tổng hợp toàn bộ kiến thức từ nền tảng blockchain, Solidity, đến việc xây dựng dApp hoàn chỉnh với Next.js và Wagmi.

---

## 🧱 Phần 1: Nền tảng Blockchain & Smart Contract

### 1.1. Các Khái niệm Cốt lõi

#### **Blockchain**

Một cuốn sổ cái kỹ thuật số, phi tập trung và không thể thay đổi. Mỗi khối (block) chứa nhiều giao dịch và được liên kết mật mã với khối trước đó, tạo thành một chuỗi không thể sửa đổi.

#### **Block**

Một "trang sổ" chứa nhiều giao dịch, được liên kết với nhau thành một chuỗi. Mỗi block có:

- **Header**: Chứa metadata (hash của block trước, timestamp, nonce)
- **Body**: Danh sách các giao dịch

#### **Smart Contract**

Một chương trình máy tính chạy trên blockchain, hoạt động như **backend phi tập trung** của dApp.

**Đặc điểm quan trọng:**

- ✅ **Không tự chạy**: Cần được kích hoạt bởi giao dịch (transaction)
- ✅ **Bất biến (Immutable)**: Sau khi deploy không thể thay đổi code
- ✅ **Minh bạch**: Code công khai, ai cũng có thể xem
- ✅ **Xác định (Deterministic)**: Cùng input luôn cho cùng output

#### **Trạng thái (State) vs. Lịch sử (History)**

| Khái niệm              | Mô tả                                             | Lưu ở đâu                                    |
| ---------------------- | ------------------------------------------------- | -------------------------------------------- |
| **Trạng thái (State)** | Giá trị _hiện tại_ của tất cả biến trong contract | Cấu trúc dữ liệu riêng của node (State Trie) |
| **Lịch sử (History)**  | Các _giao dịch_ đã làm thay đổi trạng thái        | Trong các block của blockchain               |

**Ví dụ minh họa:**

```solidity
contract Counter {
    uint256 public count; // State: giá trị hiện tại

    function increment() public {
        count++; // Transaction này sẽ được lưu trong block
    }
}
```

- **State**: `count = 42` (giá trị hiện tại)
- **History**: Block #1000 chứa tx của Alice gọi `increment()`, Block #1001 chứa tx của Bob gọi `increment()`, etc.

---

### 1.2. Phân loại Blockchain

#### **EVM vs. Non-EVM**

**🔷 EVM (Ethereum Virtual Machine)**

- "Hệ điều hành" của Ethereum
- Được nhiều blockchain khác sử dụng: BNB Chain, Polygon, Avalanche, Fantom
- Smart contract viết bằng **Solidity** (hoặc Vyper)
- **Ưu điểm**: Hệ sinh thái rộng lớn, công cụ phát triển phong phú
- **Nhược điểm**: Chi phí gas cao, hiệu suất hạn chế

**🔶 Non-EVM**
| Blockchain | Ngôn ngữ | Đặc điểm |
|------------|----------|----------|
| Solana | Rust | Tốc độ cao, phí thấp |
| Cosmos | Go | Khả năng tương tác giữa các chain |
| Near | Rust, AssemblyScript | Thân thiện với developer |
| Cardano | Plutus (Haskell) | Bảo mật cao, formal verification |

#### **Public Chain vs. Private Chain**

**🌐 Public Chain (Sepolia, Ethereum Mainnet)**

- ✅ **Công khai**: Ai cũng có thể xem giao dịch
- ✅ **Phi tập trung**: Không có cơ quan kiểm soát
- ✅ **Minh bạch**: Niềm tin từ công chúng
- ❌ **Faucet hạn chế**: Khó lấy ETH test
- ❌ **Chi phí cao**: Phải trả gas fee thật

**Khi nào dùng:** DeFi, NFT, token công khai

**🔒 Private Chain (Hardhat Node, Geth nội bộ)**

- ✅ **Riêng tư**: Kiểm soát quyền truy cập
- ✅ **Tốc độ cao**: Không cần consensus phức tạp
- ✅ **Không cần Faucet**: Tự tạo ETH không giới hạn
- ✅ **Chi phí thấp**: Không mất tiền thật
- ❌ **Tin cậy hạn chế**: Phụ thuộc vào cơ quan vận hành

**Khi nào dùng:** Ứng dụng nội bộ doanh nghiệp, hệ thống chấm công, quản lý chuỗi cung ứng nội bộ

---

## ✍️ Phần 2: Ngôn ngữ Solidity

Solidity là ngôn ngữ lập trình chính để viết smart contract trên các blockchain EVM.

### 2.1. Cấu trúc cơ bản

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MyContract {
    // 1. Biến trạng thái (State Variables)
    address public owner;
    mapping(address => bool) public whitelist;

    // 2. Sự kiện (Events)
    event OwnerChanged(address indexed oldOwner, address indexed newOwner);
    event UserWhitelisted(address indexed user);

    // 3. Modifier (Kiểm tra điều kiện)
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _; // Vị trí thực thi hàm gốc
    }

    // 4. Constructor (Hàm khởi tạo)
    constructor() {
        owner = msg.sender;
    }

    // 5. Hàm public
    function setOwner(address _newOwner) public onlyOwner {
        address oldOwner = owner;
        owner = _newOwner;
        emit OwnerChanged(oldOwner, _newOwner);
    }

    function addToWhitelist(address _user) public onlyOwner {
        whitelist[_user] = true;
        emit UserWhitelisted(_user);
    }
}
```

---

### 2.2. Các thành phần quan trọng

#### **1️⃣ Biến Trạng thái (State Variables)**

Được lưu **vĩnh viễn** trên blockchain. Mỗi lần thay đổi tốn gas.

```solidity
contract Storage {
    uint256 public count;        // Lưu trên blockchain
    string public name;          // Lưu trên blockchain

    function increment() public {
        count++;  // ⛽ Tốn gas vì thay đổi state
    }

    function getName() public view returns (string memory) {
        return name;  // ✅ Không tốn gas vì chỉ đọc
    }
}
```

#### **2️⃣ Visibility (Phạm vi truy cập)**

| Visibility | Mô tả     | Getter tự động | Ai gọi được                  |
| ---------- | --------- | -------------- | ---------------------------- |
| `public`   | Công khai | ✅ Có          | Contract, External, Internal |
| `private`  | Riêng tư  | ❌ Không       | Chỉ contract này             |
| `internal` | Nội bộ    | ❌ Không       | Contract này + contract con  |
| `external` | Bên ngoài | ❌ Không       | Chỉ từ bên ngoài             |

**⚠️ Lưu ý về `public`:**

- Chỉ tự tạo hàm **getter** (đọc)
- **Quyền ghi** phải kiểm soát bằng logic trong hàm

```solidity
contract Example {
    uint256 public count;  // ✅ Ai cũng ĐỌC được

    // ❌ Không có hàm set tự động, phải tự viết:
    function setCount(uint256 _newCount) public onlyOwner {
        count = _newCount;  // Kiểm soát quyền ghi
    }
}
```

#### **3️⃣ mapping vs. array**

**📊 So sánh:**

| Tiêu chí     | `mapping`          | `array`             |
| ------------ | ------------------ | ------------------- |
| **Tra cứu**  | O(1) - Cực nhanh   | O(n) - Chậm         |
| **Gas cost** | Thấp, cố định      | Cao, tăng theo size |
| **Thứ tự**   | ❌ Không           | ✅ Có               |
| **Lặp qua**  | ❌ Không thể       | ✅ Được             |
| **Use case** | Whitelist, balance | Danh sách có thứ tự |

**Ví dụ thực tế:**

```solidity
contract Comparison {
    // ✅ GOOD: Dùng mapping cho whitelist
    mapping(address => bool) public whitelist;

    function isWhitelisted(address _user) public view returns (bool) {
        return whitelist[_user];  // ⛽ Gas: ~2100
    }

    // ❌ BAD: Dùng array cho whitelist
    address[] public whitelistArray;

    function isWhitelistedArray(address _user) public view returns (bool) {
        for (uint i = 0; i < whitelistArray.length; i++) {
            if (whitelistArray[i] == _user) return true;
        }
        return false;  // ⛽ Gas: ~2100 * whitelistArray.length
    }
}
```

**💡 Best Practice: Kết hợp cả hai**

```solidity
contract BestPractice {
    mapping(address => bool) public isEmployee;  // Tra cứu nhanh
    address[] public employeeList;               // Lặp qua được

    function addEmployee(address _emp) public {
        require(!isEmployee[_emp], "Already exists");
        isEmployee[_emp] = true;
        employeeList.push(_emp);
    }

    function getAllEmployees() public view returns (address[] memory) {
        return employeeList;  // Frontend lấy danh sách dễ dàng
    }
}
```

#### **4️⃣ modifier - "Người gác cổng"**

Một cơ chế tái sử dụng để kiểm tra điều kiện trước khi thực thi hàm.

```solidity
contract AccessControl {
    address public owner;
    mapping(address => bool) public admins;

    // Modifier 1: Chỉ owner
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;  // Vị trí hàm gốc được chạy
    }

    // Modifier 2: Chỉ admin
    modifier onlyAdmin() {
        require(admins[msg.sender], "Not admin");
        _;
    }

    // Modifier 3: Kết hợp nhiều điều kiện
    modifier validAddress(address _addr) {
        require(_addr != address(0), "Zero address");
        require(_addr != address(this), "Contract address");
        _;
    }

    // Sử dụng modifier
    function addAdmin(address _admin)
        public
        onlyOwner
        validAddress(_admin)
    {
        admins[_admin] = true;
    }
}
```

**🔄 Thứ tự thực thi với nhiều modifier:**

```solidity
function test() public modifierA modifierB modifierC {
    // Logic
}

// Thứ tự: modifierA -> modifierB -> modifierC -> Logic -> modifierC -> modifierB -> modifierA
```

#### **5️⃣ event và emit - Logging on Blockchain**

Event là cơ chế **phát tín hiệu** từ smart contract ra thế giới bên ngoài.

**✨ Tại sao cần event:**

- ✅ **Chi phí thấp**: Lưu trong log, không lưu trong state
- ✅ **Frontend lắng nghe**: dApp cập nhật UI real-time
- ✅ **Lịch sử**: Query lịch sử hoạt động
- ✅ **Debug**: Theo dõi flow thực thi

```solidity
contract EventDemo {
    // 1. Định nghĩa event
    event Transfer(
        address indexed from,    // indexed: có thể filter
        address indexed to,      // indexed: có thể filter
        uint256 amount           // không indexed
    );

    event EmployeeCheckedIn(
        address indexed employee,
        uint256 indexed timestamp,
        string location
    );

    // 2. Emit event
    function transfer(address _to, uint256 _amount) public {
        // ... logic transfer
        emit Transfer(msg.sender, _to, _amount);
    }

    function checkIn(string memory _location) public {
        emit EmployeeCheckedIn(msg.sender, block.timestamp, _location);
    }
}
```

**📡 Frontend lắng nghe event (Wagmi):**

```typescript
import { useWatchContractEvent } from "wagmi";

function EmployeeCheckInListener() {
  useWatchContractEvent({
    address: "0x...",
    abi: contractABI,
    eventName: "EmployeeCheckedIn",
    onLogs(logs) {
      console.log("New check-in:", logs);
      // Cập nhật UI
    },
  });
}
```

**💡 Best Practice:**

- Dùng `indexed` cho các field cần filter (tối đa 3 indexed params)
- Emit event **sau** khi logic thành công
- Tên event nên là động từ quá khứ (VD: `Transferred`, `CheckedIn`)

---

### 2.3. Các Biến Global Quan trọng

```solidity
contract GlobalVariables {
    function examples() public payable {
        // Thông tin về người gọi
        address caller = msg.sender;        // Địa chỉ người gọi
        uint256 value = msg.value;          // Số ETH gửi kèm (wei)
        bytes memory data = msg.data;       // Dữ liệu gọi hàm

        // Thông tin về block
        uint256 blockNum = block.number;    // Số block hiện tại
        uint256 timestamp = block.timestamp; // Timestamp (giây)
        address miner = block.coinbase;     // Địa chỉ miner

        // Thông tin về transaction
        uint256 gasPrice = tx.gasprice;     // Giá gas
        address origin = tx.origin;         // Người khởi tạo tx gốc

        // Thông tin về contract
        address thisContract = address(this);
        uint256 balance = address(this).balance;
    }
}
```

---

## 🛠️ Phần 3: Môi trường & Công cụ Phát triển

### 3.1. Hardhat Framework

Hardhat là framework phát triển smart contract phổ biến nhất cho Ethereum.

#### **Cấu trúc project:**

```
my-project/
├── contracts/          # Smart contracts (.sol)
├── ignition/
│   └── modules/        # Deploy scripts
├── test/               # Test files
├── hardhat.config.js   # Cấu hình chính
└── package.json
```

#### **Các lệnh cơ bản:**

```bash
# 1. Compile contracts
npx hardhat compile

# 2. Chạy local blockchain
npx hardhat node

# 3. Chạy node với hostname tùy chỉnh (mở cho mạng LAN)
npx hardhat node --hostname 192.168.1.100

# 4. Deploy contract
npx hardhat ignition deploy ./ignition/modules/Deploy.js --network sepolia

# 5. Chạy test
npx hardhat test

# 6. Verify contract trên Etherscan
npx hardhat verify --network sepolia DEPLOYED_CONTRACT_ADDRESS "Constructor arg 1"

# 7. Console tương tác
npx hardhat console --network localhost
```

#### **Cấu hình hardhat.config.js:**

```javascript
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.20",
  networks: {
    // Local development
    localhost: {
      url: "http://127.0.0.1:8545",
    },

    // Private network (ví dụ: mạng công ty)
    company: {
      url: "http://192.168.1.100:8545",
      accounts: [process.env.PRIVATE_KEY],
    },

    // Public testnet
    sepolia: {
      url: `https://sepolia.infura.io/v3/${process.env.INFURA_API_KEY}`,
      accounts: [process.env.PRIVATE_KEY],
      chainId: 11155111,
    },

    // Mainnet
    mainnet: {
      url: `https://mainnet.infura.io/v3/${process.env.INFURA_API_KEY}`,
      accounts: [process.env.PRIVATE_KEY],
      chainId: 1,
    },
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY,
  },
};
```

---

### 3.2. Triển khai Smart Contract

#### **Hardhat Ignition (Recommended)**

Hệ thống deploy mặc định mới, mạnh mẽ hơn scripts truyền thống.

**File deploy: `ignition/modules/CheckIn.js`**

```javascript
const { buildModule } = require("@nomicfoundation/hardhat-ignition/modules");

module.exports = buildModule("CheckInModule", (m) => {
  // Tham số constructor (nếu có)
  const initialOwner = m.getParameter("owner", "0x...");

  // Deploy contract
  const checkIn = m.contract("CheckIn", [initialOwner]);

  // Return để có thể tham chiếu
  return { checkIn };
});
```

**Deploy:**

```bash
# Deploy lên localhost
npx hardhat ignition deploy ./ignition/modules/CheckIn.js --network localhost

# Deploy lên Sepolia
npx hardhat ignition deploy ./ignition/modules/CheckIn.js --network sepolia

# Deploy với parameters
npx hardhat ignition deploy ./ignition/modules/CheckIn.js --network sepolia \
  --parameters '{"owner": "0x1234..."}'
```

#### **⚠️ Xử lý lỗi Reconciliation**

**Lỗi:** `reconciliation failed`

**Nguyên nhân:** Ignition lưu lại trạng thái deploy để có thể resume. Nếu contract hoặc config thay đổi, sẽ conflict.

**Giải pháp:**

```bash
# Xóa deployment history
rm -rf ignition/deployments/chain-31337  # localhost
rm -rf ignition/deployments/chain-11155111  # sepolia

# Deploy lại
npx hardhat ignition deploy ./ignition/modules/CheckIn.js --network localhost
```

---

### 3.3. Testing Smart Contracts

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("CheckIn Contract", function () {
  let checkIn;
  let owner, employee1, employee2;

  beforeEach(async function () {
    // Get signers
    [owner, employee1, employee2] = await ethers.getSigners();

    // Deploy contract
    const CheckIn = await ethers.getContractFactory("CheckIn");
    checkIn = await CheckIn.deploy();
    await checkIn.waitForDeployment();
  });

  it("Should add employee correctly", async function () {
    await checkIn.addEmployee(employee1.address);
    expect(await checkIn.isEmployee(employee1.address)).to.equal(true);
  });

  it("Should emit event on check-in", async function () {
    await checkIn.addEmployee(employee1.address);

    await expect(checkIn.connect(employee1).check())
      .to.emit(checkIn, "EmployeeCheckedIn")
      .withArgs(
        employee1.address,
        (await ethers.provider.getBlockNumber()) + 1
      );
  });

  it("Should revert if non-employee tries to check-in", async function () {
    await expect(checkIn.connect(employee2).check()).to.be.revertedWith(
      "Not an employee"
    );
  });
});
```

**Chạy test:**

```bash
# Chạy tất cả test
npx hardhat test

# Chạy file cụ thể
npx hardhat test test/CheckIn.test.js

# Chạy với gas reporter
REPORT_GAS=true npx hardhat test
```

---

## 🔗 Phần 4: Kết nối dApp với Blockchain

### 4.1. Thư viện Front-end

#### **Viem**

Thư viện low-level, hiệu năng cao, type-safe để tương tác với Ethereum.

```typescript
import { createPublicClient, http } from "viem";
import { mainnet } from "viem/chains";

// Tạo client chỉ đọc (public)
const client = createPublicClient({
  chain: mainnet,
  transport: http(),
});

// Đọc dữ liệu
const balance = await client.getBalance({
  address: "0x...",
});

const blockNumber = await client.getBlockNumber();
```

#### **Wagmi**

Thư viện React Hooks xây dựng trên Viem, quản lý state kết nối, account, network.

```typescript
import { useAccount, useBalance, useConnect } from "wagmi";

function WalletInfo() {
  const { address, isConnected } = useAccount();
  const { data: balance } = useBalance({ address });
  const { connect, connectors } = useConnect();

  if (!isConnected) {
    return (
      <button onClick={() => connect({ connector: connectors[0] })}>
        Connect Wallet
      </button>
    );
  }

  return (
    <div>
      <p>Address: {address}</p>
      <p>
        Balance: {balance?.formatted} {balance?.symbol}
      </p>
    </div>
  );
}
```

---

### 4.2. Cấu hình Wagmi

#### **Setup cơ bản:**

```typescript
// src/config/wagmi.ts
import { createConfig, http } from "wagmi";
import { mainnet, sepolia } from "wagmi/chains";
import { injected, walletConnect } from "wagmi/connectors";

export const config = createConfig({
  chains: [mainnet, sepolia],
  connectors: [
    injected(), // MetaMask, Rabby, etc.
    walletConnect({
      projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
    }),
  ],
  transports: {
    [mainnet.id]: http(),
    [sepolia.id]: http(),
  },
});
```

#### **Tích hợp vào Next.js:**

```typescript
// app/layout.tsx
"use client";

import { WagmiProvider } from "wagmi";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { config } from "./config/wagmi";

const queryClient = new QueryClient();

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <WagmiProvider config={config}>
          <QueryClientProvider client={queryClient}>
            {children}
          </QueryClientProvider>
        </WagmiProvider>
      </body>
    </html>
  );
}
```

---

### 4.3. Connectors (Bộ kết nối)

#### **1️⃣ Injected Connector (MetaMask, Rabby)**

```typescript
import { injected } from "wagmi/connectors";

const injectedConnector = injected({
  shimDisconnect: true, // Xử lý disconnect đúng cách
});
```

**Sử dụng:**

```typescript
function ConnectButton() {
  const { connect } = useConnect();

  return (
    <button onClick={() => connect({ connector: injected() })}>
      Connect MetaMask
    </button>
  );
}
```

#### **2️⃣ WalletConnect (Mobile Wallets)**

```typescript
import { walletConnect } from "wagmi/connectors";

const walletConnectConnector = walletConnect({
  projectId: "YOUR_PROJECT_ID", // Lấy từ https://cloud.walletconnect.com
  metadata: {
    name: "My dApp",
    description: "My awesome dApp",
    url: "https://mydapp.com",
    icons: ["https://mydapp.com/logo.png"],
  },
  showQrModal: false, // Tự tạo UI QR code
});
```

---

### 4.4. Quy trình kết nối QR Code chi tiết

#### **Bước 1: Lấy WalletConnect Project ID**

1. Truy cập https://cloud.walletconnect.com
2. Tạo project mới
3. Copy Project ID

#### **Bước 2: Cấu hình Wagmi**

```typescript
import { createConfig } from "wagmi";
import { walletConnect } from "wagmi/connectors";

export const config = createConfig({
  chains: [sepolia], // ⚠️ Phải có ít nhất 1 public chain
  connectors: [
    walletConnect({
      projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
      showQrModal: false, // Tự xử lý QR
    }),
  ],
  // ...
});
```

**⚠️ Lưu ý quan trọng:**

- WalletConnect **KHÔNG hỗ trợ** localhost-only chains
- Phải thêm ít nhất 1 public chain (Sepolia, Mainnet)
- Có thể tự động switch về localhost sau khi kết nối

#### **Bước 3: Tạo URI và hiển thị QR Code**

```typescript
"use client";

import { QRCodeSVG } from "qrcode.react";
import { useEffect, useState } from "react";
import { useConnect } from "wagmi";

export default function QRCodeMethod() {
  const [uri, setUri] = useState<string>("");
  const { connect, connectors } = useConnect();

  const walletConnectConnector = connectors.find(
    (c) => c.id === "walletConnect"
  );

  useEffect(() => {
    if (!walletConnectConnector) return;

    // 🎯 QUAN TRỌNG: Đăng ký listener TRƯỚC khi connect
    const handleMessage = ({ type, data }: any) => {
      if (type === "display_uri" && typeof data === "string") {
        setUri(data); // URI chứa expiryTimestamp
      }
    };

    // Đăng ký listener
    walletConnectConnector.emitter.on("message", handleMessage);

    // Kích hoạt kết nối
    connect({ connector: walletConnectConnector });

    // Cleanup
    return () => {
      walletConnectConnector.emitter.off("message", handleMessage);
    };
  }, [walletConnectConnector, connect]);

  if (!uri) return <div>Loading QR Code...</div>;

  return <QRCodeSVG value={uri} size={256} />;
}
```

#### **Bước 4: Xử lý các sự kiện**

```typescript
useEffect(() => {
  if (!connector) return;

  // 1. Sự kiện nhận URI
  connector.emitter.on("message", ({ type, data }) => {
    if (type === "display_uri") {
      setUri(data);
    }
  });

  // 2. Sự kiện kết nối thành công
  connector.emitter.on("connect", async (data) => {
    console.log("Connected!", data);

    // Optional: Switch về localhost
    const provider = await connector.getProvider();
    try {
      await provider.request({
        method: "wallet_switchEthereumChain",
        params: [{ chainId: "0x7a69" }], // 31337 in hex
      });
    } catch (error) {
      console.error("Switch chain failed:", error);
    }
  });

  // 3. Sự kiện lỗi
  connector.emitter.on("error", (error) => {
    console.error("Connection error:", error);
  });

  // 4. Sự kiện ngắt kết nối
  connector.emitter.on("disconnect", () => {
    console.log("Disconnected");
    setUri(""); // Reset QR
  });

  // Cleanup
  return () => {
    connector.emitter.removeAllListeners();
  };
}, [connector]);
```

#### **Bước 5: Làm mới QR Code**

QR code có thời gian hết hạn (thường 5 phút). Để làm mới:

```typescript
const handleRefresh = async () => {
  setUri(""); // Xóa QR cũ

  // Ngắt kết nối cũ (nếu có)
  if (connector) {
    await connector.disconnect();
  }

  // Tạo kết nối mới
  connect({ connector: walletConnectConnector });
};
```

**⚠️ Lưu ý:**

- **KHÔNG** tự set `expiryTimestamp`
- Thư viện WalletConnect tự động tạo timestamp
- Muốn làm mới: Tạo yêu cầu kết nối MỚI

---

### 4.5. Gửi Giao dịch (Transactions)

#### **1️⃣ Read Contract (Không tốn gas)**

```typescript
import { useReadContract } from "wagmi";

function EmployeeStatus() {
  const { data: isEmployee } = useReadContract({
    address: "0x...",
    abi: contractABI,
    functionName: "isEmployee",
    args: ["0xUserAddress..."],
  });

  return <div>Employee status: {isEmployee ? "Yes" : "No"}</div>;
}
```

#### **2️⃣ Write Contract (Tốn gas) - Cách Đúng**

```typescript
import { useWriteContract, useSimulateContract } from "wagmi";

function CheckInButton() {
  // 🎯 Bước 1: Simulate (dry run)
  const { data: simulateData } = useSimulateContract({
    address: "0x...",
    abi: contractABI,
    functionName: "check",
    args: [],
  });

  // 🎯 Bước 2: Write (thực thi)
  const { writeContract, isPending, isSuccess } = useWriteContract();

  const handleCheckIn = () => {
    if (!simulateData?.request) {
      alert("Simulation failed! Transaction would revert.");
      return;
    }

    writeContract(simulateData.request);
  };

  return (
    <button onClick={handleCheckIn} disabled={isPending}>
      {isPending ? "Processing..." : "Check In"}
    </button>
  );
}
```

**💡 Tại sao cần `simulateContract`?**

| Không có Simulate             | Có Simulate                  |
| ----------------------------- | ---------------------------- |
| ❌ User ký → Revert → Mất gas | ✅ Pre-check → Chỉ ký nếu OK |
| ❌ UX tệ                      | ✅ UX tốt                    |
| ❌ Tốn tiền oan               | ✅ Tiết kiệm gas             |

#### **3️⃣ Write Contract với Parameters**

```typescript
function AddEmployeeButton() {
  const [address, setAddress] = useState("");
  const { writeContract } = useWriteContract();

  const handleAdd = () => {
    writeContract({
      address: "0x...",
      abi: contractABI,
      functionName: "addEmployee",
      args: [address],
    });
  };

  return (
    <div>
      <input
        value={address}
        onChange={(e) => setAddress(e.target.value)}
        placeholder="0x..."
      />
      <button onClick={handleAdd}>Add Employee</button>
    </div>
  );
}
```

#### **4️⃣ Wait for Transaction**

```typescript
import { useWaitForTransactionReceipt } from "wagmi";

function TransactionStatus() {
  const { writeContract, data: hash } = useWriteContract();

  const { isLoading, isSuccess } = useWaitForTransactionReceipt({
    hash,
  });

  return (
    <div>
      {isLoading && <p>⏳ Waiting for confirmation...</p>}
      {isSuccess && <p>✅ Transaction confirmed!</p>}
      {hash && <p>Hash: {hash}</p>}
    </div>
  );
}
```

---

### 4.6. Xử lý Errors

```typescript
import { BaseError, ContractFunctionRevertedError } from "viem";

function ErrorHandling() {
  const { writeContract, error } = useWriteContract();

  useEffect(() => {
    if (error) {
      if (error instanceof BaseError) {
        // Xử lý revert với custom error
        const revertError = error.walk(
          (err) => err instanceof ContractFunctionRevertedError
        );

        if (revertError instanceof ContractFunctionRevertedError) {
          const errorName = revertError.data?.errorName;
          console.log("Contract reverted:", errorName);
        }
      }

      // User rejected
      if (error.message.includes("User rejected")) {
        console.log("User cancelled transaction");
      }
    }
  }, [error]);
}
```

---

## 🚀 Phần 5: Xây dựng dApp Production-Grade

### 5.1. Quản lý Biến Môi trường trong Next.js

#### **Cấu trúc file .env:**

```bash
# .env.local (Development)
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=abc123...
NEXT_PUBLIC_CONTRACT_ADDRESS=0x5FbDB...
NEXT_PUBLIC_CHAIN_ID=31337
NEXT_PUBLIC_RPC_URL=http://localhost:8545

# Server-side only (KHÔNG có NEXT_PUBLIC_)
DATABASE_URL=postgresql://...
PRIVATE_KEY=0x...
API_SECRET=secret123
```

#### **Nguyên tắc NEXT*PUBLIC*:**

| Prefix         | Truy cập từ đâu     | Use case      | Ví dụ                      |
| -------------- | ------------------- | ------------- | -------------------------- |
| `NEXT_PUBLIC_` | ✅ Browser + Server | Public config | Contract address, Chain ID |
| Không prefix   | ❌ Chỉ Server       | Secrets       | Private key, API key       |

#### **Truy cập biến:**

```typescript
// ✅ Client-side (Browser)
const projectId = process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID;

// ✅ Server-side only (API routes, Server Components)
const privateKey = process.env.PRIVATE_KEY; // Không có NEXT_PUBLIC_

// ❌ LỖI: Truy cập server-side var từ client
const key = process.env.PRIVATE_KEY; // undefined trong browser!
```

#### **Type-safe env:**

```typescript
// env.ts
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID: string;
      NEXT_PUBLIC_CONTRACT_ADDRESS: `0x${string}`;
      NEXT_PUBLIC_CHAIN_ID: string;
      PRIVATE_KEY: `0x${string}`;
    }
  }
}

export {};
```

**⚠️ Lưu ý:**

- Phải **restart dev server** sau khi thay đổi `.env`
- Không commit `.env.local` vào Git
- Tạo `.env.example` cho team:

```bash
# .env.example
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=
NEXT_PUBLIC_CONTRACT_ADDRESS=
NEXT_PUBLIC_CHAIN_ID=
PRIVATE_KEY=
```

---

### 5.2. Kiến trúc Private Blockchain cho Production

Hardhat Node chỉ phù hợp cho development. Để triển khai thực tế:

#### **🏗️ Architecture Overview:**

```
┌─────────────────────────────────────────────────────────────┐
│                     Cloud Server (VPS)                       │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Geth (Go-Ethereum)                                │    │
│  │  - PoA (Proof of Authority) Consensus             │    │
│  │  - HTTP/WebSocket RPC: 8545/8546                  │    │
│  │  - NetworkID: 1337 (Custom)                       │    │
│  │  - Gas Price: 0 (Free transactions)              │    │
│  └────────────────────────────────────────────────────┘    │
│                          ↕                                  │
│  Public IP: 203.0.113.100                                  │
└─────────────────────────────────────────────────────────────┘
                          ↕
        ┌─────────────────────────────────────┐
        │   Company Employees (dApp Users)    │
        │  - Web Browser (Next.js dApp)       │
        │  - Mobile Wallet (WalletConnect)    │
        └─────────────────────────────────────┘
```

#### **Bước 1: Chọn Cloud Provider**

| Provider         | Giá/tháng | Ưu điểm                |
| ---------------- | --------- | ---------------------- |
| **AWS EC2**      | ~$10-50   | Scalable, nhiều region |
| **DigitalOcean** | ~$6-12    | Đơn giản, dễ setup     |
| **Linode**       | ~$5-10    | Giá rẻ                 |
| **Google Cloud** | ~$10-50   | Credit miễn phí $300   |

**Khuyến nghị:** DigitalOcean Droplet ($12/tháng, 2GB RAM)

#### **Bước 2: Setup Geth (Go-Ethereum)**

```bash
# 1. SSH vào server
ssh root@203.0.113.100

# 2. Cài đặt Geth
apt-get update
apt-get install software-properties-common
add-apt-repository -y ppa:ethereum/ethereum
apt-get update
apt-get install ethereum

# 3. Tạo genesis.json (PoA config)
cat > genesis.json << EOF
{
  "config": {
    "chainId": 1337,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "berlinBlock": 0,
    "londonBlock": 0,
    "clique": {
      "period": 5,
      "epoch": 30000
    }
  },
  "difficulty": "1",
  "gasLimit": "8000000",
  "extradata": "0x0000000000000000000000000000000000000000000000000000000000000000YOUR_SIGNER_ADDRESS0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "alloc": {
    "0xYOUR_ADDRESS": { "balance": "1000000000000000000000000" }
  }
}
EOF

# 4. Initialize chain
geth init --datadir ./data genesis.json

# 5. Tạo account
geth account new --datadir ./data
# Lưu lại password và address

# 6. Chạy node
geth --datadir ./data \
  --networkid 1337 \
  --http \
  --http.addr "0.0.0.0" \
  --http.port 8545 \
  --http.api "eth,net,web3,personal,admin,miner" \
  --http.corsdomain "*" \
  --allow-insecure-unlock \
  --unlock "0xYOUR_SIGNER_ADDRESS" \
  --password password.txt \
  --mine \
  --miner.etherbase "0xYOUR_SIGNER_ADDRESS" \
  console
```

#### **Bước 3: Bảo mật Server**

```bash
# 1. Firewall (chỉ cho phép port 8545 từ IP công ty)
ufw allow from COMPANY_IP to any port 8545
ufw allow 22/tcp  # SSH
ufw enable

# 2. Reverse Proxy với Nginx (Optional)
apt-get install nginx
cat > /etc/nginx/sites-available/blockchain << EOF
server {
    listen 80;
    server_name blockchain.company.com;

    location / {
        proxy_pass http://localhost:8545;
        proxy_set_header Host \$host;
    }
}
EOF
ln -s /etc/nginx/sites-available/blockchain /etc/nginx/sites-enabled/
systemctl restart nginx

# 3. SSL/TLS với Let's Encrypt
apt-get install certbot python3-certbot-nginx
certbot --nginx -d blockchain.company.com
```

#### **Bước 4: Cập nhật dApp**

```typescript
// config/wagmi.ts
import { defineChain } from "viem";

const companyChain = defineChain({
  id: 1337,
  name: "Company Blockchain",
  network: "company",
  nativeCurrency: {
    decimals: 18,
    name: "Ether",
    symbol: "ETH",
  },
  rpcUrls: {
    default: {
      http: ["https://blockchain.company.com"], // Hoặc http://203.0.113.100:8545
    },
    public: {
      http: ["https://blockchain.company.com"],
    },
  },
  blockExplorers: {
    default: {
      name: "Company Explorer",
      url: "https://explorer.company.com", // Optional
    },
  },
});

export const config = createConfig({
  chains: [companyChain],
  connectors: [injected(), walletConnect({ projectId: "..." })],
  transports: {
    [companyChain.id]: http(),
  },
});
```

#### **Bước 5: Nút "Thêm Mạng" tự động**

```typescript
function AddNetworkButton() {
  const handleAddNetwork = async () => {
    try {
      await window.ethereum.request({
        method: "wallet_addEthereumChain",
        params: [
          {
            chainId: "0x539", // 1337 in hex
            chainName: "Company Blockchain",
            nativeCurrency: {
              name: "Ether",
              symbol: "ETH",
              decimals: 18,
            },
            rpcUrls: ["https://blockchain.company.com"],
            blockExplorerUrls: ["https://explorer.company.com"],
          },
        ],
      });

      alert("Network added successfully!");
    } catch (error) {
      console.error("Failed to add network:", error);
    }
  };

  return (
    <button onClick={handleAddNetwork}>Add Company Network to Wallet</button>
  );
}
```

---

### 5.3. Sign-In with Ethereum (SIWE)

Xác thực người dùng bằng ví crypto, không cần password.

#### **Flow:**

```
1. User clicks "Sign in"
2. Frontend requests challenge from backend
3. Backend generates nonce + message
4. Frontend asks wallet to sign message
5. Frontend sends signature to backend
6. Backend verifies signature
7. Backend creates session (JWT/Cookie)
```

#### **Backend (Next.js API Route):**

```typescript
// app/api/auth/nonce/route.ts
import { generateNonce } from "siwe";

export async function GET() {
  const nonce = generateNonce();

  // Lưu nonce vào session/database
  // ...

  return Response.json({ nonce });
}

// app/api/auth/verify/route.ts
import { SiweMessage } from "siwe";

export async function POST(request: Request) {
  const { message, signature } = await request.json();

  try {
    const siweMessage = new SiweMessage(message);
    const fields = await siweMessage.verify({ signature });

    // Verify nonce từ database
    // ...

    // Tạo session
    const session = {
      address: fields.data.address,
      chainId: fields.data.chainId,
    };

    // Lưu session (JWT hoặc Cookie)
    // ...

    return Response.json({ ok: true, session });
  } catch (error) {
    return Response.json({ ok: false }, { status: 401 });
  }
}
```

#### **Frontend:**

```typescript
import { useAccount, useSignMessage } from "wagmi";
import { SiweMessage } from "siwe";

function SignInButton() {
  const { address, chainId } = useAccount();
  const { signMessageAsync } = useSignMessage();

  const handleSignIn = async () => {
    // 1. Lấy nonce
    const nonceRes = await fetch("/api/auth/nonce");
    const { nonce } = await nonceRes.json();

    // 2. Tạo message
    const message = new SiweMessage({
      domain: window.location.host,
      address,
      statement: "Sign in to My Company dApp",
      uri: window.location.origin,
      version: "1",
      chainId,
      nonce,
    });

    // 3. Sign message
    const signature = await signMessageAsync({
      message: message.prepareMessage(),
    });

    // 4. Verify
    const verifyRes = await fetch("/api/auth/verify", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ message, signature }),
    });

    const { ok, session } = await verifyRes.json();

    if (ok) {
      console.log("Signed in!", session);
    }
  };

  return <button onClick={handleSignIn}>Sign In with Ethereum</button>;
}
```

---

## 🎯 Phần 6: Best Practices & Tips

### 6.1. Smart Contract Security

```solidity
// ✅ GOOD
contract Secure {
    // 1. Checks-Effects-Interactions Pattern
    function withdraw() public {
        uint amount = balances[msg.sender];
        require(amount > 0, "No balance");

        balances[msg.sender] = 0;  // Effect TRƯỚC

        payable(msg.sender).transfer(amount);  // Interaction SAU
    }

    // 2. Dùng pull over push
    mapping(address => uint) public withdrawable;

    function claim() public {
        uint amount = withdrawable[msg.sender];
        withdrawable[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }

    // 3. Reentrancy guard
    bool private locked;

    modifier nonReentrant() {
        require(!locked, "Reentrant call");
        locked = true;
        _;
        locked = false;
    }
}
```

### 6.2. Gas Optimization

```solidity
contract GasOptimized {
    // ❌ BAD: storage read trong loop
    function sumBad() public view returns (uint) {
        uint sum = 0;
        for (uint i = 0; i < items.length; i++) {
            sum += items[i];  // Đọc storage mỗi lần
        }
        return sum;
    }

    // ✅ GOOD: cache vào memory
    function sumGood() public view returns (uint) {
        uint sum = 0;
        uint len = items.length;  // Cache length
        for (uint i = 0; i < len; i++) {
            sum += items[i];
        }
        return sum;
    }

    // ✅ GOOD: uint256 thay vì uint8
    uint256 public count;  // Cheaper
    uint8 public smallCount;  // Expensive (cần padding)

    // ✅ GOOD: Pack variables
    struct User {
        address wallet;  // 20 bytes
        uint96 balance;  // 12 bytes
        // Cùng 1 slot = tiết kiệm gas
    }
}
```

### 6.3. Frontend Best Practices

```typescript
// ✅ Handle all states
function MyComponent() {
  const { data, isLoading, isError, error } = useReadContract({...});

  if (isLoading) return <Spinner />;
  if (isError) return <Error message={error.message} />;
  if (!data) return <Empty />;

  return <Data value={data} />;
}

// ✅ Debounce user input
import { useDebouncedValue } from '@mantine/hooks';

function Search() {
  const [search, setSearch] = useState('');
  const [debounced] = useDebouncedValue(search, 500);

  const { data } = useReadContract({
    functionName: 'search',
    args: [debounced],  // Chỉ call khi user ngừng gõ
  });
}

// ✅ Optimistic updates
function LikeButton() {
  const [likes, setLikes] = useState(0);
  const { writeContract } = useWriteContract();

  const handleLike = () => {
    setLikes(prev => prev + 1);  // Update UI ngay

    writeContract({...}, {
      onError: () => setLikes(prev => prev - 1),  // Rollback nếu lỗi
    });
  };
}
```

---

## 📝 Phần 7: Tổng kết & Lộ trình học

### Checklist kiến thức đã nắm:

- ✅ **Blockchain Fundamentals**: State, History, Block, Consensus
- ✅ **Solidity**: Variables, Functions, Modifiers, Events, Mapping, Arrays
- ✅ **Hardhat**: Compile, Deploy, Test, Node
- ✅ **Viem & Wagmi**: Read/Write contract, Connectors, Hooks
- ✅ **WalletConnect**: QR Code flow, Event handling
- ✅ **Next.js Integration**: Env vars, API routes, Client/Server components
- ✅ **Production Setup**: Private chain, Geth, Cloud deployment
- ✅ **Authentication**: SIWE (Sign-In with Ethereum)

### Lộ trình tiếp theo:

1. **Advanced Solidity**

   - Upgradeable contracts (Proxy pattern)
   - ERC standards (ERC-20, ERC-721, ERC-1155)
   - Gas optimization techniques

2. **Security**

   - Common vulnerabilities (Reentrancy, Front-running)
   - Audit với Slither, Mythril
   - Formal verification

3. **Advanced dApp**

   - Subgraph (The Graph) cho indexing
   - IPFS integration
   - Multi-chain support

4. **DeFi Concepts**
   - AMM (Uniswap)
   - Lending protocols (Aave)
   - Yield farming

---

## 🔗 Resources

### Official Docs:

- Solidity: https://docs.soliditylang.org
- Hardhat: https://hardhat.org/docs
- Viem: https://viem.sh
- Wagmi: https://wagmi.sh
- WalletConnect: https://docs.walletconnect.com

### Learning:

- CryptoZombies: https://cryptozombies.io (Interactive Solidity)
- Ethernaut: https://ethernaut.openzeppelin.com (Security challenges)
- Scaffold-ETH: https://scaffoldeth.io (Boilerplates)

### Tools:

- Remix IDE: https://remix.ethereum.org
- Tenderly: https://tenderly.co (Debugging)
- OpenZeppelin Contracts: https://docs.openzeppelin.com/contracts

---

**Chúc mừng bạn đã hoàn thành hành trình học Blockchain Development! 🎉**

---

© 2025 hwHoai | [github.com/hwHoai](https://github.com/hwHoai) | License: MIT

```

```

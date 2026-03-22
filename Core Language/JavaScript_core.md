# BÍ KÍP ÔN TẬP JAVASCRIPT CORE TẦNG SÂU
*Tài liệu rà soát kiến thức cốt lõi chuẩn bị phỏng vấn vị trí Front-end / Full-stack Developer.*

## 1. Event Loop & Kiến trúc Bất đồng bộ (Asynchronous)

### Khái niệm cốt lõi:
- **JS là ngôn ngữ Đơn luồng (Single-threaded):** Chỉ có 1 **Call Stack**, xử lý từng việc một.
- **Web APIs:** Các dịch vụ của Browser (hoặc C++ trong Node.js) xử lý các tác vụ tốn thời gian (`setTimeout`, DOM, Fetch API).
- **Hàng đợi (Queues):**
  - **Microtask Queue (Hàng đợi VIP):** Dành riêng cho `Promise` (`.then`, `.catch`). Mức độ ưu tiên cao nhất.
  - **Macrotask Queue (Hàng đợi thường):** Dành cho `setTimeout`, `setInterval`.
- **Event Loop:** Liên tục kiểm tra. Khi Call Stack **TRỐNG**, nó sẽ dọn sạch toàn bộ Microtask Queue trước, sau đó mới bốc 1 task từ Macrotask Queue ném vào Call Stack.

### Code & Câu hỏi phỏng vấn:
**Hỏi:** "Thứ tự in ra màn hình của đoạn code sau là gì và giải thích tại sao?"

```javascript
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```
> **Đáp án:** `A -> D -> C -> B`
> **Giải thích:** `A` và `D` là code đồng bộ, vào Call Stack chạy ngay. `B` bị đẩy vào Macrotask Queue. `C` thuộc Promise nên bị đẩy vào Microtask Queue. Khi Call Stack trống, Event Loop ưu tiên dọn sạch Microtask (`C`) trước, rồi mới tới Macrotask (`B`).

## 2. Scope & Closures (Phạm vi và Bao đóng)

### Khái niệm cốt lõi:
- **Lexical Scope:** Phạm vi biến được quyết định lúc *viết code*, hàm con luôn có thể truy cập biến của môi trường bao ngoài nó.
- **Closure:** Một "lỗ hổng" hợp pháp. Khi một hàm con được return và gọi ở nơi khác, nó **vẫn giữ được tham chiếu** đến các biến của hàm cha, ngay cả khi hàm cha đã chạy xong và bị pop khỏi Call Stack.
- **Cơ chế bộ nhớ:** JS Engine "lén" bế các biến được Closure sử dụng từ Stack ném sang **Heap** để không bị Garbage Collector dọn dẹp.

### Code & Câu hỏi phỏng vấn:
**Hỏi:** "Đoạn code sau in ra gì? Làm sao để sửa nó in ra đúng `0, 1, 2` mà chỉ đổi 1 từ khóa?"
```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}
```
> **Đáp án ban đầu:** In ra `3, 3, 3`. Vì `var` không có Block Scope, cả 3 callback của setTimeout (chạy sau 1s) đều trỏ chung về một biến `i` duy nhất lúc này đã tăng lên 3.
> **Cách sửa:** Đổi `var` thành `let`. Vì `let` có Block Scope, mỗi vòng lặp tạo ra một vùng nhớ mới cho `i`, các callback sẽ tạo Closure ôm lấy các giá trị `0, 1, 2` riêng biệt.

## 3. Từ khóa `this` & Execution Context

### Khái niệm cốt lõi:
`this` trong JS không phụ thuộc vào nơi khai báo, mà phụ thuộc vào **CÁCH HÀM ĐƯỢC GỌI** lúc runtime.
1. **Default Binding:** Gọi độc lập (`fn()`) -> `this` là `window` (hoặc `undefined` trong strict mode).
2. **Implicit Binding:** Gọi qua object (`obj.fn()`) -> `this` là `obj` (kẻ đứng trước dấu chấm).
3. **Explicit Binding:** Ép buộc bằng tay.
   - `.call(obj, arg1, arg2)`: Ép `this` và gọi ngay.
   - `.apply(obj, [arg1, arg2])`: Ép `this`, gọi ngay, truyền tham số dạng mảng.
   - `.bind(obj)`: Ép `this`, **KHÔNG** gọi ngay, trả về một hàm mới.
4. **Lexical Binding (Arrow Function):** Mũi tên KHÔNG có `this`. Nó lấy `this` của môi trường bao ngoài (scope cha) gần nhất.

### Code & Câu hỏi phỏng vấn:
**Hỏi:** "Kết quả của 3 lời gọi hàm này là gì? Giải thích tại sao?"
```javascript
const agent = {
  name: "Ginny",
  greetNormal: function() { console.log(this.name); },
  greetArrow: () => { console.log(this.name); }
};

agent.greetNormal(); // (1)
agent.greetArrow(); // (2)

const detached = agent.greetNormal;
detached(); // (3)
```
> **Đáp án:** > (1) `"Ginny"` (Implicit: Gọi qua `agent`).
> (2) `undefined` (Lexical: Arrow function đi tìm `this` ở scope cha, object literal `{}` không tạo scope nên `this` trỏ ra Window).
> (3) `undefined` (Default: Gán ra biến ngoài rồi gọi độc lập, mất gốc).

## 4. Prototypal Inheritance (Kế thừa nguyên mẫu)

### Khái niệm cốt lõi:
- JS không có khái niệm Class như Java. Mọi thứ là Object.
- **Prototype Chain:** Khi truy cập một thuộc tính/phương thức, JS tìm ở object hiện tại. Không thấy, nó bò theo sợi dây `__proto__` lên Prototype của hàm tạo (Constructor), cứ thế lên đến `Object.prototype`.
- **Property Shadowing:** Nếu object con và Prototype cha có thuộc tính trùng tên, JS sẽ lấy của thằng con (vì tìm thấy trước) và che khuất thằng cha.

### Code & Câu hỏi phỏng vấn:
**Hỏi:** "Kết quả in ra là gì? So sánh 2 hàm `speak` có bằng nhau không?"
```javascript
function Robot(name) { this.name = name; }
Robot.prototype.speak = function() { console.log("I am " + this.name); };

const bot1 = new Robot("Alpha");
const bot2 = new Robot("Beta");

console.log(bot1.speak === bot2.speak); // (1)

bot1.speak = function() { console.log("Modified!"); };
bot1.speak(); // (2)
bot2.speak(); // (3)
```
> **Đáp án:** > (1) `true` (Cùng trỏ về chung 1 vùng nhớ trên Prototype).
> (2) `"Modified!"` (Bị che khuất - Shadowing bởi hàm gán trực tiếp cho bot1).
> (3) `"I am Beta"` (bot2 vẫn gọi hàm gốc trên Prototype).

## 5. Memory Management (Stack vs Heap)

### Khái niệm cốt lõi:
- **Kiểu nguyên thủy (Primitives):** Lưu ở **Stack**. Truyền theo Giá trị (Pass-by-value). Gán biến này cho biến kia là copy hẳn một bản mới.
- **Kiểu tham chiếu (Reference - Object/Array):** Dữ liệu thật lưu ở **Heap**, địa chỉ bộ nhớ lưu ở **Stack**. Gán biến này cho biến kia là trỏ chung về 1 cái kho. So sánh `===` là so sánh ĐỊA CHỈ, không so sánh ruột.

### Code & Câu hỏi phỏng vấn:
**Hỏi:** "Đoạn code sau in ra `true` hay `false`? Tại sao biến `user1` lại bị đổi tên?"
```javascript
let a = [1, 2, 3];
let b = [1, 2, 3];
console.log(a === b); // (1)

let user1 = { name: "A" };
let user2 = user1;
user2.name = "B";
console.log(user1.name); // (2)
```
> **Đáp án:**
> (1) `false`. Tạo ra 2 cái mảng mới ở Heap, `a` và `b` cầm 2 địa chỉ khác nhau.
> (2) `"B"`. `user1` và `user2` cùng cầm 1 địa chỉ trỏ vào chung 1 object ở Heap. `user2` sửa thì `user1` cũng bị ảnh hưởng.

## 6. ES6+ & Các trùm ẩn (Hoisting, Coercion, Event, Promises)

- **Hoisting:** `var` được kéo lên và khởi tạo `undefined`. `let/const` bị đẩy vào TDZ (Temporal Dead Zone - gọi trước khi khai báo sẽ ném lỗi).
- **Type Coercion:** Phép `+` ưu tiên nối Chuỗi (VD: `"5" + 2 = "52"`). Phép `- * /` ưu tiên ép về Số (VD: `"5" - 2 = 3`).
- **Event Delegation:** Gắn 1 listener vào thẻ cha để quản lý sự kiện cho hàng ngàn thẻ con (VD: Sơ đồ ghế). Dùng `event.target` để biết chính xác phần tử bị click, phân biệt với `event.currentTarget` (thẻ cha gắn listener).
- **ES6+ Toán tử:** - `||` (OR): Lấy bên phải nếu bên trái là Falsy (`0`, `""`, `false`, `null`, `undefined`).
  - `??` (Nullish): Lấy bên phải CHỈ KHI bên trái là `null` hoặc `undefined` (Tôn trọng số `0`).
- **Promise Concurrency:** Dùng `Promise.allSettled()` thay cho `Promise.all()` khi gọi nhiều API độc lập, tránh việc 1 API xịt làm sập toàn bộ request.

---
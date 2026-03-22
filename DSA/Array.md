# 🧠 DSA Handbook: Array & String Foundations

## 1. Kiến thức cốt lõi (Foundations)

### Array (Mảng)

- **Bản chất:** Ô nhớ liên tiếp, cùng kiểu dữ liệu.
- **Truy cập:** $O(1)$ qua chỉ số (index).
- **Chèn/Xóa:** $O(n)$ do phải dịch chuyển các phần tử phía sau.
- **Mảng động (Dynamic Array):** Tự động gấp đôi kích thước khi đầy ($O(n)$ cho thao tác resize nhưng trung bình là $O(1)$).

### String (Chuỗi)

- **Bản chất:** Mảng các ký tự.
- **Tính bất biến (Immutability):** Trong Python/Java, không thể sửa trực tiếp `s[i]`. Mọi thao tác thay đổi đều tạo ra chuỗi mới ($O(n)$).
- **Tối ưu:** Nên chuyển chuỗi thành List để xử lý, sau đó `.join()` lại.

---

## 2. Các mẫu tư duy chiến thuật (Design Patterns)

### Pattern 1: Frequency Counter (Bảng tần suất)

- **Khi nào dùng:** Đếm số lần xuất hiện, kiểm tra sự tồn tại, bài toán Anagram (đảo ngữ).
- **Công cụ:** Hash Map (Dictionary) hoặc mảng phụ.

```javascript
// Template: Kiểm tra Anagram
function frequencyCounter(s, t) {
  if (s.length !== t.length) return false;
  const count = {};
  for (const char of s) {
    count[char] = (count[char] || 0) + 1;
  }
  for (const char of t) {
    if ((count[char] || 0) === 0) return false;
    count[char]--;
  }
  return true;
}
```

### Pattern 2: Two Pointers - Đối xứng

- **Khi nào dùng:** Đảo ngược mảng/chuỗi, kiểm tra Palindrome (đối xứng).
- **Cách làm:** Một con trỏ đầu (`L`), một con trỏ cuối (`R`), thu hẹp dần.

```javascript
// Template: Đảo ngược mảng
function reverseArray(arr) {
  let L = 0,
    R = arr.length - 1;
  while (L < R) {
    [arr[L], arr[R]] = [arr[R], arr[L]];
    L++;
    R--;
  }
}
```

### Pattern 3: Two Pointers - Nhanh/Chậm (Read/Write)

- **Khi nào dùng:** Xóa phần tử trùng lặp, lọc dữ liệu tại chỗ (In-place) mà không tốn thêm bộ nhớ.
- **Cách làm:** `fast` đi tìm phần tử hợp lệ, `slow` đánh dấu vị trí ghi đè.

```javascript
// Template: Xóa phần tử tại chỗ
function removeElement(nums, val) {
  let slow = 0;
  for (let fast = 0; fast < nums.length; fast++) {
    if (nums[fast] !== val) {
      nums[slow] = nums[fast];
      slow++;
    }
  }
  return slow;
}
```

### Pattern 4: Sliding Window (Cửa sổ trượt)

- **Khi nào dùng:** Tìm chuỗi con, mảng con liên tiếp thỏa mãn điều kiện (tổng, độ dài...).
- **Cách làm:** Duy trì một khung, thêm phần tử bên phải, bớt phần tử bên trái.

```javascript
// Template: Tổng lớn nhất của k phần tử liên tiếp
function slidingWindow(nums, k) {
  let currSum = nums.slice(0, k).reduce((a, b) => a + b, 0);
  let maxSum = currSum;
  for (let i = 0; i < nums.length - k; i++) {
    currSum = currSum - nums[i] + nums[i + k];
    maxSum = Math.max(maxSum, currSum);
  }
  return maxSum;
}
```

### Pattern 5: Prefix Sum (Tổng tiền tố)

- **Khi nào dùng:** Cần tính tổng các đoạn mảng (Range Sum) nhiều lần.
- **Cách làm:** Tạo mảng tích lũy để biến phép tính tổng từ $O(n)$ về $O(1)$.

```javascript
// Template: Tạo mảng Prefix Sum
function getPrefixSum(nums) {
  const prefix = new Array(nums.length + 1).fill(0);
  for (let i = 0; i < nums.length; i++) {
    prefix[i + 1] = prefix[i] + nums[i];
  }
  return prefix;
}
// Tổng từ i đến j = prefix[j+1] - prefix[i]
```

---

## 3. Bảng tra cứu nhanh (Cheatsheet)

| Thao tác                  | Cấu trúc dữ liệu | Kỹ thuật ưu tiên             | Độ phức tạp             |
| ------------------------- | ---------------- | ---------------------------- | ----------------------- |
| **Truy cập phần tử**      | Array            | Indexing                     | $O(1)$                  |
| **Kiểm tra tồn tại**      | Array/String     | Hash Map                     | $O(1)$                  |
| **Xử lý chuỗi con**       | String           | Sliding Window               | $O(n)$                  |
| **Xử lý mảng đã sắp xếp** | Array            | Two Pointers / Binary Search | $O(n)$ hoặc $O(\log n)$ |
| **Đảo ngược/Đối xứng**    | Array/String     | Two Pointers (L & R)         | $O(n)$                  |

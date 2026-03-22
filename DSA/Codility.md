# Cẩm Nang Ôn Thi Codility (NAB Assessment)

Tài liệu này tổng hợp các bài toán kinh điển từ Lesson 1 đến Lesson 4 trên Codility, bao gồm cách tiếp cận ban đầu và cách tối ưu hóa để đạt 100% điểm hiệu năng (Performance) $\mathcal{O}(N)$ hoặc $\mathcal{O}(1)$ bộ nhớ.

## Lesson 1: Iterations (Vòng lặp)

### Bài toán: Binary Gap
Tìm khoảng trống nhị phân (dãy số `0` liên tiếp) dài nhất nằm trọn giữa hai số `1` trong biểu diễn nhị phân của một số nguyên `N`.

**Cách tiếp cận phổ thông (Dùng hàm String - Ít tối ưu thuật toán):**
*(Cách này dễ viết nhưng tốn bộ nhớ lưu chuỗi và thời gian gọi hàm tích hợp sẵn)*
```javascript
function solution(N) {
    // Chuyển số nguyên thành chuỗi nhị phân (vd: 9 -> "1001")
    const binaryStr = N.toString(2); 
    
    let maxGap = 0;
    let currentGap = 0;
    let counting = false; // Cờ đánh dấu đã gặp số 1 đầu tiên chưa

    for (let i = 0; i < binaryStr.length; i++) {
        if (binaryStr[i] === '1') {
            if (counting) {
                maxGap = Math.max(maxGap, currentGap);
            }
            counting = true; // Bắt đầu đếm khi gặp số 1
            currentGap = 0;  // Reset bộ đếm
        } else {
            if (counting) {
                currentGap++; // Tăng khoảng trống nếu đang trong cờ đếm
            }
        }
    }
    return maxGap;
}

```

**Code Tối ưu 100% Điểm (Toán học / Dịch bit):**
*(Cách này không cần tạo chuỗi, dùng trực tiếp phép chia lấy nguyên và lấy dư để chạy cực nhanh $\mathcal{O}(\log N)$)*

```javascript
function solution(N) {
    // Bước 1: Cắt bỏ các số 0 ở cuối cùng bên phải (vì không được bọc bởi số 1)
    while (N > 0 && N % 2 === 0) {
        N = Math.floor(N / 2); // Tương đương phép dịch bit N >>= 1
    }
    
    let maxGap = 0;
    let currentGap = 0;
    
    // Bước 2: Duyệt các bit còn lại từ phải sang trái
    while (N > 0) {
        if (N % 2 === 1) {
            // Gặp số 1: chốt sổ khoảng trống và cập nhật kỷ lục
            if (currentGap > maxGap) {
                maxGap = currentGap;
            }
            currentGap = 0;
        } else {
            // Gặp số 0: tăng biến đếm
            currentGap++;
        }
        
        N = Math.floor(N / 2); // Cắt bỏ bit vừa xét
    }
    
    return maxGap;
}

```

* **Tại sao tối ưu:** Không dùng hàm chuyển đổi chuỗi (string conversion). Dịch bit và chia nguyên giúp thuật toán chạy sát với phần cứng nhất, độ phức tạp thời gian $\mathcal{O}(\log N)$.

## Lesson 2: Arrays (Mảng)

### Bài toán 1: CyclicRotation

Dịch chuyển các phần tử trong mảng `A` sang phải `K` lần.

**Code của tôi (Bị lỗi Time Limit với mảng lớn):**

```javascript
function solution(A, K) {
    if(K === 0 || K === A.length) {
        return A
    }
    for (let i = 0; i < K; i++ ) {
        const last = A[A.length - 1];
        A.unshift(last);
        A.pop();
    }
    return A
}

```

**Code Tối ưu 100% Điểm:**

```javascript
function solution(A, K) {
    if (A.length === 0 || K === 0 || K % A.length === 0) {
        return A;
    }
    const actualK = K % A.length;
    const tail = A.slice(-actualK); 
    const head = A.slice(0, A.length - actualK);
    return [...tail, ...head]; 
}

```

* **Tại sao tối ưu:** Lệnh `unshift()` trong vòng lặp đẩy độ phức tạp lên $\mathcal{O}(N \times K)$. Cắt mảng bằng `slice()` và dùng toán tử Spread (`...`) giúp giải quyết bài toán chỉ trong 1 thao tác $\mathcal{O}(N)$.

### Bài toán 2: OddOccurrencesInArray

Tìm phần tử lẻ loi không có cặp trùng lặp trong mảng.

**Code của tôi (Tốn bộ nhớ tạo Object):**

```javascript
function solution(A) {
    const dic = {}
    for(let i = 0; i < A.length; i++) {
        dic[A[i]] = dic[A[i]] + 1 || 1
    }
    return parseInt(Object.entries(dic).filter(([key, val])  => val == 1)[0][0])
}

```

**Code Tối ưu 100% Điểm (Bitwise XOR):**

```javascript
function solution(A) {
    let result = 0;
    for (let i = 0; i < A.length; i++) {
        result ^= A[i]; 
    }
    return result;
}

```

* **Tại sao tối ưu:** Giảm độ phức tạp Không gian từ $\mathcal{O}(N)$ xuống $\mathcal{O}(1)$. Các số giống nhau khi XOR (`^`) sẽ tự triệt tiêu thành `0`, phần tử lẻ loi tự động lòi ra ở kết quả cuối cùng.

---

## Lesson 4: Counting Elements (Đếm phần tử)

### Bài toán 1: PermCheck

Kiểm tra xem mảng `A` (độ dài `N`) có phải là một hoán vị chứa đầy đủ các số từ `1` đến `N` hay không.

**Code của tôi (Lỗi logic tổng & Includes vòng lặp ngầm):**

```typescript
function solution(A: number[]): number {
    const N = A.length
    let seen = new Array(N).fill(false)
    for(let i = 0; i < N; i++) {
        if(seen[A[i] - 1] === true) return 0
        if(A[i] <= N) seen[A[i] - 1] = true
    }
    if(seen.includes(false)) return 0
    return 1
}

```

**Code Tối ưu 100% Điểm (Early Exit):**

```typescript
function solution(A: number[]): number {
    const N = A.length;
    const seen = new Array(N).fill(false);

    for(let i = 0; i < N; i++) {
        if(A[i] < 1 || A[i] > N || seen[A[i] - 1] === true) {
            return 0; 
        }
        seen[A[i] - 1] = true;
    }
    return 1;
}

```

* **Tại sao tối ưu:** Gộp mọi rào cản (số quá nhỏ, số quá lớn, số trùng lặp) vào một câu `if` để thoát sớm. Bỏ hàm `includes()` ở cuối giúp tránh việc phải duyệt mảng `seen` thêm một lần nữa.

### Bài toán 2: MaxCounters

Cập nhật hệ thống máy đếm. Nếu $A[i] \le N$: tăng counter. Nếu $A[i] = N + 1$: gán mọi counter bằng giá trị lớn nhất hiện tại.

**Code của tôi (Lỗi Time Limit do dùng hàm .map):**

```typescript
function solution(N: number, A: number[]): number[] {
    let couters = new Array(N).fill(0)
    let maxCouterValue = 0
    const nA = A.length
    for (let i = 0; i < nA; i++) {
        if(A[i] <= N) {
            couters[A[i] - 1] ++
            if(maxCouterValue < couters[A[i] - 1 ]) maxCouterValue = couters[A[i] - 1]
            continue
        }
        couters = couters.map(e => maxCouterValue) // Gây lỗi TLE
    }
    return couters
}

```

**Code Tối ưu 100% Điểm (Lazy Evaluation - Cập nhật lười):**

```typescript
function solution(N: number, A: number[]): number[] {
    let counters = new Array(N).fill(0);
    let maxVal = 0;  
    let lastMax = 0; 

    for (let i = 0; i < A.length; i++) {
        if (A[i] <= N) {
            let idx = A[i] - 1;
            if (counters[idx] < lastMax) counters[idx] = lastMax; 
            counters[idx]++; 
            if (counters[idx] > maxVal) maxVal = counters[idx];
        } else {
            lastMax = maxVal; // Chỉ cập nhật mốc đáy, O(1)
        }
    }

    // Quét lần cuối những counter chưa được chạm tới
    for (let i = 0; i < N; i++) {
        if (counters[i] < lastMax) counters[i] = lastMax;
    }

    return counters;
}

```

* **Tại sao tối ưu:** Thay vì ghi đè cả mảng $\mathcal{O}(N)$ mỗi khi có lệnh Max Counter, ta chỉ lưu lại "mức sàn" (`lastMax`). Ai thực sự cần tăng điểm mới được kéo lên mức sàn đó, giảm tổng thời gian xuống còn $\mathcal{O}(N + M)$.

### Bài toán 3: MissingInteger

Tìm số nguyên dương nhỏ nhất bị thiếu trong mảng.

**Code của tôi (Lỗi mảng rỗng do sort & tốn CPU):**

```typescript
function solution(A: number[]): number {
    const N = A.length // Lỗi: lấy N trước khi filter
    let lastNum = 1
    A = A.filter(e => e > 0).sort((a, b) => a - b) // O(N log N)
    for (let i = 0; i < N; i++) {
        if(A[i] == lastNum) {
            lastNum = A[i] + 1 
        }
    }
    return lastNum
}

```

**Code Tối ưu 100% Điểm (Cấu trúc Set):**

```typescript
function solution(A: number[]): number {
    const numSet = new Set(A);
    let smallest = 1;
    while (numSet.has(smallest)) {
        smallest++;
    }
    return smallest;
}

```

* **Tại sao tối ưu:** Hàm `sort()` tốn $\mathcal{O}(N \log N)$. Đổ dữ liệu vào cấu trúc `Set` giúp thời gian tra cứu `has()` đạt $\mathcal{O}(1)$, đẩy tốc độ tổng thể của thuật toán lên ngưỡng hoàn hảo $\mathcal{O}(N)$. Không cần bận tâm đến số âm hay trùng lặp.
---

## Lesson 5: Prefix Sums & Math Trick (Tổng tiền tố)

### Bài toán 1: PassingCars
Đếm số xe hướng Đông (0) đi ngang qua xe hướng Tây (1).

**Code Tối ưu 100% Điểm (Đếm dồn - Prefix Sums):**
```typescript
function solution(A: number[]): number {
    let zerosSeen = 0, passingCars = 0;
    
    // Quét mảng đúng 1 lần từ trái sang phải O(N)
    for (let i = 0; i < A.length; i++) {
        if (A[i] === 0) {
            zerosSeen++; // Ghi nhận thêm 1 xe đi hướng Đông
        } else {
            passingCars += zerosSeen; // Xe 1 (Tây) cắt mặt TẤT CẢ xe 0 trước đó
            
            // Bẫy của Codility: Trả về -1 nếu vượt mốc 1 tỷ
            if (passingCars > 1000000000) return -1; 
        }
    }
    return passingCars;
}
```

### Bài toán 2: GenomicRangeQuery
Tìm nucleotide nhỏ nhất trong đoạn DNA. (A=1, C=2, G=3, T=4). Dữ liệu truy vấn cực lớn.

**Code Tối ưu 100% Điểm (Prefix Sums - Lập bảng thống kê):**
```typescript
function solution(S: string, P: number[], Q: number[]): number[] {
    const N = S.length;
    // Mảng lưu trữ tổng số lượng A, C, G tính đến vị trí hiện tại
    const countA = new Array(N + 1).fill(0);
    const countC = new Array(N + 1).fill(0);
    const countG = new Array(N + 1).fill(0);
    
    // Pha 1: Lập bảng thống kê O(N)
    for (let i = 0; i < N; i++) {
        countA[i + 1] = countA[i] + (S[i] === 'A' ? 1 : 0);
        countC[i + 1] = countC[i] + (S[i] === 'C' ? 1 : 0);
        countG[i + 1] = countG[i] + (S[i] === 'G' ? 1 : 0);
    }
    
    const result = [];
    
    // Pha 2: Truy vấn O(M) bằng phép trừ lớp 1
    for (let i = 0; i < P.length; i++) {
        const start = P[i], end = Q[i] + 1; // +1 do mảng thống kê bị dịch phải
        
        // Trừ điểm cuối cho điểm đầu, nếu > 0 tức là đoạn đó CÓ chứa nucleotide này
        if (countA[end] - countA[start] > 0) result.push(1);
        else if (countC[end] - countC[start] > 0) result.push(2);
        else if (countG[end] - countG[start] > 0) result.push(3);
        else result.push(4); // Không có A, C, G thì chắc chắn min là T (4)
    }
    return result;
}
```

### Bài toán 3: MinAvgTwoSlice
Tìm vị trí bắt đầu của lát cắt có trung bình cộng nhỏ nhất. Bẫy vòng lặp lồng nhau $\mathcal{O}(N^2)$.

**Code Tối ưu 100% Điểm (Quy tắc Toán học độ dài 2 và 3):**
```typescript
function solution(A: number[]): number {
    let minAvg = Infinity, minIndex = 0;
    
    // Định lý: Lát cắt lớn luôn được tạo từ các lát cắt nhỏ dài 2 và 3.
    // Nên chỉ cần kiểm tra các lát cắt dài 2 và 3 là đủ, kéo thời gian về O(N).
    for (let i = 0; i < A.length - 1; i++) {
        // Kiểm tra lát cắt dài 2
        let avg2 = (A[i] + A[i + 1]) / 2;
        if (avg2 < minAvg) { minAvg = avg2; minIndex = i; }

        // Kiểm tra lát cắt dài 3 (phải rào điều kiện tránh index out of bounds)
        if (i < A.length - 2) {
            let avg3 = (A[i] + A[i + 1] + A[i + 2]) / 3;
            if (avg3 < minAvg) { minAvg = avg3; minIndex = i; }
        }
    }
    return minIndex;
}
```
## Lesson 7: Stacks and Queues (Ngăn xếp)

### Bài toán: Brackets
Kiểm tra xem chuỗi ngoặc `{[()]}` có được lồng nhau hợp lệ không. Nếu dùng hàm `replace` chuỗi liên tục sẽ bị sập Time Limit.

**Code Tối ưu 100% Điểm (Dùng Stack LIFO - Vào sau ra trước):**
```typescript
function solution(S: string): number {
    // Độ dài lẻ chắc chắn không thể ghép cặp hoàn chỉnh
    if (S.length % 2 !== 0) return 0; 
    
    const stack: string[] = [];
    
    // Dùng Hash Map (Object) để tra cứu cặp ngoặc đối xứng O(1)
    const pairs: Record<string, string> = {
        ')': '(',
        ']': '[',
        '}': '{'
    };

    // Duyệt mảng O(N)
    for (let i = 0; i < S.length; i++) {
        const char = S[i];
        
        // Gặp ngoặc MỞ -> Nhét vào đầu rổ (Stack)
        if (char === '(' || char === '[' || char === '{') {
            stack.push(char);
        } 
        // Gặp ngoặc ĐÓNG -> Rút đỉnh rổ ra kiểm tra
        else {
            if (stack.length === 0) return 0; // Có đóng mà không có mở -> Sai
            
            const top = stack.pop(); // Lấy phần tử trên cùng ra
            
            if (pairs[char] !== top) return 0; // Lệch pha (Ví dụ: '}' đi với '[') -> Sai
        }
    }
    
    // Nếu quét xong mà rổ rỗng -> Hoàn hảo. Nếu còn dư -> Thiếu ngoặc đóng
    return stack.length === 0 ? 1 : 0;
}
```
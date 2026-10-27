# 02. JavaScript - Hàm, scope, closure

## 1. Ba cách viết hàm

```js
// 1. Function declaration
function add(a, b) { return a + b; }

// 2. Function expression
const sub = function (a, b) { return a - b; };

// 3. Arrow function
const mul = (a, b) => a * b;
```

Khác nhau:
- Declaration: hoist cả hàm, gọi trước dòng khai báo vẫn chạy.
- Expression và arrow: chỉ biến `const` được hoist (nằm trong TDZ), gọi trước sẽ lỗi.
- Arrow function không có `this` riêng, không có `arguments`, không dùng với `new` được.
- Muốn lấy mọi tham số trong arrow function thì dùng `...args`.

```js
sayHi();                       // chạy được
function sayHi() { console.log("hi"); }

sayBye();                      // ReferenceError: Cannot access 'sayBye' before initialization
const sayBye = () => console.log("bye");

const nums = (...args) => args.length;
nums(1, 2, 3);                 // 3

const P = () => {};
// new P();                    // TypeError: P is not a constructor
```

## 2. Execution Context và Call Stack

- Khi chương trình bắt đầu, JS tạo Global Execution Context đầu tiên.
- Mỗi lần gọi hàm, JS tạo thêm 1 Function Execution Context và push vào Call Stack.
- Hàm chạy xong thì context bị pop ra khỏi stack.

```js
function a() { b(); }
function b() { c(); }
function c() { console.log("xong"); }
a();
```

```
Bước 1      Bước 2     Bước 3     Bước 4 (in "xong")
            |  b  |    |  c  |
  | a |     |  a  |    |  b  |
  | GEC |   | GEC |    |  a  |    ...rồi pop dần c, b, a
                       | GEC |
```

- Đệ quy không có điều kiện dừng thì stack đầy, báo `RangeError: Maximum call stack size exceeded` (stack overflow).

```js
function loop() { loop(); }
loop(); // RangeError: Maximum call stack size exceeded
```

## 3. Scope và Scope Chain

- Lexical scope: phạm vi của biến do vị trí viết code quyết định, không phụ thuộc chỗ gọi hàm.
- Khi tìm biến, JS tìm từ trong ra ngoài: scope hiện tại, rồi scope hàm cha, rồi global. Hết mà không thấy thì báo `ReferenceError`.

```js
const g = "global";
function outer() {
  const o = "outer";
  function inner() {
    const i = "inner";
    console.log(i, o, g);  // inner outer global (tìm dần ra ngoài)
  }
  inner();
}
outer();
console.log(o);            // ReferenceError: o is not defined
```

## 4. Closure

Closure là hàm con nhớ được biến của hàm cha, kể cả khi hàm cha đã chạy xong.

```js
function makeCounter() {
  let count = 0;           // biến này được hàm bên dưới giữ lại
  return function () {
    count++;
    return count;
  };
}

const c1 = makeCounter();
console.log(c1());  // 1
console.log(c1());  // 2
const c2 = makeCounter();
console.log(c2());  // 1  (mỗi lần gọi makeCounter có count riêng)
```

Vì sao `count` không mất:
- Bình thường hàm cha xong thì biến bị xóa khỏi stack.
- Nhưng hàm con vẫn dùng `count`, nên V8 chuyển `count` sang Heap để giữ lại.

```
makeCounter() chạy xong -> pop khỏi Call Stack
                            count = 0 chuyển sang Heap
c1 (hàm con) ------------------> vẫn trỏ tới count
```

Dùng closure để làm gì:
- Giấu dữ liệu (private): bên ngoài không đọc/sửa trực tiếp được.
- Function factory: tạo ra hàm theo cấu hình.

```js
function createTax(rate) {
  return (price) => price + price * rate;
}
const vat10 = createTax(0.1);
const vat30 = createTax(0.3);
console.log(vat10(100000)); // 110000
console.log(vat30(100000)); // 130000
```

```js
function createVault(start) {
  let balance = start;
  return {
    deposit(n) { balance += n; },
    getBalance() { return balance; },
  };
}
const v = createVault(1000);
v.deposit(500);
console.log(v.getBalance()); // 1500
console.log(v.balance);      // undefined, không truy cập trực tiếp được
```

## Lưu ý / hay nhầm

- Closure giữ biến, không giữ giá trị tại thời điểm tạo. Biến đổi sau thì hàm con thấy giá trị mới.
- Dùng `var` trong vòng `for` với `setTimeout` sẽ in ra cùng 1 giá trị, đổi sang `let` là mỗi vòng có biến riêng.

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 0); // 3 3 3
for (let j = 0; j < 3; j++) setTimeout(() => console.log(j), 0); // 0 1 2
```

- Closure giữ object lớn mà không giải phóng (gán vào biến global hoặc event listener không gỡ) thì Garbage Collector không dọn được, gây tốn RAM.
- Declaration hoist cả hàm, expression/arrow thì không.

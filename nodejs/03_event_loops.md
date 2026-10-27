# 3. NodeJS - Event loop

## 1. Các phase của event loop

Event loop của NodeJS do libuv điều khiển, chạy lặp qua các phase theo thứ tự:

```
   +---------------------------+
+->|  1. Timers                |  setTimeout, setInterval
|  +---------------------------+
|  |  2. Pending callbacks     |  một số callback I/O bị hoãn
|  +---------------------------+
|  |  3. Idle, prepare         |  nội bộ, không cần quan tâm
|  +---------------------------+
|  |  4. Poll                  |  lấy kết quả I/O mới (file, mạng)
|  +---------------------------+
|  |  5. Check                 |  setImmediate
|  +---------------------------+
|  |  6. Close callbacks       |  vd socket.on("close")
+--+---------------------------+
```

Cần nhớ chính: Timers, Poll, Check.

## 2. Giữa các phase: nextTick và Promise

Trước khi sang phase kế tiếp, NodeJS chạy hết các hàng đợi này (theo thứ tự ưu tiên):
1. `process.nextTick()`: ưu tiên cao nhất.
2. Promise `.then/.catch/.finally`: chạy sau nextTick.

```js
setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
Promise.resolve().then(() => console.log("promise"));
process.nextTick(() => console.log("nextTick"));
console.log("sync");
// in ra: sync, nextTick, promise, rồi timeout/immediate
```

Thứ tự: code đồng bộ, nextTick, Promise, rồi mới đến các phase (timeout, immediate).

## 3. setTimeout(0) và setImmediate

- `setTimeout(fn, 0)`: callback vào phase Timers.
- `setImmediate(fn)`: callback vào phase Check (ngay sau Poll).

Ở code chính (ngoài I/O), thứ tự giữa 2 cái này không cố định (phụ thuộc máy).

Trong callback của I/O thì `setImmediate` luôn chạy trước:

```js
const fs = require("fs");
fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
// luôn in: immediate, rồi timeout
```

Vì đang ở phase Poll, phase kế tiếp là Check (setImmediate), sau đó mới quay vòng lại Timers.

## 4. So với event loop trong trình duyệt

- Trình duyệt: Call Stack, rồi Microtask, rồi Macrotask (đã ghi ở note JS 05).
- NodeJS: chia macrotask thành nhiều phase (Timers, Poll, Check...), và có thêm `process.nextTick`.

## Lưu ý / hay nhầm

- `setTimeout(fn, 0)` không chạy ngay, ít nhất 1ms và phải chờ tới phase Timers.
- `process.nextTick` gọi đệ quy liên tục sẽ làm event loop đứng (không sang phase khác được).
- `nextTick` chạy trước Promise, dù cả 2 đều gọi sớm.
- Code đồng bộ chạy lâu sẽ làm trễ mọi timer, vì event loop không quay được.

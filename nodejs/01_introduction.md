# 1. NodeJS - Giới thiệu

## 1. NodeJS là gì

- NodeJS là môi trường chạy JavaScript ngoài trình duyệt (JavaScript runtime), xây trên V8 engine của Chrome.
- Nhờ NodeJS mà JS chạy được ở phía server, đọc ghi file, mở server mạng...
- NodeJS không phải ngôn ngữ và không phải framework, nó là môi trường chạy.

## 2. Kiến trúc: V8 + libuv

```
        Code JavaScript của mình
                  |
        Node.js (API: fs, http, ...)
          |                    |
     V8 engine              libuv (C++)
  - dịch JS ra mã máy     - event loop
  - quản lý Heap, Stack   - thread pool
                          - I/O mạng, file
```

- V8: đọc code JS rồi biên dịch thành mã máy, quản lý bộ nhớ (Heap, Call Stack).
- libuv: thư viện C++ lo việc I/O (file, mạng), chạy event loop, và có thread pool (mặc định 4 thread) để làm các việc nặng ở phía sau.

## 3. Single-thread và Non-blocking I/O

- Code JavaScript của mình chạy trên 1 luồng chính (main thread).
- Khi gặp I/O (đọc file, gọi DB, gọi mạng), NodeJS không đứng chờ mà giao cho libuv làm. Xong thì callback được đưa về cho luồng chính chạy tiếp.

Ví dụ so sánh:

```js
const fs = require("fs");

// Blocking: dừng ở đây đến khi đọc xong
const data = fs.readFileSync("a.txt", "utf8");
console.log("1. doc xong:", data);

// Non-blocking: không chờ, chạy tiếp ngay
fs.readFile("a.txt", "utf8", (err, data) => {
  console.log("3. doc xong (bat dong bo)");
});
console.log("2. chay tiep, khong cho");
// thứ tự in: 1, 2, 3
```

Dòng "2" in trước dòng "3" vì `readFile` giao việc cho libuv rồi chạy tiếp, còn callback chờ đến lượt.

## 4. Chạy file NodeJS

```bash
node app.js          # chạy 1 file
node                 # vào REPL gõ code thử
node -v              # xem phiên bản
```

## Lưu ý / hay nhầm

- Single-thread không có nghĩa NodeJS làm được 1 việc 1 lúc: I/O vẫn chạy song song ở phía sau nhờ libuv.
- Code tính toán nặng (vòng lặp lớn, mã hóa) vẫn chạy trên luồng chính và sẽ làm cả server đứng. Non-blocking chỉ giúp với I/O.
- Không dùng hàm `...Sync` (như `readFileSync`) trong server đang phục vụ nhiều người, vì nó chặn cả luồng chính.
- Trong NodeJS không có `window`, `document`; đối tượng toàn cục là `global` (hoặc `globalThis`).

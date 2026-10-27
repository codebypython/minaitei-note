# 4. NodeJS - Callbacks

## 1. Callback là gì

Callback là hàm truyền vào hàm khác, được gọi lại sau khi một việc xong (thường là việc bất đồng bộ: đọc file, gọi API, truy vấn DB).

```js
function chao(ten, callback) {
  console.log("Xin chao " + ten);
  callback();
}
chao("An", () => console.log("xong"));
// Xin chao An
// xong
```

Nhờ callback mà NodeJS không phải đứng chờ I/O.

## 2. Error-first callback

Quy ước trong NodeJS:
- Tham số đầu tiên luôn là lỗi (`err`). Không có lỗi thì là `null`.
- Các tham số sau là kết quả (`data`).

```js
const fs = require("fs");

fs.readFile("data.txt", "utf8", (err, data) => {
  if (err) {
    console.error("Loi doc file:", err.message);
    return;                        // dừng lại, không chạy tiếp
  }
  console.log("Noi dung:", data);
});
```

Tự viết hàm theo chuẩn này:

```js
function layUser(id, callback) {
  setTimeout(() => {
    if (id <= 0) return callback(new Error("ID khong hop le"));
    callback(null, { id, name: "An" });
  }, 300);
}

layUser(1, (err, user) => {
  if (err) return console.log("Loi:", err.message);
  console.log(user);              // { id: 1, name: 'An' }
});

layUser(-5, (err, user) => {
  if (err) return console.log("Loi:", err.message);   // Loi: ID khong hop le
});
```

Lý do `err` đứng đầu: bắt người viết phải kiểm tra lỗi trước khi dùng dữ liệu.

## 3. Callback hell

Khi nhiều việc bất đồng bộ phải làm lần lượt, callback lồng vào nhau thành hình kim tự tháp:

```js
doA((err, a) => {
  if (err) return console.log(err);
  doB(a, (err, b) => {
    if (err) return console.log(err);
    doC(b, (err, c) => {
      if (err) return console.log(err);
      console.log(c);
    });
  });
});
```

Vấn đề: khó đọc, lặp code xử lý lỗi ở mỗi tầng, khó sửa.

Cách giảm:

1. Tách thành các hàm có tên:

```js
function xuLyC(err, c) { if (err) return console.log(err); console.log(c); }
function xuLyB(err, b) { if (err) return console.log(err); doC(b, xuLyC); }
function xuLyA(err, a) { if (err) return console.log(err); doB(a, xuLyB); }
doA(xuLyA);
```

2. Đổi sang Promise bằng `util.promisify`, rồi dùng `async/await`:

```js
const util = require("util");
const fs = require("fs");
const readFile = util.promisify(fs.readFile);

async function load() {
  try {
    const data = await readFile("data.txt", "utf8");
    console.log(data);
  } catch (err) {
    console.error(err.message);
  }
}
load();
```

Hoặc dùng sẵn `fs.promises`:

```js
const fsp = require("fs").promises;
fsp.readFile("data.txt", "utf8").then(console.log).catch(console.error);
```

## Lưu ý / hay nhầm

- Quên `return` sau khi gọi `callback(err)` thì code bên dưới vẫn chạy tiếp, dễ gọi callback 2 lần.
- `try/catch` ngoài không bắt được lỗi ném ra trong callback bất đồng bộ.
- Không để hàm vừa gọi callback đồng bộ, vừa gọi bất đồng bộ tùy trường hợp (thứ tự chạy sẽ khó đoán).
- Callback của bên thứ ba có thể bị gọi nhiều lần hoặc không gọi, đây là điểm yếu của callback so với Promise.

# 5. NodeJS - Event emitter

## 1. Ý tưởng

NodeJS hoạt động theo hướng sự kiện (event-driven). Nhiều module có sẵn (`http`, `fs` stream, `net`) đều dựa trên class `EventEmitter` của module `events`.

Thay vì truyền callback, một bên "phát" sự kiện (emit), các bên khác "lắng nghe" (on).

## 2. Các hàm chính

- `emitter.on(tên, fn)`: lắng nghe, chạy mỗi lần sự kiện được phát.
- `emitter.once(tên, fn)`: chỉ lắng nghe 1 lần đầu, sau đó tự gỡ.
- `emitter.emit(tên, ...dữ liệu)`: phát sự kiện, truyền dữ liệu cho listener.
- `emitter.off(tên, fn)` hoặc `removeListener`: gỡ 1 listener.
- `emitter.removeAllListeners(tên)`: gỡ hết listener của 1 sự kiện.

## 3. Ví dụ cơ bản

```js
const EventEmitter = require("events");
const shop = new EventEmitter();

shop.on("dat_hang", (id, tien) => {
  console.log(`Gui mail xac nhan don #${id}: ${tien}d`);
});

shop.once("dat_hang", () => {
  console.log("Don dau tien trong phien!");
});

shop.emit("dat_hang", 101, 250000);
shop.emit("dat_hang", 102, 100000);
```

Kết quả:

```
Gui mail xac nhan don #101: 250000d
Don dau tien trong phien!
Gui mail xac nhan don #102: 100000d
```

Lần phát thứ 2 không in dòng "Don dau tien" vì `once` đã tự gỡ.

## 4. Tự tạo class kế thừa EventEmitter

```js
class Download extends EventEmitter {
  start() {
    this.emit("start");
    setTimeout(() => this.emit("done", "file.zip"), 500);
  }
}

const d = new Download();
d.on("start", () => console.log("bat dau tai"));
d.on("done", (name) => console.log("tai xong", name));
d.start();
```

## 5. Sự kiện error

Phát sự kiện `error` mà không có listener nào bắt thì NodeJS ném lỗi và làm app dừng (crash).

```js
const e = new EventEmitter();
e.emit("error", new Error("loi"));   // không có listener -> crash chương trình
```

Cách an toàn: luôn đăng ký listener cho `error`.

```js
e.on("error", (err) => console.log("Bat duoc loi:", err.message));
e.emit("error", new Error("loi"));   // Bat duoc loi: loi
```

## 6. Listener chạy đồng bộ

`emit()` gọi lần lượt từng listener theo thứ tự đăng ký, ngay lúc đó (đồng bộ).

```js
const e = new EventEmitter();
e.on("x", () => console.log("A"));
e.on("x", () => console.log("B"));
console.log("truoc");
e.emit("x");           // in A, B ngay tại đây
console.log("sau");
// in ra: truoc, A, B, sau
```

Muốn chạy bất đồng bộ phải tự bọc bằng `setImmediate` hoặc `process.nextTick`.

## Lưu ý / hay nhầm

- Không có listener cho sự kiện thì `emit` bình thường vẫn không lỗi (chỉ riêng `error` mới crash).
- `on` mỗi lần gọi lại đăng ký thêm 1 listener. Gọi `on` trong vòng lặp hoặc trong hàm chạy nhiều lần sẽ tích tụ listener (memory leak).
- Mặc định tối đa 10 listener cho 1 sự kiện, vượt quá thì NodeJS in warning (chỉnh bằng `setMaxListeners`).
- Muốn gỡ listener bằng `off` thì phải truyền đúng hàm đã đăng ký, hàm ẩn danh mới viết lại thì không gỡ được.
- Listener nào ném lỗi đồng bộ cũng làm `emit` dừng, các listener sau không chạy.

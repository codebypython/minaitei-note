# 05. JavaScript - Bất đồng bộ, Event Loop, Promise

## 1. Vì sao cần bất đồng bộ

- JS chỉ có 1 Call Stack, tức là cùng lúc chỉ chạy 1 việc (single-thread).
- Nếu việc chậm (gọi API, đọc file, hẹn giờ) chạy đồng bộ thì cả chương trình đứng chờ.
- Cách giải quyết: giao việc chậm cho môi trường bên ngoài (Web API của trình duyệt, hoặc libuv của Node), xong thì callback được đưa lại vào hàng đợi.

## 2. Event Loop

```
            Call Stack
                |
     gặp setTimeout / fetch / đọc file
                v
     Web API / libuv (xử lý ngầm)
                | xong việc
                v
   +-----------------------------+
   | Microtask Queue             |  Promise.then, queueMicrotask
   +-----------------------------+
   | Macrotask Queue             |  setTimeout, setInterval, I/O
   +-----------------------------+
                |
       Event Loop lấy việc đưa lại vào Call Stack
```

Thứ tự Event Loop:
1. Chạy hết code đồng bộ trong Call Stack.
2. Chạy hết toàn bộ Microtask Queue (kể cả microtask mới sinh ra trong lúc chạy).
3. Lấy 1 macrotask đưa vào chạy.
4. Quay lại bước 2.

Tóm lại: đồng bộ, rồi microtask, rồi macrotask.

```js
console.log("1");
setTimeout(() => console.log("5"), 0);
Promise.resolve().then(() => console.log("3"));
Promise.resolve().then(() => console.log("4"));
console.log("2");
// in ra: 1 2 3 4 5
```

Giải thích: `1` và `2` là code đồng bộ chạy trước. `3`, `4` là microtask chạy ngay sau khi stack rỗng. `5` là macrotask nên chạy cuối, dù timeout là 0ms.

## 3. Promise

3 trạng thái:
- `pending`: đang chờ.
- `fulfilled`: thành công (`resolve(data)`).
- `rejected`: thất bại (`reject(err)`).

Đã fulfilled hoặc rejected thì khóa luôn, không đổi nữa.

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => resolve("xong"), 100);
});

p.then((v) => console.log(v))        // "xong"
 .catch((e) => console.log(e))
 .finally(() => console.log("luon chay"));
```

Mỗi `.then` trả về Promise mới, nên nối chuỗi được. Lỗi ở đâu trong chuỗi cũng rơi xuống `.catch` gần nhất.

## 4. async / await

`async/await` là cách viết khác của Promise, đọc như code đồng bộ.

```js
async function load() {
  try {
    const res = await fetch("https://example.com/api");
    const data = await res.json();
    return data;
  } catch (err) {
    console.log("loi:", err.message);
  }
}
```

- Hàm `async` luôn trả về Promise.
- `await` tạm dừng hàm đó (không chặn cả chương trình) cho đến khi Promise xong.
- Bắt lỗi bằng `try/catch`.

## 5. Chạy nhiều Promise cùng lúc

- `Promise.all([...])`: chờ tất cả thành công, có 1 cái lỗi thì reject ngay (fail-fast).
- `Promise.allSettled([...])`: chờ tất cả xong, không bao giờ reject, trả về trạng thái từng cái.

```js
const ok = new Promise((r) => setTimeout(() => r("A"), 50));
const bad = new Promise((_, rej) => setTimeout(() => rej(new Error("loi B")), 80));

Promise.all([ok, bad]).catch((e) => console.log(e.message));   // loi B

Promise.allSettled([ok, bad]).then((rs) => console.log(rs));
// [ { status: 'fulfilled', value: 'A' },
//   { status: 'rejected', reason: Error: loi B } ]
```

## 6. Xử lý lỗi bất đồng bộ

```js
// try/catch KHÔNG bắt được lỗi trong callback bất đồng bộ
try {
  setTimeout(() => { throw new Error("loi"); }, 0);
} catch (e) {
  console.log("khong bao gio vao day");
}

// Dùng Promise + catch, hoặc async/await + try/catch thì bắt được
async function run() {
  try {
    await Promise.reject(new Error("loi"));
  } catch (e) {
    console.log("bat duoc:", e.message);
  }
}
```

## Lưu ý / hay nhầm

- Quên `await` thì nhận về Promise chứ không phải giá trị.
- `await` trong vòng `for` chạy tuần tự từng cái. Muốn chạy song song thì dùng `Promise.all`.
- `setTimeout(fn, 0)` không chạy ngay lập tức, vẫn phải chờ hết code đồng bộ và microtask.
- Promise không có `.catch` mà bị reject sẽ gây `UnhandledPromiseRejection`.
- Vòng lặp vô hạn microtask (microtask sinh microtask mãi) sẽ làm macrotask không bao giờ được chạy.

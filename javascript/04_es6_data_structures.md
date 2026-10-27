# 04. JavaScript - Map, Set, WeakMap và toán tử ES6

## 1. Map

Map là tập các cặp key-value, key có thể là mọi kiểu (object, function, số...).

```js
const m = new Map();
const keyObj = { id: 1 };
m.set(keyObj, "du lieu A");
m.set("name", "An");
m.set(404, "Not Found");

console.log(m.get(keyObj));   // "du lieu A"
console.log(m.has(404));      // true
console.log(m.size);          // 3
m.delete(404);

for (const [k, v] of m) console.log(k, v);
```

So với object thường:
- Key của object chỉ là string hoặc symbol, key của Map là mọi kiểu.
- Map giữ đúng thứ tự thêm vào.
- Map có `.size`, object phải dùng `Object.keys(obj).length`.
- Map duyệt trực tiếp bằng `for...of`.

## 2. Set

Set là tập các giá trị không trùng nhau.

```js
const s = new Set([1, 2, 2, 3, 3]);
console.log(s);          // Set { 1, 2, 3 }
s.add(4);
console.log(s.has(2));   // true
console.log(s.size);     // 4

const unique = [...new Set([1, 1, 2, 3, 3])];
console.log(unique);     // [1, 2, 3]
```

- `set.has(x)` tìm nhanh hơn `array.includes(x)` khi dữ liệu lớn (Set tìm theo hash, mảng phải duyệt từng phần tử).
- Dùng Set để lọc phần tử trùng trong mảng.

Giao và hợp 2 Set:

```js
const A = new Set(["JS", "TS", "Node"]);
const B = new Set(["TS", "Go"]);
const giao = [...A].filter((x) => B.has(x));   // ["TS"]
const hop = [...new Set([...A, ...B])];        // ["JS","TS","Node","Go"]
```

## 3. WeakMap và WeakSet

- Key của `WeakMap` bắt buộc là object.
- Tham chiếu của WeakMap tới key là tham chiếu yếu: không giữ object sống.
- Nếu không còn biến nào khác trỏ tới object key, Garbage Collector dọn object đó, và mục trong WeakMap tự mất.

```js
const meta = new WeakMap();
let session = { id: "S1" };
meta.set(session, { ip: "192.168.1.1" });
console.log(meta.get(session));   // { ip: "192.168.1.1" }

session = null;   // không còn tham chiếu mạnh -> GC dọn được, mục trong WeakMap cũng biến mất
```

So sánh với Map thường:

```
Map:      map --(mạnh)--> object   => object không bao giờ bị dọn khi map còn sống (rò rỉ RAM)
WeakMap:  map --(yếu)---> object   => object bị dọn khi hết tham chiếu khác
```

Hạn chế:
- WeakMap không có `.size`, không duyệt được bằng `for...of`, vì lúc nào GC chạy là không biết trước.

Dùng khi:
- Gắn dữ liệu phụ (metadata) cho object mà không muốn làm object sống lâu hơn.
- Cache theo object, object hết thì cache tự dọn.

## 4. Destructuring

```js
const res = { data: { userId: 101, username: "kien" }, status: 200 };
const { data: { userId: id, username }, status, timestamp = 0 } = res;
console.log(id, username, status, timestamp);   // 101 kien 200 0

const [first, , third] = [10, 20, 30];
console.log(first, third);   // 10 30
```

- `userId: id` là đổi tên biến.
- `timestamp = 0` là giá trị mặc định khi không có trường đó.

## 5. Optional chaining ?.

Gặp `null` hoặc `undefined` thì dừng và trả về `undefined`, không báo lỗi.

```js
const user = { profile: null };
console.log(user.profile.city);     // TypeError: Cannot read properties of null
console.log(user?.profile?.city);   // undefined
user.sayHi?.();                     // không có hàm thì bỏ qua, không lỗi
```

## 6. ?? và ||

- `||`: lấy vế phải nếu vế trái là falsy (`0`, `""`, `false`, `null`, `undefined`, `NaN`).
- `??`: chỉ lấy vế phải nếu vế trái là `null` hoặc `undefined`.

```js
const port = 0;
console.log(port || 3000);   // 3000  (sai, vì 0 là cổng hợp lệ)
console.log(port ?? 3000);   // 0     (đúng)

console.log("" || "DEV");    // "DEV"
console.log("" ?? "DEV");    // ""
```

## Lưu ý / hay nhầm

- Key của Map so sánh theo tham chiếu: `new Map().set({a:1}, 1).get({a:1})` ra `undefined` vì 2 object khác địa chỉ.
- WeakMap/WeakSet chỉ nhận object làm key, đưa số hay chuỗi vào sẽ lỗi `TypeError`.
- Không dùng `||` để đặt giá trị mặc định khi giá trị hợp lệ có thể là `0`, `""`, `false`.
- `?.` chỉ chặn `null/undefined`, không chặn các lỗi khác.

# 01. TypeScript - Kiểu dữ liệu, interface, hàm

## 1. Khai báo kiểu và để TS tự suy luận

```ts
let count = 0;             // TS tự hiểu là number (inference)
let name: string = "An";   // ghi rõ kiểu (annotation)
let data: string;          // chưa gán giá trị thì nên ghi kiểu, không thì thành any
```

Nên ghi kiểu rõ ở:
- tham số của hàm
- biến khai báo chưa gán giá trị
- chỗ mô tả dữ liệu API (interface, type)

Còn biến có giá trị khởi tạo rõ ràng thì để TS tự suy luận cho gọn.

## 2. Array và Tuple

```ts
const nums: number[] = [1, 2, 3];
const names: Array<string> = ["An", "Binh"];

// Tuple: mảng có độ dài và kiểu từng vị trí cố định
const point: [number, number] = [10, 20];
const user: [number, string] = [1, "An"];
// const bad: [number, string] = ["An", 1];   // lỗi: sai thứ tự kiểu
```

- Array: mọi phần tử cùng kiểu, độ dài tùy ý.
- Tuple: từng vị trí có kiểu riêng, độ dài cố định.

## 3. Object type

```ts
const student: { id: number; name: string; email?: string } = {
  id: 1,
  name: "An",
};
```

- `email?` là thuộc tính tùy chọn, có thể có hoặc không.
- `readonly id: number` thì không gán lại được sau khi tạo.

## 4. Enum

```ts
enum Role {
  Admin,      // 0
  User,       // 1
  Guest,      // 2
}
console.log(Role.Admin);   // 0
console.log(Role[1]);      // "User"

enum Status {
  Pending = "PENDING",
  Done = "DONE",
}
console.log(Status.Done);  // "DONE"
```

- Enum số tự tăng từ 0, tra ngược được (`Role[1]`).
- Enum chuỗi dùng khi cần giá trị đọc được (hay dùng khi gửi lên API).
- Cách nhẹ hơn là union kiểu chuỗi: `type Status = "PENDING" | "DONE"`.

## 5. Interface và type

```ts
interface User {
  id: number;
  name: string;
}

interface Admin extends User {
  permissions: string[];
}

type Id = string | number;           // type dùng được cho union
type Point = [number, number];       // và tuple
type AdminUser = User & { role: string };  // intersection
```

Khác nhau chính:
- `interface` có thể khai báo trùng tên để tự gộp (declaration merging), `type` thì báo lỗi `Duplicate identifier`.
- `interface` mở rộng bằng `extends`, `type` ghép bằng `&`.
- `type` viết được union, tuple, kiểu nguyên thủy, `interface` thì không.

```ts
interface Config { name: string }
interface Config { port: number }     // tự gộp thành { name, port }
const c: Config = { name: "app", port: 3000 };
```

Quy ước hay dùng: object/class dùng `interface`, union và tuple dùng `type`.

## 6. Kiểu cho hàm

```ts
function add(a: number, b: number): number {
  return a + b;
}

const greet = (name: string): string => `Hi ${name}`;

function log(msg: string): void {      // void: không trả về gì
  console.log(msg);
}

function fail(msg: string): never {    // never: không bao giờ chạy xong bình thường
  throw new Error(msg);
}
```

Khác nhau giữa `void` và `never`:
- `void`: hàm chạy xong và không trả giá trị.
- `never`: hàm không bao giờ kết thúc bình thường (luôn ném lỗi hoặc lặp vô hạn).

Tham số tùy chọn và mặc định:

```ts
function hello(name: string, title?: string, times: number = 1) {}
```

## 7. Function overloading

Dùng khi 1 hàm nhận nhiều kiểu đầu vào và kiểu trả về phụ thuộc đầu vào.

```ts
function format(x: string): string;
function format(x: number): string;
function format(x: string | number): string {
  return typeof x === "string" ? x.toUpperCase() : x.toFixed(2);
}

format("abc");   // "ABC"
format(3.14159); // "3.14"
// format(true); // lỗi: không khớp overload nào
```

- Các dòng đầu chỉ là chữ ký (signature) để người gọi nhìn thấy.
- Dòng cuối là phần cài đặt thật, không được gọi trực tiếp từ bên ngoài.

## 8. any, unknown, never

- `any`: tắt kiểm tra kiểu, biến lấy từ `any` cũng thành `any` (lây lan lỗi).
- `unknown`: chưa biết kiểu, phải kiểm tra kiểu trước khi dùng.
- `never`: tập rỗng, không có giá trị nào.

```ts
let a: any = "hello";
a.foo.bar();              // không báo lỗi lúc compile, chạy sẽ crash

let u: unknown = "hello";
// u.toUpperCase();       // lỗi compile: phải kiểm tra kiểu trước
if (typeof u === "string") {
  u.toUpperCase();        // được
}
```

Dùng `unknown` thay `any` cho dữ liệu chưa rõ kiểu (ví dụ `JSON.parse`).

## Lưu ý / hay nhầm

- Gán object literal có thuộc tính thừa trực tiếp vào biến có kiểu thì TS báo lỗi (excess property check). Gán qua biến trung gian thì không báo.

```ts
interface U { id: number }
const u1: U = { id: 1, age: 20 };       // lỗi: 'age' không có trong U
const raw = { id: 1, age: 20 };
const u2: U = raw;                      // được
```

- Kiểu của TS chỉ có lúc compile, chạy ra JavaScript là mất hết (type erasure).
- Enum số có thể gán số bất kỳ, dễ lỗi ngầm. Cẩn thận khi dùng.
- Tuple chỉ kiểm tra khi khai báo, `push` vẫn có thể làm dài thêm.

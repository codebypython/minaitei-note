# 02. TypeScript - Type narrowing, discriminated union, type guard

## 1. Type narrowing là gì

Biến có thể mang kiểu rộng (`string | number`, `unknown`). Narrowing là TS dựa vào các câu `if`/`switch` để thu hẹp về kiểu cụ thể trong từng nhánh.

```ts
function show(x: string | number) {
  if (typeof x === "string") {
    x.toUpperCase();   // trong nhánh này x là string
  } else {
    x.toFixed(2);      // nhánh này x là number
  }
}
```

## 2. Các cách thu hẹp kiểu

typeof (cho kiểu nguyên thủy):

```ts
function len(x: string | string[]) {
  return typeof x === "string" ? x.length : x.length;
}
```

instanceof (kiểm tra theo class):

```ts
function printDate(d: Date | string) {
  if (d instanceof Date) return d.getFullYear();
  return d.toUpperCase();
}
```

in (kiểm tra có thuộc tính hay không):

```ts
interface Cat { meow(): void }
interface Dog { bark(): void }
function speak(p: Cat | Dog) {
  if ("meow" in p) p.meow();
  else p.bark();
}
```

Truthy check (loại `null`/`undefined`):

```ts
function hello(name: string | null) {
  if (name) console.log(name.toUpperCase());
}
```

## 3. Discriminated union

Gồm:
1. Nhiều kiểu object ghép bằng `|`.
2. Mỗi kiểu có 1 thuộc tính chung với giá trị literal khác nhau (thường tên `status`, `type`, `kind`).
3. Dùng `switch` theo thuộc tính đó để chia nhánh.

Cách viết không tốt: cho phép trạng thái vô lý (vừa có data, vừa có error).

```ts
interface BadResponse {
  isLoading: boolean;
  data?: string;
  error?: string;
}
```

Cách viết tốt:

```ts
type Response =
  | { status: "LOADING" }
  | { status: "SUCCESS"; data: string }
  | { status: "ERROR"; error: string };

function render(r: Response) {
  switch (r.status) {
    case "LOADING": return "Dang tai...";
    case "SUCCESS": return r.data;      // ở đây r có data
    case "ERROR":   return r.error;     // ở đây r có error
  }
}
```

Lợi ích: không tạo ra trạng thái sai ngay từ lúc viết kiểu, TS lỗi ngay từ compile.

## 4. Exhaustive check với never

Mục đích: khi thêm trạng thái mới vào union mà quên xử lý ở đâu đó thì TS báo lỗi.

```ts
function render(r: Response): string {
  switch (r.status) {
    case "LOADING": return "Dang tai...";
    case "SUCCESS": return r.data;
    case "ERROR":   return r.error;
    default: {
      const check: never = r;   // nếu còn case chưa xử lý, dòng này lỗi compile
      return check;
    }
  }
}
```

Ví dụ thêm `{ status: "REFUNDED" }` vào type mà quên viết case thì ở `default`, `r` không còn là `never` nên báo lỗi: `Type '{ status: "REFUNDED" }' is not assignable to type 'never'`.

## 5. Custom type guard (từ khóa is)

Hàm tự viết để kiểm tra lúc chạy và báo cho TS biết kiểu. Hay dùng với dữ liệu `unknown` (JSON, API).

```ts
interface Product {
  id: number;
  title: string;
  price: number;
}

function isProduct(data: unknown): data is Product {
  return (
    typeof data === "object" &&
    data !== null &&
    "id" in data &&
    "title" in data &&
    "price" in data &&
    typeof (data as Product).price === "number"
  );
}

function handle(input: unknown) {
  if (isProduct(input)) {
    console.log(input.title, input.price);   // trong đây input là Product
  } else {
    console.log("du lieu khong hop le");
  }
}
```

- Kiểu trả về `data is Product` báo cho TS: nếu hàm trả `true` thì `data` là `Product`.
- TS tin hoàn toàn vào hàm này, nên điều kiện kiểm tra phải đúng.

## Lưu ý / hay nhầm

- `typeof null` là `"object"`, nên khi check object phải thêm `data !== null`.
- `typeof` không phân biệt được array và object (cả 2 là `"object"`), dùng `Array.isArray`.
- Type guard viết sai (trả `true` bừa) thì TS vẫn tin, lỗi sẽ xảy ra lúc chạy.
- Discriminated union phải dùng thuộc tính literal (`"LOADING"`), kiểu `string` chung chung thì không thu hẹp được.

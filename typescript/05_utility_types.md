# 05. TypeScript - Utility types

Utility types là các kiểu có sẵn của TS, dùng để biến đổi 1 kiểu thành kiểu khác mà không phải viết lại.

Dùng chung interface này cho các ví dụ:

```ts
interface User {
  id: number;
  name: string;
  email: string;
  phone?: string;
}
```

## 1. Partial<T>

Biến mọi thuộc tính thành tùy chọn. Hay dùng cho dữ liệu cập nhật từng phần (PATCH).

```ts
type UpdateUser = Partial<User>;
// { id?: number; name?: string; email?: string; phone?: string }

const patch: UpdateUser = { phone: "0905123456" };   // chỉ gửi trường cần đổi
```

## 2. Required<T>

Ngược lại với Partial: bỏ hết dấu `?`, bắt buộc có đủ.

```ts
type FullUser = Required<User>;
// phone cũng bắt buộc phải có
```

## 3. Readonly<T>

Mọi thuộc tính chỉ đọc, không gán lại được.

```ts
const u: Readonly<User> = { id: 1, name: "An", email: "a@x.com" };
// u.name = "Binh";   // lỗi: Cannot assign to 'name' because it is a read-only property
```

## 4. Pick<T, K>

Chọn ra vài thuộc tính từ `T`.

```ts
type PublicUser = Pick<User, "id" | "name">;
// { id: number; name: string }
const p: PublicUser = { id: 1, name: "An" };
```

## 5. Omit<T, K>

Bỏ đi vài thuộc tính khỏi `T`.

```ts
type UserNoEmail = Omit<User, "email">;
// { id: number; name: string; phone?: string }
```

Pick và Omit làm ngược nhau: Pick liệt kê cái giữ lại, Omit liệt kê cái bỏ đi. Cái nào liệt kê ít hơn thì dùng cái đó.

## 6. Record<K, V>

Tạo object có key thuộc `K` và value kiểu `V`.

```ts
type Role = "ADMIN" | "MOD" | "STUDENT";

const permissions: Record<Role, string[]> = {
  ADMIN: ["CREATE", "READ", "UPDATE", "DELETE"],
  MOD: ["READ", "UPDATE"],
  STUDENT: ["READ"],
};
// thiếu 1 role là lỗi compile
```

## 7. ReturnType<T>

Lấy kiểu trả về của 1 hàm.

```ts
function getUser() {
  return { id: 1, name: "An" };
}
type U = ReturnType<typeof getUser>;   // { id: number; name: string }
```

## 8. as const

Khóa giá trị thành literal chỉ đọc, hay dùng cho cấu hình và danh sách cố định.

```ts
const ENV = {
  API_URL: "https://api.example.com",
  TIMEOUT: 5000,
} as const;
// ENV.TIMEOUT = 1;   // lỗi: read-only

const METHODS = ["GET", "POST", "PUT"] as const;
type Method = (typeof METHODS)[number];   // "GET" | "POST" | "PUT"
```

- Không có `as const` thì `"GET"` bị suy luận là `string`, mất thông tin literal.

## Kết hợp

```ts
type CreateUserDto = Omit<User, "id">;                   // tạo mới: chưa có id
type UpdateUserDto = Partial<Omit<User, "id">>;          // sửa: bỏ id, các trường tùy chọn
type UserSummary = Readonly<Pick<User, "id" | "name">>;  // chỉ đọc, chỉ 2 trường
```

## Lưu ý / hay nhầm

- Utility types chỉ có lúc compile, không kiểm tra dữ liệu lúc chạy. Dữ liệu từ API vẫn phải tự kiểm tra.
- `Partial` chỉ làm tùy chọn tầng ngoài, object lồng bên trong không đổi.
- `Readonly` chỉ chặn gán lại tầng ngoài. Mảng bên trong vẫn `push` được nếu không dùng `readonly string[]`.
- `Omit` không báo lỗi nếu ghi sai tên thuộc tính, `Pick` thì có báo. Gõ sai tên cho `Omit` sẽ không có tác dụng mà không cảnh báo.

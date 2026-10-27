# 04. TypeScript - Generics và ràng buộc

## 1. Generics là gì

Cho phép viết hàm/class dùng với nhiều kiểu mà vẫn giữ được kiểm tra kiểu. Kiểu được truyền vào như tham số, thường ký hiệu `<T>`.

Dùng `any` thì mất kiểm tra:

```ts
function first(arr: any[]): any { return arr[0]; }
const x = first([1, 2, 3]);   // x là any, mất thông tin
```

Dùng generic thì giữ được kiểu:

```ts
function first<T>(arr: T[]): T { return arr[0]; }
const a = first([1, 2, 3]);       // a: number
const b = first(["x", "y"]);      // b: string
const c = first<boolean>([true]); // truyền kiểu rõ ràng
```

- TS thường tự suy luận `T` từ đối số, không cần ghi `<number>`.

## 2. Generic với interface và class

```ts
interface Box<T> {
  value: T;
}
const nBox: Box<number> = { value: 5 };
const sBox: Box<string> = { value: "hi" };

class Stack<T> {
  private items: T[] = [];
  push(item: T) { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
}
const s = new Stack<number>();
s.push(1);
// s.push("a");   // lỗi: "a" không phải number
```

## 3. Ràng buộc bằng extends

Mặc định `T` là kiểu gì cũng được. Muốn bắt buộc `T` phải có thuộc tính nào đó thì dùng `extends`.

```ts
interface HasId { id: number }

function logId<T extends HasId>(item: T): void {
  console.log(item.id);
}

logId({ id: 1, name: "An" });   // được
// logId({ name: "An" });       // lỗi: thiếu id
```

## 4. Ràng buộc bằng keyof

- `keyof T`: union tên các thuộc tính của `T`.
- `K extends keyof T`: `K` phải là tên thuộc tính có thật của `T`.
- `T[K]`: kiểu của thuộc tính đó.

```ts
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "An", age: 21 };
const n = getProp(user, "name");    // n: string
const a = getProp(user, "age");     // a: number
// getProp(user, "salary");         // lỗi: "salary" không có trong user
```

Ví dụ lấy 1 cột từ mảng object:

```ts
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((i) => i[key]);
}

const scores = [
  { subject: "Web", score: 9 },
  { subject: "DB", score: 8 },
];
console.log(pluck(scores, "score"));    // [9, 8]
console.log(pluck(scores, "subject"));  // ["Web", "DB"]
```

## 5. Generic Repository

Mẫu hay gặp ở backend: 1 class dùng chung để lưu/lấy dữ liệu cho nhiều loại entity.

```ts
interface BaseEntity {
  id: number;
}

class Repository<T extends BaseEntity> {
  private items = new Map<number, T>();

  save(item: T) { this.items.set(item.id, item); }
  findById(id: number): T | undefined { return this.items.get(id); }
  findAll(): T[] { return Array.from(this.items.values()); }
  delete(id: number): boolean { return this.items.delete(id); }
}

interface User extends BaseEntity { name: string }

const repo = new Repository<User>();
repo.save({ id: 1, name: "An" });
console.log(repo.findById(1));    // { id: 1, name: "An" }
console.log(repo.findById(99));   // undefined
```

- `T extends BaseEntity` đảm bảo mọi entity đều có `id`, nên class dùng `item.id` an toàn.
- `findById` trả `T | undefined` vì có thể không tìm thấy, phải kiểm tra trước khi dùng.

## Lưu ý / hay nhầm

- Generic chỉ có lúc compile, chạy ra JS là mất, nên không dùng `T` để `new T()` hay `typeof T` lúc chạy.
- `<T>` trong file `.tsx` hay bị nhầm với JSX, cần viết `<T,>` hoặc `<T extends unknown>`.
- Quên ràng buộc thì không dùng được thuộc tính của `T` (`item.id` lỗi nếu `T` không có `extends`).
- Không phải cứ có hàm nhận nhiều kiểu là cần generic, nếu chỉ có 2-3 kiểu cố định thì union đơn giản hơn.

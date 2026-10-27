# 03. TypeScript - Class, access modifier, abstract

## 1. Class cơ bản

```ts
class Student {
  public name: string;
  readonly id: number;

  constructor(name: string, id: number) {
    this.name = name;
    this.id = id;
  }

  intro(): string {
    return `${this.name} (${this.id})`;
  }
}

const s = new Student("An", 1);
console.log(s.intro());   // An (1)
// s.id = 2;              // lỗi: id là readonly
```

## 2. Access modifier

- `public` (mặc định): truy cập được ở mọi nơi.
- `protected`: trong class và class con, không truy cập được từ bên ngoài.
- `private`: chỉ trong chính class đó.
- `readonly`: chỉ gán lúc khai báo hoặc trong constructor.

```ts
class Account {
  public owner: string;
  protected balance: number;
  private pin: string;

  constructor(owner: string, balance: number, pin: string) {
    this.owner = owner;
    this.balance = balance;
    this.pin = pin;
  }
}

class VipAccount extends Account {
  show() {
    console.log(this.balance);   // được (protected)
    // console.log(this.pin);    // lỗi (private)
  }
}

const a = new VipAccount("An", 1000, "1234");
// a.balance;                    // lỗi (protected)
// a.pin;                        // lỗi (private)
```

## 3. TS private và #field

TS `private`:
- Chỉ kiểm tra lúc compile.
- Compile ra JS thì chữ `private` biến mất, thuộc tính thành property bình thường.
- Vẫn đọc được bằng ép kiểu: `(obj as any).pin`.

`#field` (native private của JavaScript):
- Do JavaScript engine bảo vệ lúc chạy.
- Truy cập từ ngoài class thì lỗi cú pháp, không có cách nào đọc.

```ts
class Config {
  private softKey = "abc";
  #hardKey = "xyz";

  checkKey(k: string) {
    return this.#hardKey === k;
  }
}

const c = new Config();
console.log((c as any).softKey);   // "abc"  -> vẫn đọc được lúc chạy
console.log((c as any).hardKey);   // undefined, không đọc được #hardKey
console.log(c.checkKey("xyz"));    // true
```

Muốn bảo mật thật lúc chạy thì dùng `#field`.

## 4. abstract class

- Là class không tạo trực tiếp được bằng `new`, chỉ để class khác kế thừa.
- Có thể chứa method đã cài sẵn và method `abstract` (chỉ khai báo, class con bắt buộc cài).

```ts
abstract class Payment {
  process(amount: number) {          // method chung, cài sẵn
    this.validate(amount);
    this.pay(amount);
  }
  protected validate(amount: number) {
    if (amount <= 0) throw new Error("So tien khong hop le");
  }
  protected abstract pay(amount: number): void;   // class con phải cài
}

class CardPayment extends Payment {
  protected pay(amount: number) {
    console.log("Thanh toan the:", amount);
  }
}

// new Payment();                  // lỗi: không tạo trực tiếp được
new CardPayment().process(500);    // Thanh toan the: 500
```

## 5. abstract class vs interface

- `interface`: chỉ mô tả hình dạng, không có code, biến mất sau khi compile. Một class `implements` được nhiều interface.
- `abstract class`: có thể chứa code dùng chung, vẫn tồn tại lúc chạy. Một class chỉ `extends` được 1 class cha.

```ts
interface Printable { print(): void }
class Doc implements Printable {
  print() { console.log("in"); }
}
```

Dùng interface khi chỉ cần mô tả kiểu (DTO, response). Dùng abstract class khi muốn có sẵn logic chung cho các class con.

## Lưu ý / hay nhầm

- `private` của TS không bảo vệ được lúc chạy, chỉ giúp báo lỗi khi viết code.
- `protected` cho phép class con dùng, `private` thì không.
- Class con có `constructor` phải gọi `super()` trước khi dùng `this`.
- `abstract` method không có thân hàm, class con không cài sẽ lỗi compile.
- `interface` không tồn tại lúc chạy nên không dùng được với `instanceof`.

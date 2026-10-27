# 03. JavaScript - this, prototype, class

## 1. this là gì

`this` được quyết định lúc hàm được gọi (gọi như thế nào), không phải lúc khai báo.

## 2. Các quy tắc xác định this

### Default binding: gọi hàm đơn lẻ
- `foo()` thì `this` là global object (`window` / `global`).
- Trong `"use strict"` thì `this` là `undefined`.

```js
function show() { console.log(this); }
show();   // global (không strict), undefined (strict)
```

### Implicit binding: gọi qua object
- `obj.foo()` thì `this` là object đứng trước dấu chấm.

```js
const teacher = {
  name: "Co Mai",
  hello() { return "Xin chao " + this.name; },
};
console.log(teacher.hello());   // Xin chao Co Mai
```

Hay bị mất this khi tách hàm ra khỏi object:

```js
const fn = teacher.hello;
console.log(fn());              // Xin chao undefined  (lúc này là default binding)

setTimeout(teacher.hello, 0);   // cũng mất this vì truyền như callback
```

### Explicit binding: call, apply, bind
- `fn.call(obj, a, b)`: gọi ngay, tham số truyền rời nhau.
- `fn.apply(obj, [a, b])`: gọi ngay, tham số truyền trong mảng.
- `fn.bind(obj)`: không gọi ngay, trả về hàm mới đã khóa `this`.

```js
const other = { name: "Thay Bach" };
console.log(teacher.hello.call(other));    // Xin chao Thay Bach
console.log(teacher.hello.apply(other));   // Xin chao Thay Bach
const bound = teacher.hello.bind(other);
console.log(bound());                      // Xin chao Thay Bach
```

### new binding
Khi gọi `new Foo()`, JS làm 4 bước:
1. Tạo object rỗng `{}`.
2. Gắn `__proto__` của nó trỏ tới `Foo.prototype`.
3. Gán `this` trong `Foo` là object mới.
4. Trả về object đó (nếu hàm không return object khác).

```js
function Person(name) { this.name = name; }
const p = new Person("An");
console.log(p.name);   // An
```

### Arrow function: lexical this
- Arrow không có `this` riêng, lấy `this` của chỗ nó được viết ra.
- `call`, `apply`, `bind` không đổi được `this` của arrow.

```js
const timer = {
  seconds: 0,
  start() {
    setInterval(() => {
      this.seconds++;          // this vẫn là timer, nhờ arrow
    }, 1000);
  },
};
```

Thứ tự ưu tiên: `new` > explicit (`call/apply/bind`) > implicit (`obj.fn()`) > default.

## 3. Prototype chain

Mọi object có 1 liên kết ẩn tới object cha gọi là `[[Prototype]]` (truy cập qua `__proto__`).

```
myDog
  | __proto__
  v
Dog.prototype      (bark)
  | __proto__
  v
Animal.prototype   (eat)
  | __proto__
  v
Object.prototype   (toString, hasOwnProperty)
  | __proto__
  v
null
```

Khi gọi `myDog.eat()`:
1. Tìm `eat` trong `myDog`, không có.
2. Tìm lên `Dog.prototype`, không có.
3. Tìm lên `Animal.prototype`, thấy và chạy.
4. Lên tới `null` mà vẫn không thấy thì trả về `undefined` (hoặc lỗi nếu gọi như hàm).

`prototype` khác `__proto__`:
- `prototype`: có trên function/class, là "khuôn" cho các object tạo bằng `new`.
- `__proto__`: có trên mọi object, trỏ tới `prototype` của hàm đã tạo ra nó.

```js
function Animal(name) { this.name = name; }
Animal.prototype.eat = function () { return this.name + " an"; };

function Dog(name) { Animal.call(this, name); }
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function () { return this.name + " sua"; };

const d = new Dog("Lu");
console.log(d.bark());                       // Lu sua
console.log(d.eat());                        // Lu an
console.log(d.__proto__ === Dog.prototype);  // true
console.log(d instanceof Animal);            // true
```

## 4. ES6 class

`class` chỉ là cách viết gọn, bên dưới vẫn là function và prototype.

```js
class Animal {
  constructor(name) { this.name = name; }
  eat() { return this.name + " an"; }
}
class Dog extends Animal {
  bark() { return this.name + " sua"; }
}
const d = new Dog("Lu");
console.log(typeof Animal);                  // "function"
console.log(d.eat === Animal.prototype.eat); // true, method nằm trên prototype
```

- `extends` thiết lập prototype chain giống cách viết tay ở trên.
- Trong class con có `constructor` thì phải gọi `super()` trước khi dùng `this`.

## Lưu ý / hay nhầm

- Truyền method làm callback (`setTimeout(obj.fn)`, `arr.map(obj.fn)`) là hay mất `this`. Sửa bằng `bind` hoặc bọc arrow function.
- Không dùng arrow function làm method trong object nếu cần `this` trỏ về object.
- `instanceof` kiểm tra bằng prototype chain, không phải kiểm tra tên class.
- Method khai báo trong `class` dùng chung trên prototype, không bị copy cho từng instance.

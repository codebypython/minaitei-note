# 01. JavaScript - Biến, kiểu dữ liệu, hoisting

## 1. var / let / const

- `var`: phạm vi theo hàm (function scope), khai báo lại được, bị hoist và có giá trị `undefined`.
- `let`: phạm vi theo khối `{ }`, gán lại được, không khai báo lại được.
- `const`: phạm vi theo khối, không gán lại được (nhưng bên trong object vẫn sửa được).

```js
var a = 1;
var a = 2;          // được, không báo lỗi -> dễ ghi đè nhầm biến

let b = 1;
// let b = 2;       // SyntaxError: Identifier 'b' has already been declared

const c = 1;
// c = 2;           // TypeError: Assignment to constant variable.

const user = { name: "An" };
user.name = "Binh"; // được, vì chỉ đổi nội dung, không đổi địa chỉ
```

Ví dụ phạm vi:

```js
if (true) {
  var x = 10;   // var không bị giới hạn bởi block
  let y = 20;   // let chỉ sống trong block
}
console.log(x); // 10
console.log(y); // ReferenceError: y is not defined
```

Quy tắc dùng: mặc định dùng `const`, cần gán lại thì dùng `let`, không dùng `var`.

## 2. Primitive và Reference

Primitive (lưu giá trị trực tiếp): `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.
Reference (lưu địa chỉ): `object`, `array`, `function`.

```
Call Stack                    Memory Heap
a = 5                         
b = 5   (copy giá trị)

userA ---------\
                >----------> { name: "An" }
userB ---------/
```

- Primitive: gán `b = a` là copy giá trị, sửa `b` không ảnh hưởng `a`.
- Reference: gán `userB = userA` là copy địa chỉ, 2 biến cùng trỏ 1 object trên Heap.

```js
const userA = { name: "An" };
const userB = userA;
userB.name = "Binh";
console.log(userA.name); // "Binh"  (userA bị đổi theo)
```

Kết quả so sánh:

```js
const u1 = { id: 1 };
const u2 = { id: 1 };
console.log(u1 === u2);   // false, vì 2 địa chỉ khác nhau
console.log(u1 === u1);   // true
```

## 3. Hoisting và TDZ

JS chạy mỗi đoạn code theo 2 bước:
1. Bước tạo (creation): quét hết khai báo, cấp ô nhớ trước.
2. Bước chạy (execution): chạy từng dòng từ trên xuống.

Kết quả của bước 1:
- `function` khai báo: đưa nguyên cả hàm vào bộ nhớ, nên gọi trước dòng khai báo vẫn được.
- `var`: có ô nhớ, giá trị tạm là `undefined`.
- `let`, `const`: có ô nhớ nhưng chưa khởi tạo, nằm trong vùng TDZ.

```js
console.log(v);   // undefined (var đã hoist)
var v = 1;

console.log(l);   // ReferenceError: Cannot access 'l' before initialization
let l = 2;

hello();          // chạy được
function hello() { console.log("hi"); }
```

TDZ (Temporal Dead Zone) là đoạn từ đầu block đến dòng khai báo `let`/`const`. Đụng vào biến trong đoạn này thì báo `ReferenceError`.

## 4. So sánh == và ===

- `==`: tự ép kiểu rồi mới so sánh.
- `===`: so sánh cả kiểu lẫn giá trị, không ép kiểu.

```js
0 == false;           // true
0 === false;          // false
"5" == 5;             // true
"5" === 5;            // false
null == undefined;    // true
null === undefined;   // false
NaN === NaN;          // false
Number.isNaN(NaN);    // true  (cách kiểm tra NaN đúng)
```

Luôn dùng `===`.

## 5. Shallow copy và deep copy

```js
const original = { name: "An", skills: ["JS", "TS"] };

const shallow = { ...original };
shallow.skills.push("NodeJS");
console.log(original.skills); // ["JS", "TS", "NodeJS"]  (bị đổi theo)

const deep = structuredClone(original);
deep.skills.push("React");
console.log(original.skills); // không bị đổi
```

- Shallow copy (`{ ...obj }`, `Object.assign`): chỉ copy tầng ngoài, object con vẫn dùng chung địa chỉ.
- Deep copy (`structuredClone`): copy toàn bộ các tầng.

## Lưu ý / hay nhầm

- `const` không có nghĩa là object bất biến, chỉ là không gán lại biến.
- `typeof null` trả về `"object"` (lỗi cũ của JS), kiểm tra null phải dùng `=== null`.
- Hàm khai báo `function f(){}` gọi trước được, nhưng `const f = () => {}` gọi trước sẽ lỗi TDZ.
- Copy object bằng dấu `=` không tạo bản sao, chỉ tạo thêm 1 tên trỏ vào cùng object.
- Dùng `{ ...obj }` rồi sửa mảng con vẫn làm đổi object gốc.

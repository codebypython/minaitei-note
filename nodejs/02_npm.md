# 2. NodeJS - npm

## 1. npm là gì

npm (Node Package Manager) đi kèm NodeJS, gồm:
- npm registry: kho chứa thư viện JavaScript.
- npm CLI: lệnh để cài, gỡ, chạy script.

## 2. package.json

File mô tả dự án: tên, phiên bản, script, thư viện cần dùng.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "express": "^4.19.2"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  }
}
```

- `dependencies`: thư viện cần khi chạy thật (express, dotenv...).
- `devDependencies`: chỉ cần khi đang code/test (nodemon, typescript...).
- `scripts`: lệnh viết tắt, chạy bằng `npm run <tên>`.

## 3. package-lock.json

- Ghi chính xác phiên bản của từng package đã cài (kể cả package con).
- Để máy nào clone về cũng cài ra đúng bản giống nhau.
- Không sửa tay, commit lên git cùng `package.json`.

## 4. node_modules

- Thư mục chứa code của các thư viện đã cài.
- Rất nặng, không commit lên git (thêm vào `.gitignore`).
- Mất cũng không sao, chạy `npm install` là tạo lại từ `package.json`.

## 5. Phiên bản (SemVer)

Dạng `MAJOR.MINOR.PATCH`, ví dụ `4.19.2`:
- PATCH (đổi số cuối): sửa lỗi nhỏ.
- MINOR (số giữa): thêm tính năng, vẫn tương thích bản cũ.
- MAJOR (số đầu): thay đổi lớn, code cũ có thể hỏng.

Ký hiệu trước số:
- `^1.2.3`: cho cập nhật MINOR và PATCH (nhỏ hơn 2.0.0).
- `~1.2.3`: chỉ cập nhật PATCH (nhỏ hơn 1.3.0).
- `1.2.3`: cố định đúng bản này.

## 6. Lệnh hay dùng

```bash
npm init -y                    # tạo nhanh package.json
npm install express            # cài vào dependencies
npm install -D nodemon         # cài vào devDependencies
npm install                    # cài tất cả theo package.json
npm uninstall express          # gỡ package
npm install -g <tên>           # cài global (dùng ở mọi nơi)
npm run dev                    # chạy script "dev"
npm start                      # chạy script "start" (không cần chữ run)
```

## Lưu ý / hay nhầm

- `npm install` thiếu `-D` thì thư viện dev bị vào `dependencies`, làm nặng bản chạy thật.
- Chỉ `start` và `test` chạy không cần `run`; script khác phải `npm run <tên>`.
- Clone dự án về mà thiếu `node_modules` thì phải chạy `npm install` trước khi chạy.
- Xóa `package-lock.json` bừa bãi có thể làm phiên bản thư viện thay đổi giữa các máy.

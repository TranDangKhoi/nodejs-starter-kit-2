# Setup dự án NodeJS + Typescript + ESLint + Prettier

### 1. Đầu tiên là khởi tạo dự án

Chạy câu lệnh sau để ngay lập tức tạo ra một file package.json mà không cần nhập thông tin:

```bash
npm init -y
```

### 2. Thêm Typescript vào làm devDeps

Chạy câu lệnh sau:

```bash
npm install typescript --save-dev
```

Những ai sử dụng `yarn` hay `pnpm` có thể tự chỉnh sửa câu lệnh trên theo nhu cầu của mình

### 3. Cài đặt kiểu dữ liệu TypeScript cho Node.js

Chạy câu lệnh sau:

```bash
npm install @types/node --save-dev
```

Những ai sử dụng `yarn` hay `pnpm` có thể tự chỉnh sửa câu lệnh trên theo nhu cầu của mình

### 4. Cài đặt các package config cần thiết còn lại

```bash
npm install eslint prettier eslint-config-prettier eslint-plugin-prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser ts-node tsc-alias tsconfig-paths rimraf nodemon --save-dev
```

Những ai sử dụng `yarn` hay `pnpm` có thể tự chỉnh sửa câu lệnh trên theo nhu cầu của mình

- eslint: Linter (bộ kiểm tra lỗi) chính

- prettier: Code formatter chính

- eslint-config-prettier: Cấu hình ESLint để không bị xung đột với Prettier

- eslint-plugin-prettier: Dùng thêm một số rule prettier cho eslint

- @typescript-eslint/eslint-plugin: ESLint plugin cung cấp các rule cho Typescript

- @typescript-eslint/parser: Parser cho phép ESLint kiểm tra lỗi Typescript

- ts-node: Dùng để chạy TypeScript code trực tiếp mà không cần build

- tsc-alias: Xử lý alias khi build

- tsconfig-paths: Khi setting alias import trong dự án dùng ts-node thì chúng ta cần dùng tsconfig-paths để nó hiểu được paths và baseUrl trong file tsconfig.json

- rimraf: Dùng để xóa folder dist khi trước khi build

- nodemon: Dùng để tự động restart server khi có sự thay đổi trong code

### 5. Cấu hình tsconfig.json

Tạo file tsconfig.json tại thư mục root, có thể tạo bằng lệnh `touch tsconfig.json`

Tiếp theo copy và paste cấu hình dưới đây vào file tsconfig.json của bạn

```json
{
  "compilerOptions": {
    "module": "CommonJS", // Quy định output module được sử dụng
    "moduleResolution": "node", //
    "target": "ES2020", // Target ouput cho code
    "outDir": "dist", // Đường dẫn output cho thư mục build
    "esModuleInterop": true /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */,
    "strict": true /* Enable all strict type-checking options. */,
    "skipLibCheck": true /* Skip type checking all .d.ts files. */,
    "baseUrl": ".", // Đường dẫn base cho các import
    "paths": {
      "~/*": ["src/*"] // Đường dẫn tương đối cho các import (alias)
    }
  },
  "ts-node": {
    "require": ["tsconfig-paths/register"]
  },
  "files": ["src/types.d.ts"], // Các file dùng để defined global type cho dự án
  "include": ["src/**/*"] // Đường dẫn include cho các file cần build
}
```

### 6. Tạo file .eslintrc và .prettierrc

Tạo 2 file .eslintrc và .prettierrc ở thư mục root và config như phía dưới

- File .eslintrc config như sau:

```json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint", "prettier"],
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended", "eslint-config-prettier", "prettier"],
  "rules": {
    "@typescript-eslint/no-explicit-any": "off",
    "@typescript-eslint/no-unused-vars": "off",
    "prettier/prettier": [
      "warn",
      {
        "arrowParens": "always",
        "semi": true,
        "trailingComma": "all",
        "tabWidth": 2,
        "endOfLine": "auto",
        "useTabs": false,
        "singleQuote": false,
        "printWidth": 120,
        "jsxSingleQuote": false,
        "singleAttributePerLine": true
      }
    ]
  }
}
```

- File .prettierrc config như sau:

```json
{
  "arrowParens": "always",
  "semi": true,
  "trailingComma": "all",
  "tabWidth": 2,
  "endOfLine": "auto",
  "useTabs": false,
  "singleQuote": false,
  "printWidth": 120,
  "jsxSingleQuote": false,
  "singleAttributePerLine": true
}
```

### 7. Tạo file .eslintignore và .prettierignore

- Tạo 2 file như tiêu đề ở thư mục root với nội dung như sau:

```json
node_modules/
dist/
```

### 8. Cấu hình nodemon

Tạo file nodemon.json ở thư mục root

Mục đích là cấu hình nodemon để tự động restart server khi có sự thay đổi trong code

```json
{
  "watch": ["src"],
  "ext": ".ts,.js",
  "ignore": [],
  "exec": "npx ts-node ./src/index.ts"
}
```

### 9. Cấu hình scripts trong package.json

Thêm đoạn scripts sau vào package.json

```json
 "scripts": {
    "dev": "npx nodemon",
    "build": "rimraf ./dist && tsc && tsc-alias",
    "start": "node dist/index.js",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "prettier": "prettier --check .",
    "prettier:fix": "prettier --write ."
  }
```

### 10. Tạo file types.d.ts

Tạo file `types.d.ts` trong thư mục `src`, tạm thời bây giờ để trống cũng được. Mục đích là để define các global types cho dự án (ví dụ như là **kiểu dữ liệu cho biến môi trường**).

Các bạn mở file tsconfig.json lên sẽ thấy dòng chúng ta add file này vào để cho typescript nó nhận diện

### 11. Tạo file index.ts

Tạo file `index.ts` trong thư mục `src`, thêm dòng lệnh dưới sau và chạy thử để test xem mọi thứ đã okla chưa:

```ts
const myName: string = "Khoi";
console.log(myName);
```

Nếu các bạn có bắt gặp lỗi `Type string trivially inferred from a string literal, remove type annotation.` thì hãy cứ kệ nó và chạy thử chương trình. Lỗi này là do Typescript đã hiểu rằng `myName` có kiểu dữ liệu là string rồi nên không cần viết thêm `: string` làm gì

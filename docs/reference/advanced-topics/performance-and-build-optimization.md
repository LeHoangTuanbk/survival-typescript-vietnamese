---
sidebar_label: Performance and Build Optimization
---

# Performance and Build Optimization

## 1. TypeScript ảnh hưởng (và không ảnh hưởng) điều gì

Trước khi tối ưu, cần làm rõ một điều quan trọng: **TypeScript không ảnh hưởng trực tiếp đến performance khi chạy chương trình**.

- **Chỉ tồn tại ở compile time**
  TypeScript type sẽ bị loại bỏ hoàn toàn sau khi build. Ở runtime, chỉ còn JavaScript thuần.

- **Hiệu năng runtime phụ thuộc vào**:

  - JavaScript được generate ra
  - Môi trường chạy (browser / Node.js)
  - Cách bundler và minifier được cấu hình

- **Mục tiêu tối ưu chính của TypeScript**:

  - Giảm thời gian type-check
  - Giảm thời gian build
  - Rút ngắn vòng phản hồi cho developer (feedback loop)

---

## 2. Tối ưu hiệu năng type-checking

### 2.1 Incremental Compilation

TypeScript có thể ghi nhớ trạng thái compile trước đó để tránh chạy lại từ đầu.

- Bật cấu hình:

  ```json
  {
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo"
  }
  ```

- Lợi ích:

  - Tái sử dụng metadata của lần build trước
  - Giảm đáng kể thời gian compile lại

- Phù hợp nhất cho:

  - Project lớn
  - Watch mode
  - Monorepo

---

### 2.2 Project References (Kiến trúc mở rộng tốt)

Thay vì một project TypeScript khổng lồ, hãy chia codebase thành các đơn vị logic rõ ràng:

- Ví dụ:

  - `core`
  - `utils`
  - `frontend`
  - `backend`

- Lợi ích:

  - Build song song
  - Cache kết quả compile
  - Dependency graph rõ ràng

- Cấu hình bắt buộc:

  ```json
  {
    "composite": true,
    "declaration": true
  }
  ```

---

### 2.3 Bỏ qua những kiểm tra không cần thiết

Không phải mọi thứ đều cần được type-check.

- `"skipLibCheck": true`

  - Bỏ qua việc kiểm tra `.d.ts` trong `node_modules`
  - Tăng tốc build rất lớn
  - Rủi ro thấp trong thực tế

- `"skipDefaultLibCheck": true`

  - Bỏ qua kiểm tra các file lib mặc định của TypeScript

---

### 2.4 Giảm phạm vi file cần compile

Phạm vi compile càng lớn, build càng chậm.

- Tránh include quá rộng:

  ```json
  "include": ["src"]
  ```

- Loại trừ rõ ràng các file không cần thiết:

  ```json
  "exclude": ["node_modules", "dist", "build", "**/*.test.ts"]
  ```

---

## 3. Tối ưu nâng cao trong `tsconfig.json`

### 3.1 Chiến lược Target & Module

Càng target JavaScript mới, compiler càng phải làm ít việc hơn.

- Nên dùng target cao:

  ```json
  "target": "ES2020"
  ```

- Module mới:

  ```json
  "module": "ESNext"
  ```

- Giảm:

  - Polyfill
  - Helper code
  - Độ phức tạp khi transpile

---

### 3.2 Đánh đổi giữa strictness và tốc độ

- An toàn cao hơn nhưng chậm hơn:

  ```json
  "strict": true
  ```

- Một số flag có thể ảnh hưởng đáng kể đến tốc độ:

  - `strictNullChecks`
  - `noUncheckedIndexedAccess`

- Thực tế tốt nhất:

  - Strict đầy đủ trong CI
  - Có thể nới lỏng trong local dev (nếu cần)

---

### 3.3 Isolated Modules

```json
"isolatedModules": true
```

- Bắt buộc khi dùng:

  - Babel
  - SWC
  - esbuild

- Ngăn TypeScript đưa ra các giả định type giữa nhiều file

---

## 4. Sử dụng build tools nhanh hơn

### 4.1 Babel kết hợp TypeScript

Tách riêng hai trách nhiệm:

- Type-checking:

  ```bash
  tsc --noEmit
  ```

- Transpiling:

  ```bash
  babel src --out-dir dist
  ```

- Phù hợp cho frontend app lớn

---

### 4.2 SWC / esbuild

- Cực nhanh (viết bằng Rust / Go)

- Không type-check trong quá trình bundle

- Mô hình phổ biến:

  - Dev: esbuild / SWC
  - CI: `tsc --noEmit`

---

## 5. Bundling, Tree Shaking & tối ưu output

### 5.1 Nguyên tắc Tree Shaking

- Nên dùng:

  ```ts
  export function foo() {}
  ```

- Tránh:

  ```ts
  export default { foo };
  ```

- Ưu tiên named exports

- Khai báo `"sideEffects": false` trong `package.json`

---

### 5.2 Loại bỏ dead code

- Ưu tiên `as const` thay vì enum
- Tránh `require` động
- Dùng import tĩnh

---

## 6. Best practices cho runtime performance

### 6.1 Enums vs const objects

- Enums tạo code runtime
- Nên dùng:

  ```ts
  const Status = {
    ACTIVE: "active",
    INACTIVE: "inactive",
  } as const;
  ```

---

### 6.2 Classes vs plain objects

- Classes:

  - Tốn bộ nhớ hơn
  - Có prototype chain

- Objects:

  - Tạo nhanh hơn
  - Runtime đơn giản hơn

→ Chỉ dùng class khi thật sự cần encapsulation hành vi hoặc state.

---

### 6.3 Tránh type quá phức tạp

- Type nâng cao làm chậm compiler
- Tránh:

  - Conditional types lồng sâu
  - Recursive types
  - Mapped types quá nặng trong hot path

---

## 7. Tối ưu performance cho monorepo

### 7.1 Dùng base config chung

```json
{
  "extends": "../../tsconfig.base.json"
}
```

---

### 7.2 Build caching

- Dùng:

  - Turborepo
  - Nx

- Cache:

  - `tsc`
  - Output của bundler

→ Tránh build lại những phần không thay đổi

---

## 8. Phân biệt Dev / CI / Production

### Development

- Rebuild nhanh
- Check một phần
- Có source maps

### CI

- Type-check đầy đủ
- `--noEmit`
- Lint + test

### Production

- Minify
- Tree shaking
- Tắt hoặc ẩn source maps

---

## 9. Đo và phân tích hiệu năng

### Compiler diagnostics

```bash
tsc --diagnostics
tsc --extendedDiagnostics
```

Dùng để xác định:

- File compile chậm
- Mức sử dụng bộ nhớ
- Bottleneck trong type-check

---

## 10. Các anti-pattern thường gặp

- Chạy `tsc` mỗi lần gõ phím
- Một `tsconfig` khổng lồ cho mọi thứ
- Compile cả test vào build production
- Overengineering type-level logic

---

## 11. Best practices cấp độ enterprise (Tóm tắt)

- Chia project bằng project references
- Dùng transpiler nhanh cho dev
- Cache nhiều nhất có thể
- Giới hạn scope compile
- Target JS mới nhất có thể
- Type-check một lần, build nhiều lần

---

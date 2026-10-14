# Báo Cáo & Hướng Dẫn Bài 4: Khai Báo GitHub Secrets & Cấu Hình Đăng Nhập Container Registry (GHCR)

## 1. Thông Tin Chung
- **Môn học**: Thực hành DevOps / CI-CD (IT209)
- **Bài thực hành**: Session 11 - Bài 4: Khai báo GitHub Secrets và cấu hình Đăng nhập Container Registry
- **Học viên**: Nguyễn Việt Nam
- **Đường dẫn bài nộp**: `homework/session_11/ex4/`
- **Kho lưu trữ GitHub**: [Breaveluv/IT209-homework-session_11-ex4-](https://github.com/Breaveluv/IT209-homework-session_11-ex4-)

---

## 2. Mục Tiêu & Yêu Cầu Bài Thực Hành

### 2.1. Mục Tiêu
1. Tạo tệp mã hóa an toàn (**Repository Secret**) lưu trữ Token cá nhân trên GitHub.
2. Cấu hình bước xác thực đăng nhập tự động vào **GitHub Container Registry (GHCR - ghcr.io)** trong luồng CI/CD.

### 2.2. Yêu Cầu Cụ Thể
- **Yêu cầu 1**: Tạo **Personal Access Token (classic)** trên GitHub với quyền (scope) `write:packages`.
- **Yêu cầu 2**: Vào **Settings -> Secrets and variables -> Actions** của Repository, tạo bí mật mới đặt tên là `REGISTRY_TOKEN` và dán token vừa tạo.
- **Yêu cầu 3**: Trong file kịch bản `.github/workflows/ci.yml`, khai báo bước đăng nhập `docker/login-action@v3` truyền vào biến `secrets.REGISTRY_TOKEN`.
- **Yêu cầu 4**: Đẩy code lên GitHub, mở log chi tiết của bước Đăng nhập Registry trên tab Actions và xác nhận hiển thị **`Login Succeeded`**.
- **Yêu cầu nộp bài**: Tệp kịch bản `.github/workflows/ci.yml` (chứa bước `docker/login-action@v3`) và ảnh chụp màn hình mục Repository Secrets hiển thị biến `REGISTRY_TOKEN` đã khởi tạo.

---

## 3. Cấu Trúc Thư Mục Dự Án

```text
├── .github/
│   └── workflows/
│       └── ci.yml             # Kịch bản CI tích hợp Gradle Caching & GHCR Registry Login
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── src/
│   ├── main/
│   │   ├── java/com/example/caching/
│   │   │   ├── Session11Ex3Application.java
│   │   │   └── controller/
│   │   │       └── GreetingController.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/example/caching/
│           ├── Session11Ex3ApplicationTests.java
│           └── GreetingControllerTest.java
├── .gitignore
├── build.gradle               # Khai báo dependencies Spring Boot & Gradle
├── settings.gradle            # Tên root project (session11-ex4-ghcr-login)
├── gradlew                    # Script thực thi Gradle trên Linux/macOS
├── gradlew.bat                # Script thực thi Gradle trên Windows
├── report.md                  # Báo cáo thực hành chi tiết
└── README.md                  # Hướng dẫn chi tiết quy trình thực hiện
```

---

## 4. Hướng Dẫn Thực Hiện Chi Tiết

### Bước 1: Khởi Tạo Personal Access Token (Classic) Trên GitHub
1. Truy cập trang cài đặt Token của tài khoản cá nhân:
   - Đường dẫn: [GitHub Personal Access Tokens](https://github.com/settings/tokens) (Settings -> Developer Settings -> Personal access tokens -> Tokens (classic)).
2. Nhấn nút **Generate new token** -> Chọn **Generate new token (classic)**.
3. Cấu hình thông tin Token:
   - **Note**: `GHCR_REGISTRY_TOKEN` (hoặc tên gợi nhớ mục đích).
   - **Expiration**: Chọn thời hạn phù hợp (30 days, 60 days hoặc tùy ý).
   - **Select scopes**: Tích chọn quyền:
     - `write:packages` (Upload packages to GitHub Package Registry). Hệ thống sẽ tự động kích hoạt `read:packages`.
4. Cuộn xuống cuối trang và nhấn nút xanh **Generate token**.
5. **Quan trọng**: Sao chép chuỗi mã token ngay lập tức (token dạng `ghp_...`). Chuỗi này chỉ hiển thị một lần duy nhất.

### Bước 2: Cấu Hình Repository Secret Trên Repository
1. Truy cập vào kho lưu trữ GitHub của bài tập:
   - `https://github.com/Breaveluv/IT209-homework-session_11-ex4-`
2. Chọn tab **Settings** ở thanh menu ngang trên cùng của repo.
3. Ở menu điều hướng bên trái, tìm nhóm **Security** -> chọn **Secrets and variables** -> bấm **Actions**.
4. Tại khu vực **Repository secrets**, nhấn nút xanh **New repository secret**.
5. Nhập thông tin:
   - **Name**: `REGISTRY_TOKEN` (bắt buộc đúng tên chữ in hoa theo đề bài).
   - **Secret**: Dán mã Personal Access Token đã sao chép từ Bước 1.
6. Nhấn nút **Add secret**.
7. Sau khi thêm thành công, giao diện hiển thị biến `REGISTRY_TOKEN` cùng thông tin ngày khởi tạo.

### Bước 3: Khai Báo Bước Đăng Nhập Trong `.github/workflows/ci.yml`
Tệp `.github/workflows/ci.yml` được cấu hình để thực hiện kiểm thử, đóng gói ứng dụng và xác thực vào GHCR:

```yaml
name: Java CI & GHCR Registry Login

on:
  push:
    branches: [ "main", "master" ]
  pull_request:
    branches: [ "main", "master" ]
  workflow_dispatch:

jobs:
  build-and-login:
    name: Build, Test & GHCR Login
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v4

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền thực thi cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Chạy kiểm thử tự động (Test Suite)
        run: ./gradlew test --no-daemon

      - name: Đóng gói sản phẩm (Build & Assemble)
        run: ./gradlew assemble --no-daemon

      - name: Đăng nhập vào GitHub Container Registry (GHCR)
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

### Bước 4: Đẩy Code Lên GitHub và Xác Minh Kết Quả
1. Thêm tệp và tạo commit:
   ```bash
   git add .
   git commit -m "feat: configure GHCR login using REGISTRY_TOKEN secret"
   git push origin main
   ```
2. Mở tab **Actions** trên GitHub repository:
   - Chọn workflow chạy mới nhất.
   - Chọn job `Build, Test & GHCR Login`.
   - Bấm mở rộng bước **Đăng nhập vào GitHub Container Registry (GHCR)**.
   - Xác nhận log hiển thị dòng thông báo:
     ```text
     Login Succeeded
     ```

---

## 5. Nguyên Lý Bảo Mật & Best Practices

1. **Cơ chế mã hóa GitHub Secrets**:
   - GitHub Secrets sử dụng thuật toán mã hóa bất đối xứng **Libsodium (Curve25519, Salsa20, Poly1305)**.
   - Dữ liệu mật được mã hóa ngay trên trình duyệt trước khi gửi về máy chủ GitHub và chỉ được giải mã tạm thời trong RAM của máy chủ thực thi (Runner) khi Job chạy.
2. **Cơ chế Masking trong Log**:
   - Tất cả giá trị được định nghĩa trong Secrets tự động bị che (`***`) trong toàn bộ stdout/stderr logs để phòng ngừa rò rỉ dữ liệu ngoài ý muốn.
3. **Nguyên tắc Đặc quyền Tối thiểu (Principle of Least Privilege)**:
   - Token chỉ cấp quyền `write:packages` phục vụ đẩy image vào GHCR, tuyệt đối không cấp các quyền thừa (như full `repo` admin, `delete_repo`, `admin:org`).

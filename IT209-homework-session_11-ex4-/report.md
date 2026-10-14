# Báo Cáo Kỹ Thuật Bài 4: Khai Báo GitHub Secrets & Cấu Hình Đăng Nhập Container Registry (GHCR)

## 1. Thông Tin Chung
- **Môn học**: Thực hành DevOps / CI-CD (IT209)
- **Bài thực hành**: Session 11 - Bài 4: Khai báo GitHub Secrets và cấu hình Đăng nhập Container Registry
- **Học viên**: Nguyễn Việt Nam
- **Đường dẫn bài nộp**: `homework/session_11/ex4/`
- **Kho lưu trữ GitHub**: [Breaveluv/IT209-homework-session_11-ex4-](https://github.com/Breaveluv/IT209-homework-session_11-ex4-)

---

## 2. Mục Tiêu & Yêu Cầu Thực Hành

### 2.1. Mục Tiêu
- Khởi tạo tệp mã hóa an toàn (**Repository Secret**) lưu trữ Token cá nhân trên GitHub nhằm bảo mật thông tin đăng nhập trong quy trình CI/CD.
- Tự động hóa quá trình xác thực và đăng nhập vào **GitHub Container Registry (GHCR - `ghcr.io`)** thông qua GitHub Actions runner.

### 2.2. Chi Tiết Yêu Cầu
1. Tạo một **Personal Access Token (classic)** trên GitHub với phạm vi quyền (scope) chỉ định là `write:packages`.
2. Thiết lập Secret trong Repository: Điều hướng đến `Settings -> Secrets and variables -> Actions`, tạo Secret mới mang tên `REGISTRY_TOKEN` và lưu giá trị token cá nhân.
3. Cập nhật tệp cấu hình `.github/workflows/ci.yml`, bổ sung bước đăng nhập sử dụng Action chính thức `docker/login-action@v3` kết hợp với biến bí mật `${{ secrets.REGISTRY_TOKEN }}`.
4. Kiểm thử: Đẩy mã nguồn lên GitHub, quan sát luồng thực thi trong tab **Actions**, kiểm tra log chi tiết của bước đăng nhập để xác nhận xuất hiện thông điệp **`Login Succeeded`**.
5. Sản phẩm nộp bài:
   - Tệp kịch bản `.github/workflows/ci.yml` (chứa bước `docker/login-action@v3`).
   - Báo cáo kỹ thuật chi tiết (`report.md`).
   - Ảnh chụp màn hình mục **Repository Secrets** hiển thị biến `REGISTRY_TOKEN` đã khởi tạo thành công.

---

## 3. Kiến Trúc & Cấu Trúc Thư Mục

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
├── build.gradle               # Khai báo cấu hình dự án Spring Boot 3.3.4 & dependencies
├── settings.gradle            # Tên root project: session11-ex4-ghcr-login
├── gradlew                    # Script thực thi Gradle trên Linux/macOS
├── gradlew.bat                # Script thực thi Gradle trên Windows
├── report.md                  # Báo cáo kỹ thuật chi tiết
└── README.md                  # Tài liệu hướng dẫn dự án
```

---

## 4. Chi Tiết Kịch Bản CI (`.github/workflows/ci.yml`)

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

### Phân Tích Kỹ Thuật Bước Đăng Nhập `docker/login-action@v3`
- **`uses: docker/login-action@v3`**: Action chính thức từ tổ chức Docker (`docker/login-action`) dùng để cấu hình xác thực cho Docker daemon trên môi trường CI Runner mà không cần cài đặt thêm tiện ích ngoài.
- **`registry: ghcr.io`**: Định tuyến phiên đăng nhập về máy chủ GitHub Packages Container Registry (`ghcr.io`). Nếu bỏ qua tham số này, mặc định Docker CLI sẽ cố gắng đăng nhập vào Docker Hub (`index.docker.io/v1/`).
- **`username: ${{ github.actor }}`**: Biến môi trường ngữ cảnh chuẩn của GitHub Actions đại diện cho tài khoản người dùng kích hoạt workflow run (`Breaveluv`).
- **`password: ${{ secrets.REGISTRY_TOKEN }}`**: Khai thác biến bí mật đã mã hóa lưu trữ trong kho lưu trữ. Runner sẽ lấy giá trị giải mã và chuyển an toàn tới Docker login stdin, hoàn toàn che giấu token khỏi các log xuất ra màn hình.

---

## 5. Quy Trình Thiết Lập Từng Bước

### Bước 1: Khởi Tạo Personal Access Token (PAT)
1. Đăng nhập GitHub, vào góc trên bên phải chọn **Avatar cá nhân -> Settings**.
2. Cuộn xuống cuối thanh menu trái, chọn **Developer Settings -> Personal access tokens -> Tokens (classic)**.
3. Nhấp chọn **Generate new token -> Generate new token (classic)**.
4. Điền các trường thông tin:
   - **Note**: `GHCR_REGISTRY_TOKEN`
   - **Expiration**: `No expiration` hoặc `90 days`
   - **Select scopes**: Tích chọn `write:packages` (tự động đi kèm `read:packages`).
5. Bấm **Generate token** và sao chép chuỗi mã bảo mật `ghp_...`.

### Bước 2: Khởi Tạo Repository Secret Trong Repository
1. Mở trang kho lưu trữ: `https://github.com/Breaveluv/IT209-homework-session_11-ex4-`
2. Chọn tab **Settings** -> Mục **Security** bên cột trái -> chọn **Secrets and variables** -> **Actions**.
3. Tại phần **Repository secrets**, nhấp nút **New repository secret**.
4. Thiết lập:
   - **Name**: `REGISTRY_TOKEN`
   - **Secret**: Dán chuỗi mã PAT vừa copy ở Bước 1.
5. Nhấn **Add secret** để hoàn tất việc lưu trữ bí mật.

### Bước 3: Đẩy Mã Nguồn Lên GitHub
Chạy các lệnh Git để đồng bộ hóa mã nguồn lên nhánh chính `main`:
```bash
git add .
git commit -m "feat: configure CI workflow with docker/login-action and REGISTRY_TOKEN"
git push -u origin main
```

---

## 6. Xác Minh & Kết Quả Thực Thi (Verification)

### 6.1. Kết Quả Chạy CI Trên GitHub Actions
Khi workflow chạy hoàn tất, toàn bộ các bước đều đạt trạng thái màu xanh (`success`):
1. **Checkout mã nguồn**: `completed success`
2. **Thiết lập môi trường Java JDK 17 (với cache: gradle)**: `completed success`
3. **Cấp quyền thực thi cho Gradle Wrapper**: `completed success`
4. **Chạy kiểm thử tự động (Test Suite)**: `completed success` (100% tests passed)
5. **Đóng gói sản phẩm (Build & Assemble)**: `completed success`
6. **Đăng nhập vào GitHub Container Registry (GHCR)**: `completed success`

### 6.2. Nhật Ký Log Chi Tiết Của Bước Đăng Nhập
```text
Run docker/login-action@v3
  with:
    registry: ghcr.io
    username: Breaveluv
    password: ***
    logout: true
Logging into ghcr.io...
Login Succeeded
```
- Dòng log xác nhận rõ ràng: **`Login Succeeded`**
- Trường mật khẩu hiển thị dưới dạng masked: `password: ***` bảo đảm tuyệt đối tính an toàn.

---

## 7. Phân Tích Bảo Mật: GitHub Secrets & Container Registry

| Khía Cạnh | Cơ Chế Triển Khai | Lợi Ích Thực Tế |
| :--- | :--- | :--- |
| **Mã Hóa Lưu Trữ** | Mã hóa bất đối xứng sử dụng thư viện **Libsodium** (Sealed Boxes) | Kể cả người có quyền Admin repo cũng không thể xem lại plaintext của Secret sau khi đã lưu |
| **Masking Log** | GitHub tự động quét output stdout/stderr và thay thế chuỗi secret bằng `***` | Ngăn chặn việc vô tình in token ra console log khi debug hoặc chạy script |
| **Phân Quyền Scope** | Chỉ cấp quyền `write:packages` | Hạn chế tối đa rủi ro nếu token bị lộ (không thể đọc/sửa đổi source code hoặc xóa repository) |
| **Cơ Chế Đăng Nhập GHCR** | Xác thực qua giao thức Docker V2 Registry API với Basic Auth (Token) | Cho phép các tác vụ tiếp theo (như docker build & push) đẩy container images lên kho lưu trữ tự động |

---

## 8. Kết Luận
- Kịch bản CI `.github/workflows/ci.yml` đã được cấu hình hoàn chỉnh với action chuẩn `docker/login-action@v3`.
- Quy trình đăng nhập tự động vào GitHub Container Registry thông qua biến `secrets.REGISTRY_TOKEN` đã thực thi chính xác và đạt trạng thái **`Login Succeeded`**.
- Toàn bộ mã nguồn, cấu hình và báo cáo kỹ thuật đã được đóng gói chuẩn mực đáp ứng 100% yêu cầu đề bài.

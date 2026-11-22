# Bài tập 1: Khắc phục sự cố Pipeline CI/CD cơ bản

## 1. Phân tích nguyên nhân file `.gitlab-ci.yml` ban đầu bị lỗi

File cấu hình gốc:
```yaml
stages:
  build_app

build_job:
  stage: build_app
  script:
    - ./gradlew clean build -x test
```

Khi chạy trên GitLab Runner mặc định, pipeline thất bại do **3 lỗi/thiếu sót cốt lõi**:

1. **Thiếu khai báo `image:` (Nguyên nhân trực tiếp gây lỗi `command not found` / thiếu môi trường):**
   - File gốc không chỉ định `image:` nên Runner mặc định sử dụng môi trường container tối giản (hoặc shell mặc định không có sẵn JDK 17 / Gradle). Khi chạy `./gradlew`, hệ thống không tìm thấy `java` (`JAVA_HOME`) hoặc công cụ cần thiết nên văng lỗi `command not found`.
   - **Cách sửa:** Bổ sung `image: gradle:8.5-jdk17-alpine` ở đầu file.
2. **Sai cú pháp mảng YAML ở mục `stages:`:**
   - Trong YAML, `stages` bắt buộc là một **danh sách (List/Array)** sử dụng dấu gạch đầu dòng `-`. Viết `build_app` không có dấu `-` biến `stages` thành kiểu chuỗi (String), dẫn đến lỗi cú pháp YAML (`stages should be an array of strings`).
   - **Cách sửa:** Đổi thành `- build_app`.
3. **Thiếu bước cấp quyền thực thi (`chmod +x ./gradlew`) trên Linux Runner:**
   - Nếu file `gradlew` được đẩy từ Windows lên Git mà chưa có bit thực thi (`+x`), Runner chạy Linux sẽ báo `./gradlew: Permission denied`.
   - **Cách sửa:** Bổ sung `before_script: - chmod +x ./gradlew`.

## 2. File `.gitlab-ci.yml` đã sửa hoàn chỉnh
Xem tại [`.gitlab-ci.yml`](./.gitlab-ci.yml).

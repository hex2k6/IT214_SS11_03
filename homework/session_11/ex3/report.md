# Bài 3: Gradle Dependency Caching

## Cấu hình

Workflow: [ci.yml](../../../.github/workflows/ci.yml). Thêm `cache: 'gradle'` trong `with` của `actions/setup-java@v5`, cùng Java 17 và distribution Temurin. Các bước test, bootJar và upload artifact app-jar được giữ nguyên.

```yaml
- name: Thiết lập môi trường Java JDK 17
  uses: actions/setup-java@v5
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: 'gradle'
```

## Cách kiểm tra

Ngày thực hiện: 13/10/2026, múi giờ Asia/Saigon.

1. Push commit bật cache vào repository mới, lần đầu chưa có cache Gradle.
2. Đợi lần đầu hoàn thành để cache được lưu.
3. Push commit rỗng thứ hai; mã nguồn, dependencies và workflow không thay đổi.
4. So sánh thời gian và đọc log setup-java. Cả hai lần đều Success.

- [Lần 1 — chưa có cache](https://github.com/hex2k6/IT214_SS11_03/actions/runs/38285017294), commit `74d5a0e`.
- [Lần 2 — đã có cache](https://github.com/hex2k6/IT214_SS11_03/actions/runs/38285270234), commit `8d9d504`.

## Kết quả đo

Thời gian tính từ `completed_at - started_at` của job/bước trong GitHub Actions API, độ phân giải một giây. Tổng job không bao gồm thời gian chờ runner.

| Hạng mục | Lần 1: cache miss | Lần 2: cache hit |
|---|---:|---:|
| setup-java | 1 giây | 5 giây |
| ./gradlew test | 56 giây | 17 giây |
| ./gradlew bootJar | 2 giây | 2 giây |
| Upload artifact | 2 giây | 2 giây |
| Post setup-java (lưu cache) | 5 giây | 0 giây |
| Tổng job Run Unit Tests | 72 giây | 36 giây |
| Trạng thái | Success | Success |

Job lần 1: 00:33:38–00:34:50. Job lần 2: 00:35:57–00:36:33 (Asia/Saigon).

## Bằng chứng cache

Log lần 1:

```text
gradle cache is not found
Cache saved with the key: setup-java-Linux-x64-gradle-aa9561ceccfd8db4f6217837fa8a98e546764b4c489c3f94318e8dbc17d3368e
```

Log lần 2, bước Thiết lập môi trường Java JDK 17:

```text
Cache hit for: setup-java-Linux-x64-gradle-aa9561ceccfd8db4f6217837fa8a98e546764b4c489c3f94318e8dbc17d3368e
Cache restored successfully
Cache restored from key: setup-java-Linux-x64-gradle-aa9561ceccfd8db4f6217837fa8a98e546764b4c489c3f94318e8dbc17d3368e
```

Gradle Wrapper cũng được khôi phục cache thành công. Tiền tố key thực tế là `setup-java-Linux-x64-gradle-`, khác cách viết rút gọn trong đề bài.

## Nhận xét

Tổng thời gian job giảm 36 giây, tương đương 50%. Bước test giảm 39 giây, khoảng 69,6%. Bước setup-java tăng 4 giây do cần tải và giải nén cache, nhưng tổng job vẫn nhanh hơn rõ rệt. Hai lần chạy có cùng nội dung mã nguồn; commit thứ hai là commit rỗng.

Đây là kết quả của một cặp lần chạy; thời gian các lần khác có thể thay đổi theo runner và mạng. Dependency caching không đồng nghĩa bỏ qua việc chạy test.
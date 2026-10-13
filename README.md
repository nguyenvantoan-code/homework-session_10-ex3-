# Bài 3: Tối ưu hóa Docker Image với Multi-stage Build

## 1. Mục tiêu

- Tìm hiểu Single-stage Build và Multi-stage Build.
- Tận dụng Layer Caching để giảm thời gian build.
- Loại bỏ các tệp không cần thiết khỏi Image chạy ứng dụng.
- So sánh dung lượng và số lượng layer của hai Docker Image.

## 2. Công nghệ sử dụng

- Docker
- Node.js 18 Alpine
- Express.js

## 3. Kiến trúc Multi-stage Build

Dockerfile được chia thành hai giai đoạn.

**Stage 1 - Builder:** Sao chép `package.json`, cài đặt dependencies và chuẩn bị mã nguồn ứng dụng.

**Stage 2 - Runtime:** Sử dụng Node.js Alpine và chỉ sao chép `package.json`, `node_modules` cùng `server.js` từ giai đoạn builder.

Việc tách hai giai đoạn giúp hạn chế các tệp không cần thiết xuất hiện trong Image cuối cùng.

## 4. Layer Caching

Docker tạo các layer tương ứng với các chỉ thị trong Dockerfile. Khi nội dung đầu vào của một chỉ thị không thay đổi, Docker có thể tái sử dụng layer đã được cache.

Trong dự án này, `package.json` được sao chép trước khi chạy `npm install`. Nhờ đó, khi chỉ thay đổi `server.js`, Docker có thể tái sử dụng layer cài dependencies nếu các đầu vào liên quan không thay đổi.

## 5. Build Docker Image

Build Image Single-stage:

```bash
docker build -f Dockerfile.single -t hw-backend:single .
```

Build Image Multi-stage:

```bash
docker build -t hw-backend:multistage .
```

## 6. So sánh dung lượng và số lượng layer

Sau khi build hai Image, sử dụng các lệnh sau để lấy số liệu thực tế:

```bash
docker images
docker history hw-backend:single
docker history hw-backend:multistage
```

Điền kết quả vào bảng:

| Tiêu chí | Single-stage | Multi-stage |
|---|---|---|
| Tên Image | hw-backend:single | hw-backend:multistage |
| Dung lượng Image | [Điền kết quả thực tế] | [Điền kết quả thực tế] |
| Số layer hiển thị trong docker history | [Điền kết quả thực tế] | [Điền kết quả thực tế] |
| Cấu trúc build | Một giai đoạn | Hai giai đoạn |
| Dependencies | Cài đặt trong Image | Sao chép từ builder |
| Mã nguồn chạy | Có | Có |
| Khả năng tái sử dụng cache | Có | Có |

Lưu ý: số dòng trong `docker history` là số mục lịch sử build hiển thị, không nhất thiết bằng số layer duy nhất được lưu trữ. Các Image có thể dùng chung layer nền.

## 7. Kiểm tra ứng dụng

Chạy Image Multi-stage:

```bash
docker run -d --name app-multistage -p 8081:3000 hw-backend:multistage
```

Kiểm tra trạng thái:

```bash
docker ps
docker logs app-multistage
```

Gọi API:

```bash
curl -i http://localhost:8081/
```

Kết quả mong đợi là HTTP 200 OK với JSON:

```json
{
  "status": "success",
  "message": "Hello from Docker Single-stage!"
}
```

## 8. Kết luận

Multi-stage Build giúp tách quá trình chuẩn bị ứng dụng khỏi môi trường chạy cuối cùng. Cách làm này hỗ trợ loại bỏ các thành phần không cần thiết, cải thiện khả năng bảo trì và giảm nguy cơ đưa công cụ build vào Image production.

Mức giảm dung lượng thực tế phụ thuộc vào ứng dụng, dependencies, Base Image và các tệp được sao chép. Vì vậy, cần dựa vào kết quả đo từ Docker để đánh giá hiệu quả thay vì mặc định rằng mọi Image Multi-stage đều giảm 50–70%.

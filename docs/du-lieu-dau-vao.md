# Dữ liệu đầu vào

Trong tab **Dữ liệu đầu vào**, người dùng cung cấp 5 nhóm dữ liệu:

| STT | Dữ liệu | Kiểu dữ liệu | Vai trò |
|---|---|---|---|
| 1 | Các kênh ảnh Sentinel-2 | Raster | Tính MNDWI, SAVI và trích xuất thực vật |
| 2 | Raster dân số WorldPop | Raster | Ước tính dân số trong vùng nghiên cứu và vùng phục vụ |
| 3 | Vector khu vực nghiên cứu | Polygon | Giới hạn phạm vi xử lý và đơn vị không gian nghiên cứu |
| 4 | Vector công viên/vườn hoa | Polygon | Cung cấp vùng mẫu để xác định ngưỡng SAVI và hỗ trợ xử lý mảng xanh |
| 5 | Vector dấu vết công trình xây dựng | Polygon | Đại diện cho không gian xây dựng và giới hạn phạm vi phân bố của raster dân số |

## Yêu cầu đối với Sentinel-2

Quy trình phân tích yêu cầu bắt buộc các kênh **B03, B04, B08, B11**. Các kênh **B02** và **SCL** là tùy chọn.

- **B02**: hỗ trợ tạo ảnh True Color RGB.
- **SCL**: hỗ trợ lọc mây, bóng mây và các pixel không phù hợp trước khi tính chỉ số phổ.

## Hệ tọa độ

Vector khu vực nghiên cứu nên sử dụng hệ tọa độ phẳng phù hợp với khu vực nghiên cứu. Trong đồ án thử nghiệm tại Biên Hòa, dữ liệu được chuẩn hóa về hệ tọa độ VN-2000.

!!! warning "Lưu ý về dữ liệu dấu vết công trình"
    Vector dấu vết công trình xây dựng được dùng như lớp đại diện cho **không gian xây dựng**, không được hiểu là dữ liệu nhà ở hoặc khu dân cư thực tế.

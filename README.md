# Green Space Evaluator

**Green Space Evaluator** (tên hiển thị trong QGIS: *Urban Green Space Service Evaluator*) là Plugin QGIS hỗ trợ tự động hóa quy trình đánh giá mức độ phục vụ của mảng xanh đô thị.

Plugin tích hợp chuỗi 5 module xử lý từ dữ liệu ảnh viễn thám Sentinel-2 và dữ liệu dân số WorldPop, mô hình hóa vùng phục vụ theo khoảng cách Euclid và sức tải mục tiêu, phân tích không gian xây dựng và tổng hợp các chỉ tiêu thống kê theo đơn vị hành chính..

![Giao diện chính của Plugin Green Space Evaluator](docs/images/giao_dien_chinh_plugin.png)

---

## Tính năng chính

- **Tiền xử lý và chuẩn hóa dữ liệu**: Tự động lọc mây SCL, gộp kênh ảnh Sentinel-2 và chuẩn hóa độ phân giải raster dân số WorldPop bảo toàn tổng dân số.
- **Trích xuất mảng xanh từ ảnh Sentinel-2**: Tính toán chỉ số phổ MNDWI và SAVI, hỗ trợ tự động xác định ngưỡng SAVI từ lớp công viên mẫu và lọc nhiễu theo diện tích tối thiểu.
- **Mô hình hóa vùng phục vụ**: Tính ma trận khoảng cách hình học Euclid và áp dụng thuật toán tìm kiếm nhị phân (Binary Search) xác định bán kính phục vụ lớn nhất $R^*$ thỏa mãn điều kiện sức tải mục tiêu ($C_{min}$).
- **Phân tích không gian xây dựng**: Phân loại các khối công trình xây dựng (Google Open Buildings / OSM) nằm trong vùng phục vụ và ngoài vùng phục vụ.
- **Tổng hợp chỉ tiêu theo đơn vị hành chính**: Tự động tính toán 6 chỉ số định lượng theo từng phường/xã và hỗ trợ xuất bảng dữ liệu thống kê định dạng Excel (`.xlsx`) hoặc CSV (`.csv`).

---

## Yêu cầu

| Thành phần | Yêu cầu |
|---|---|
| **Phần mềm QGIS** | QGIS 3.44.12+ |
| **Môi trường Python** | Python 3 tích hợp sẵn trong QGIS |
| **Thư viện phụ thuộc** | `numpy`, `scipy`, `matplotlib`, `gdal/osgeo` (tùy chọn: `pandas`, `openpyxl`) |

---

## Cài đặt

1. Tải mã nguồn Plugin từ repository.
2. Sao chép thư mục `green_space_evaluator` vào thư mục plugins của QGIS:
   - **Windows**: `%APPDATA%\QGIS\QGIS3\profiles\default\python\plugins\`
3. Mở QGIS, vào **Plugins** → **Manage and Install Plugins...** → tích chọn kích hoạt **Urban Green Space Service Evaluator**.
4. Mở Plugin từ menu **Plugins** hoặc nhấn vào biểu tượng chiếc lá trên thanh công cụ.

👉 Xem hướng dẫn cài đặt chi tiết: [Tài liệu Cài đặt Plugin](https://xuanphi1702.github.io/green-space-evaluator/cai-dat/)

---

## Hướng dẫn sử dụng

Tài liệu hướng dẫn sử dụng đầy đủ và chi tiết được biên soạn tại website tài liệu chính thức:

- 📖 **Trang chủ tài liệu**: [https://xuanphi1702.github.io/green-space-evaluator/](https://xuanphi1702.github.io/green-space-evaluator/)
- 📥 [Chuẩn bị dữ liệu đầu vào](https://xuanphi1702.github.io/green-space-evaluator/du-lieu-dau-vao/)
- ⚙️ [Cài đặt tham số](https://xuanphi1702.github.io/green-space-evaluator/tham-so/)
- 🚀 [Chạy phân tích](https://xuanphi1702.github.io/green-space-evaluator/chay-phan-tich/)
- 📊 [Sản phẩm & chỉ tiêu thống kê](https://xuanphi1702.github.io/green-space-evaluator/san-pham/)
- 🛠️ [Lưu ý & Xử lý lỗi thường gặp](https://xuanphi1702.github.io/green-space-evaluator/loi-thuong-gap/)

---

## Sản phẩm đầu ra

Plugin tự động tạo ra 5 sản phẩm chính:

1. **Raster mảng xanh đô thị**: Lớp raster thể hiện phân bố thảm thực vật mảng xanh sau bóc tách và lọc diện tích.
2. **Vector vùng phục vụ tương ứng R\***: Phạm vi không gian phục vụ bán kính lớn nhất $R^*$ thỏa mãn điều kiện sức tải mục tiêu.
3. **Vector không gian xây dựng trong vùng phục vụ**: Các polygon công trình có giao cắt với vùng phục vụ theo bán kính \(R^*\).
4. **Vector không gian xây dựng ngoài vùng phục vụ**: Các khối công trình nằm ngoài vùng phục vụ theo tiêu chí đánh giá.
5. **Vector thống kê theo đơn vị hành chính**: Lớp ranh giới tích hợp 6 trường chỉ số định lượng (`T_DanSo`, `S_MangXanh`, `S_XayDung`, `S_ThieuXanh`, `D_ThieuXanh`, `TL_ThieuXanh`).

---

## Tác giả

- **Huỳnh Hoàng Xuân Phi**
- Khoa Trắc địa, Bản đồ và Công trình – Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh (HCMUNRE)
- Năm thực hiện: 2026

---

## Tài liệu

Xem tài liệu hướng dẫn trực tuyến tại [https://xuanphi1702.github.io/green-space-evaluator/](https://xuanphi1702.github.io/green-space-evaluator/) hoặc truy cập các tệp hướng dẫn trong thư mục `docs/`.
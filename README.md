# Green Space Evaluator

**Green Space Evaluator** (tên hiển thị trong QGIS: *Urban Green Space Service Evaluator*) là Plugin chạy trên nền tảng QGIS hỗ trợ tự động hóa toàn diện quy trình đánh giá mức độ phục vụ của mảng xanh đô thị.

Plugin tích hợp chuỗi 5 module xử lý từ ảnh viễn thám Sentinel-2, dữ liệu ranh giới hành chính và dân số thống kê, lớp công viên/vườn hoa và dấu vết công trình xây dựng (footprint); mô hình hóa vùng phục vụ cho từng mảng xanh độc lập theo khoảng cách Euclid và chỉ tiêu diện tích bình quân đầu người; phân tích không gian xây dựng và tổng hợp 10 trường chỉ số định lượng theo đơn vị hành chính.

![Giao diện chính của Plugin Green Space Evaluator](docs/images/giao_dien_chinh_plugin.png)

---

## Tính năng chính

- **Tiền xử lý và chuẩn hóa dữ liệu**: Tự động kiểm tra tính hợp lệ hình học các lớp vector, chuẩn hóa trường dân số thống kê thành `POP_STAT`, đồng bộ hệ tọa độ phẳng dự chiếu (đơn vị mét) và tạo ảnh Sentinel-2 Stack 10 m (hỗ trợ lọc mây SCL).
- **Trích xuất mảng xanh từ ảnh Sentinel-2**: Tính toán chỉ số phổ MNDWI và SAVI, hỗ trợ tự động xác định ngưỡng SAVI từ lớp công viên mẫu (P10) hoặc nhập thủ công, phân tách mặt nước, lọc mảng xanh theo diện tích tối thiểu và định danh từng mảng xanh riêng biệt (`PATCH_ID`).
- **Mô hình hóa vùng phục vụ mảng xanh**: Phân bổ dân số thống kê xuống không gian xây dựng (`POP_ALLOC`), tính lưới khoảng cách Euclid từ biên mảng xanh, xác định quy mô dân số phục vụ mục tiêu ($P_{target,i} = S_i / C$) và áp dụng thuật toán tìm kiếm nhị phân (Binary Search) xác định bán kính phục vụ lớn nhất $R^*$ thỏa điều kiện mô hình.
- **Phân tích không gian xây dựng**: Phân tách footprint công trình thành các phần hình học (`footprint-part`), phân loại công trình nằm trong và ngoài vùng phục vụ, áp dụng thuật toán gán độc quyền (**Exclusive Assignment**) cho mảng xanh gần nhất để loại trừ hoàn toàn việc đếm lặp.
- **Tổng hợp chỉ tiêu định lượng theo đơn vị hành chính**: Tự động tính toán 10 trường chỉ số thống kê theo từng phường/xã, thực hiện 8 phép kiểm tra tính toàn vẹn dữ liệu (**Balance Checks**) và hỗ trợ xuất bảng số liệu định dạng Excel (`.xlsx`) hoặc CSV (`.csv`).

---

## 4 Nhóm dữ liệu đầu vào

1. **Ảnh Sentinel-2**: Kênh bắt buộc B03, B04, B08, B11; kênh tùy chọn B02, SCL.
2. **Ranh giới hành chính và dân số thống kê**: Lớp polygon ranh giới hành chính (CRS phẳng, đơn vị mét) kèm trường dân số thống kê chính thức (chuẩn hóa thành `POP_STAT`).
3. **Công viên/vườn hoa**: Vector polygon phạm vi công viên, vườn hoa làm mẫu trích xuất SAVI và bảo toàn mảng xanh.
4. **Dấu vết công trình xây dựng (Footprint)**: Vector polygon dấu vết chân công trình xây dựng (Google Open Buildings / OSM) đại diện cho không gian xây dựng vật lý.

---

## 5 Sản phẩm đầu ra chính

1. **Mảng xanh đô thị** (Raster GeoTIFF `.tif`): Lớp thực vật mảng xanh sau khi bóc tách, trừ mặt nước và lọc diện tích.
2. **Vùng phục vụ mảng xanh (R\*)** (Vector Polygon): Phạm vi không gian đệm bán kính lớn nhất $R^*$ thỏa điều kiện của mô hình cho từng mảng xanh độc lập.
3. **Không gian xây dựng trong vùng phục vụ** (Vector Polygon): Các phần hình học công trình (`footprint-part`) được phục vụ bởi mảng xanh đô thị.
4. **Không gian xây dựng ngoài vùng phục vụ** (Vector Polygon): Các phần hình học công trình (`footprint-part`) nằm ngoài vùng phục vụ mảng xanh.
5. **Thống kê theo đơn vị hành chính** (Vector Polygon): Lớp ranh giới tích hợp 10 trường chỉ số định lượng đánh giá mức độ phục vụ mảng xanh.

*(Bảng thống kê Excel/CSV là sản phẩm báo cáo bổ sung tùy chọn theo cấu hình của người dùng).*

---

## Yêu cầu môi trường

| Thành phần | Yêu cầu |
|---|---|
| **Phần mềm QGIS** | QGIS 3.28 LTR, 3.34 LTR hoặc 3.44+ |
| **Môi trường Python** | Python 3 tích hợp sẵn trong QGIS |
| **Thư viện bắt buộc** | `gdal/osgeo`, `numpy`, `scipy`, `matplotlib` |
| **Thư viện tùy chọn** | `openpyxl`, `pandas` (hỗ trợ xuất trực tiếp bảng báo cáo Excel `.xlsx`) |

---

## Cài đặt

1. Tải mã nguồn Plugin từ repository.
2. Sao chép thư mục `green_space_evaluator` vào thư mục plugins của QGIS:
   - **Windows**: `%APPDATA%\QGIS\QGIS3\profiles\default\python\plugins\`
3. Mở QGIS, vào **Plugins** → **Manage and Install Plugins...** → tích chọn kích hoạt **Urban Green Space Service Evaluator**.
4. Mở Plugin từ menu **Plugins** hoặc nhấn vào biểu tượng chiếc lá trên thanh công cụ.

👉 Xem hướng dẫn cài đặt chi tiết: [Tài liệu Cài đặt Plugin](https://xuanphi1702.github.io/green-space-evaluator/cai-dat/)

---

## Tài liệu hướng dẫn sử dụng

Tài liệu hướng dẫn sử dụng đầy đủ và chi tiết được biên soạn tại website chính thức:

- 📖 **Trang chủ tài liệu**: [https://xuanphi1702.github.io/green-space-evaluator/](https://xuanphi1702.github.io/green-space-evaluator/)
- 📥 [Chuẩn bị dữ liệu đầu vào](https://xuanphi1702.github.io/green-space-evaluator/du-lieu-dau-vao/)
- ⚙️ [Cài đặt tham số](https://xuanphi1702.github.io/green-space-evaluator/tham-so/)
- 🚀 [Chạy phân tích](https://xuanphi1702.github.io/green-space-evaluator/chay-phan-tich/)
- 📊 [Sản phẩm & chỉ tiêu thống kê](https://xuanphi1702.github.io/green-space-evaluator/san-pham/)
- 💡 [Lưu ý khi sử dụng](https://xuanphi1702.github.io/green-space-evaluator/luu-y/)
- 🛠️ [Lỗi thường gặp và cách xử lý](https://xuanphi1702.github.io/green-space-evaluator/loi-thuong-gap/)

---

## Tác giả

- **Huỳnh Hoàng Xuân Phi**
- Khoa Trắc địa, Bản đồ và Công trình – Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh (HCMUNRE)
- Năm thực hiện: 2026

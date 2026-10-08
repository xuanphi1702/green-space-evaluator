# Green Space Evaluator

**Green Space Evaluator** (tên hiển thị trong QGIS: *Urban Green Space Service Evaluator*) là Plugin chạy trên nền tảng QGIS hỗ trợ tự động hóa toàn diện quy trình đánh giá mức độ phục vụ của mảng xanh đô thị.

Plugin tích hợp chuỗi 5 module xử lý từ ảnh viễn thám Sentinel-2, dữ liệu ranh giới hành chính kèm dân số thống kê, và dấu vết công trình xây dựng (building footprints); mô hình hóa vùng phục vụ cho từng mảng xanh độc lập theo bán kính khả thi $R^*$ dựa trên dân số phân bổ (`POP_ALLOC`) và chỉ tiêu diện tích mảng xanh bình quân đầu người tối thiểu ($C_{\min}$); hợp nhất vùng phục vụ chung (`SERVICE_UNION`), phân tích không gian xây dựng (`SERVED`/`OUTSIDE`) và tổng hợp 10 trường chỉ số định lượng theo đơn vị hành chính.

![Giao diện chính của Plugin Green Space Evaluator](docs/images/giao_dien_chinh_plugin.png)

---

## Tính năng chính

- **Tiền xử lý và chuẩn hóa dữ liệu**: Tự động kiểm tra tính hợp lệ hình học các lớp vector, chuẩn hóa trường dân số thống kê của đơn vị hành chính thành `POP_STAT`, chuyển đổi và đồng bộ CRS dự chiếu (đơn vị mét) và tạo ảnh Sentinel-2 Stack 10 m (hỗ trợ lọc mây SCL).
- **Trích xuất mảng xanh từ ảnh Sentinel-2**: Tính toán chỉ số phổ MNDWI (ngưỡng 0.000) loại trừ mặt nước, tính SAVI ($L=0.50$, ngưỡng mặc định 0.165) bóc tách thảm thực vật, lọc mảng xanh theo diện tích tối thiểu ($A_{\min} = 5000\text{ m}^2$) và định danh từng mảng xanh riêng biệt (`PATCH_ID`).
- **Mô hình hóa vùng phục vụ mảng xanh**: Cắt chia footprint theo ranh giới hành chính và phân bổ dân số thống kê theo tỷ lệ diện tích (`POP_ALLOC`), xây dựng chỉ mục không gian `QgsSpatialIndex`, giới hạn dân số phục vụ tối đa ($P_{\max, i} = S_i / C_{\min}$) và áp dụng phương pháp tìm kiếm hai giai đoạn (Giai đoạn 1: Khoanh vùng nghiệm theo bước $\Delta d = 50\text{ m}$; Giai đoạn 2: Tinh chỉnh nghiệm bằng tìm kiếm nhị phân theo ngưỡng hội tụ $e = 1.0\text{ người}$) để xác định bán kính khả thi $R^* = R_{\text{low}}$ cho từng mảng xanh độc lập (`OUT_BUFFER`).
- **Phân tích không gian xây dựng**: Hợp nhất các vùng phục vụ riêng (`OUT_BUFFER`) thành vùng phục vụ chung `SERVICE_UNION` (loại trừ hoàn toàn việc đếm lặp vùng chồng lấn), phân rã không gian xây dựng thành các phần diện tích xây dựng nằm trong vùng phục vụ (`SERVED`) và ngoài vùng phục vụ (`OUTSIDE`), bảo toàn dân số theo tỷ lệ diện tích (`POP_FRAGMENT` qua `SOURCE_POP_ALLOC`).
- **Tổng hợp chỉ tiêu định lượng theo đơn vị hành chính**: Tự động tính toán 10 trường chỉ số thống kê theo từng phường/xã, thực hiện 8 phép kiểm tra tính toàn vẹn dữ liệu (**Balance Checks**) và hỗ trợ xuất bảng số liệu định dạng Excel (`.xlsx`) hoặc CSV (`.csv`).

---

## 4 Nhóm dữ liệu đầu vào

1. **Ảnh Sentinel-2**: Kênh bắt buộc B03, B04, B08, B11; kênh tùy chọn B02, SCL.
2. **Ranh giới hành chính và dân số thống kê**: Lớp polygon ranh giới hành chính (CRS dự chiếu phẳng, đơn vị mét) kèm trường dân số thống kê chính thức (chuẩn hóa thành `POP_STAT`).
3. **Dấu vết công trình xây dựng (Footprint)**: Vector polygon chân công trình xây dựng (Google Open Buildings) đại diện cho không gian xây dựng vật lý.
4. **Tham số phân tích**: Ngưỡng MNDWI, SAVI, diện tích mảng xanh tối thiểu ($A_{\min}$), bán kính tối đa ($R_{\max}$), chỉ tiêu mảng xanh bình quân ($C_{\min}$).

---

## 5 Sản phẩm đầu ra chính

![Các sản phẩm chính sau khi chạy Plugin Green Space Evaluator](docs/images/ket_qua.png)

1. **Mảng xanh đô thị** (Raster GeoTIFF `.tif`, `OUT_PARK`): Lớp thực vật mảng xanh sau khi bóc tách, trừ mặt nước và lọc diện tích.
2. **Vùng phục vụ mảng xanh** (Vector Polygon, `SERVICE_UNION`): Vùng phục vụ chung được tạo bằng cách hợp nhất các vùng phục vụ riêng của từng mảng xanh theo bán kính khả thi $R^*$ (loại bỏ hoàn toàn đếm lặp diện tích chồng lấn).
3. **Không gian xây dựng trong vùng phục vụ** (Vector Polygon, `SERVED`): Các phần diện tích công trình được phục vụ bởi mảng xanh đô thị kèm trường dân số phân bổ `POP_FRAGMENT`.
4. **Không gian xây dựng ngoài vùng phục vụ** (Vector Polygon, `OUTSIDE`): Các phần diện tích công trình nằm ngoài vùng phục vụ mảng xanh.
5. **Thống kê kết quả theo đơn vị hành chính** (Vector Polygon, `OUT_STATS`): Lớp ranh giới tích hợp 10 trường chỉ số định lượng đánh giá mức độ phục vụ mảng xanh.

*(Lớp vùng phục vụ riêng `OUT_BUFFER` của từng mảng xanh là sản phẩm trung gian/bổ sung; Bảng thống kê Excel/CSV là sản phẩm báo cáo bổ sung tùy chọn theo cấu hình của người dùng).*

---

## Yêu cầu môi trường

| Thành phần | Yêu cầu |
|---|---|
| **Phần mềm QGIS** | QGIS 3.44.12+ (khuyến nghị phiên bản chuẩn LTR) |
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

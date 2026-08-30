# Sản phẩm và chỉ tiêu thống kê

Sau khi thực thi phân tích, Plugin **Green Space Evaluator** sẽ xuất ra **5 sản phẩm đầu ra chính** cùng một hệ thống các **sản phẩm trung gian** phục vụ kiểm tra và đối soát chất lượng dữ liệu. Trang này cung cấp tài liệu tham chiếu chi tiết về cấu trúc dữ liệu, ý nghĩa các trường thống kê và cách diễn giải kết quả thu được.

---

## 1. Sản phẩm chính

Năm sản phẩm chính được cấu hình tại thẻ **Sản phẩm đầu ra** trên giao diện Plugin:

| Sản phẩm | Loại dữ liệu | Định dạng | Vai trò / Nội dung |
|---|---|---|---|
| **1. Raster mảng xanh đô thị** | Raster | `.tif` (GeoTIFF) | Lớp raster thể hiện phân bố thảm thực vật mảng xanh đô thị sau bóc tách và lọc diện tích |
| **2. Vector vùng phục vụ tương ứng R\*** | Vector (Polygon) | `.gpkg`, `.shp`, `.geojson` | Phạm vi không gian đệm bán kính lớn nhất $R^*$ thỏa mãn điều kiện sức tải mục tiêu |
| **3. Vector không gian xây dựng trong vùng phục vụ** | Vector (Polygon) | `.gpkg`, `.shp`, `.geojson` | Các polygon công trình xây dựng có giao cắt với vùng phục vụ |
| **4. Vector không gian xây dựng ngoài vùng phục vụ** | Vector (Polygon) | `.gpkg`, `.shp`, `.geojson` | Các polygon công trình xây dựng nằm ngoài vùng phục vụ theo tiêu chí của Plugin |
| **5. Vector thống kê theo đơn vị hành chính** | Vector (Polygon) | `.gpkg`, `.shp`, `.geojson` | Lớp ranh giới tích hợp 6 trường chỉ số định lượng đánh giá mảng xanh |

### 1.1. Raster mảng xanh đô thị
- **Bản chất dữ liệu**: Là kết quả tổng hợp của thảm thực vật được bảo toàn nguyên vẹn trong ranh giới công viên mẫu và thảm thực vật ngoài công viên đã qua bộ lọc diện tích tối thiểu (`MIN_GREEN_AREA = 5000 m²`).
- **Vai trò trong quy trình**: Đóng vai trò là nguồn phát mảng xanh để tính ma trận khoảng cách hình học Euclid và làm tử số $S_{UGS}$ trong hàm tính sức tải mảng xanh.
- **Cách kiểm tra**: Mở layer raster trong QGIS để quan sát sự phân bố của các mảng cây xanh tập trung so với ảnh vệ tinh nền.
- **Lưu ý**: Lớp dữ liệu này thể hiện hiện trạng thảm thực vật phản xạ phổ tại thời điểm chụp ảnh Sentinel-2, không đồng nhất hoàn toàn với diện tích đất cây xanh quy hoạch theo hồ sơ pháp lý.

### 1.2. Vùng phục vụ tương ứng R*
- **Bản chất dữ liệu**: Là vùng đệm không gian được tạo ra từ việc mở rộng các mảng xanh đô thị với bán kính $R^*$.
- **Mô hình tính toán**: Vùng phục vụ được xác định theo mô hình khoảng cách Euclid của Plugin.

!!! note "Cách hiểu vùng phục vụ"
    $R^*$ là bán kính lớn nhất trong miền tìm kiếm được cấu hình mà điều kiện sức tải vẫn được đáp ứng. Vùng phục vụ được xác định theo khoảng cách Euclid trong mô hình của Plugin và không đại diện trực tiếp cho khoảng cách đi bộ theo mạng lưới giao thông.

### 1.3. Không gian xây dựng trong vùng phục vụ
- **Bản chất dữ liệu**: Tập hợp các polygon công trình xây dựng có quan hệ không gian giao cắt (`Intersects`) với lớp vùng phục vụ bán kính $R^*$.
- **Ý nghĩa**: Đại diện cho các công trình nằm trong vùng phục vụ theo khoảng cách Euclid và điều kiện sức tải của mô hình.

### 1.4. Không gian xây dựng ngoài vùng phục vụ
- **Bản chất dữ liệu**: Tập hợp các polygon công trình xây dựng nằm tách biệt hoàn toàn (`Disjoint`) ngoài vùng phục vụ theo tiêu chí của Plugin.
- **Ý nghĩa**: Đại diện cho các công trình nằm ngoài vùng phục vụ theo khoảng cách Euclid và điều kiện sức tải của mô hình.

### 1.5. Thống kê theo đơn vị hành chính
- **Bản chất dữ liệu**: Lớp vector ranh giới hành chính (phường/xã) kế thừa cấu trúc hình học từ dữ liệu khu vực nghiên cứu đầu vào (`MASK_LAYER`).
- **Nội dung tích hợp**: Chứa toàn bộ 6 trường chỉ số định lượng được tính toán tự động qua chuỗi thuật toán Zonal Statistics liên hoàn.

---

## 2. Sản phẩm trung gian

Các sản phẩm trung gian được tạo ra trong chuỗi xử lý 5 module nhằm hỗ trợ kiểm tra chất lượng, truy vết quy trình và phục vụ phân tích chuyên sâu:

| Sản phẩm trung gian | Định dạng | Vai trò kỹ thuật |
|---|---|---|
| **Ảnh Sentinel-2 gộp kênh** | Raster (`.tif`) | Tệp raster đa phổ đã cắt theo ranh giới và chuẩn hóa độ phân giải 10m |
| **Raster dân số chuẩn hóa** | Raster (`.tif`) | Raster WorldPop đã được chuyển đổi về hệ tọa độ và độ phân giải thống nhất với khu vực nghiên cứu (10 m) và bảo toàn tổng dân số.|
| **Raster MNDWI** | Raster (`.tif`) | Ảnh chỉ số khác biệt nước cải tiến |
| **Raster mặt nước** | Raster (`.tif`) | Mặt nạ nhị phân các vùng mặt nước sông ngòi, kênh rạch |
| **Raster SAVI** | Raster (`.tif`) | Ảnh chỉ số thực vật điều chỉnh theo đất |
| **Raster SAVI (loại bỏ nước)** | Raster (`.tif`) | Chỉ số SAVI trên đất liền sau khi trừ mặt nạ nước |
| **Raster thực vật thô** | Raster (`.tif`) | Mặt nạ thảm thực vật ban đầu trước khi lọc diện tích |
| **Ảnh Histogram MNDWI & SAVI** | Ảnh (`.png`) | Biểu đồ phân bố tần suất giá trị phổ hỗ trợ đánh giá ngưỡng |
| **Raster dân số trong công trình** | Raster (`.tif`) | Dân số WorldPop được trích xuất trong phạm vi không gian xây dựng |

!!! tip "Quy tắc lưu sản phẩm trung gian"
    Các sản phẩm trung gian được tự động xử lý dưới dạng tệp tạm (temporary files). Người dùng chỉ cần chỉ định đường dẫn lưu trong cửa sổ **Cài đặt** (Tab *Sản phẩm trung gian*) khi có nhu cầu lưu trữ hoặc kiểm tra chi tiết từng bước viễn thám; việc để trống các trường này không ảnh hưởng đến kết quả chạy chính.

---

## 3. Các trường thống kê

Bảng thuộc tính của lớp vector thống kê hành chính (`OUT_STATS`) chứa 6 trường chỉ số định lượng:

| Tên trường | Nội dung chỉ số | Đơn vị | Kiểu dữ liệu |
|---|---|---|---|
| `T_DanSo` | Tổng dân số ước tính của đơn vị hành chính | người | Số nguyên (Integer) |
| `S_MangXanh` | Tổng diện tích mảng xanh đô thị | ha | Số thực (Double, 3 chữ số thập phân) |
| `S_XayDung` | Tổng diện tích không gian xây dựng | ha | Số thực (Double, 3 chữ số thập phân) |
| `S_ThieuXanh` | Diện tích không gian xây dựng nằm ngoài vùng phục vụ | ha | Số thực (Double, 3 chữ số thập phân) |
| `D_ThieuXanh` | Dân số ước tính nằm ngoài vùng phục vụ | người | Số nguyên (Integer) |
| `TL_ThieuXanh` | Tỷ lệ dân số ước tính nằm ngoài vùng phục vụ | % | Số thực (Double, 3 chữ số thập phân) |

!!! note "Bản chất chỉ tiêu dân số"
    Các chỉ tiêu dân số (`T_DanSo`, `D_ThieuXanh`, `TL_ThieuXanh`) là giá trị ước tính từ dữ liệu không gian WorldPop và được diễn giải trong phạm vi mô hình phân tích của Plugin.

---

## 4. Cách đọc các chỉ tiêu

Khi mở bảng thuộc tính của lớp thống kê hành chính hoặc xem file báo cáo:

- **`S_MangXanh (ha)`**: Thể hiện quy mô diện tích mảng xanh được bóc tách theo cấu hình và dữ liệu đầu vào của Plugin.
- **`S_ThieuXanh (ha)`**: Thể hiện diện tích không gian xây dựng nằm ngoài vùng phục vụ được mô hình hóa theo bán kính $R^*$.
- **`D_ThieuXanh(người)`**: Thể hiện số lượng dân số ước tính phân bố trong các khối nhà/công trình nằm ngoài vùng phục vụ mảng xanh.
- **`TL_ThieuXanh (%)`**: Phản ánh tỷ lệ phần trăm dân số ước tính nằm ngoài vùng phục vụ của mảng xanh so với tổng dân số của từng đơn vị hành chính:

$$
\text{TL_ThieuXanh} = \frac{\text{D_ThieuXanh}}{\text{T_DanSo}} \times 100
$$

---

## 5. Xuất bảng thống kê

Bên cạnh lớp vector GIS, Plugin hỗ trợ xuất bảng số liệu thống kê ra file độc lập:

- **Định dạng hỗ trợ**: **Excel (`.xlsx`)** hoặc **CSV (`.csv`)**.
- **Cấu trúc tệp xuất**: Mỗi hàng tương ứng một đơn vị hành chính và chứa các chỉ tiêu thống kê.
- **Mã hóa tiếng Việt**: Tệp `.csv` được tự động ghi với chuẩn mã hóa `UTF-8-SIG`, giúp mở trực tiếp trên Microsoft Excel mà không bị lỗi hiển thị phông chữ tiếng Việt.

---

## 6. Lưu ý khi diễn giải kết quả

!!! warning "Lưu ý khi diễn giải kết quả"
    Các sản phẩm và chỉ tiêu thống kê là kết quả tính toán từ dữ liệu đầu vào và mô hình phân tích của Plugin. Đặc biệt, các chỉ tiêu dân số và “ngoài vùng phục vụ” không nên được diễn giải trực tiếp là số liệu kiểm kê thực địa hoặc số người thực tế hoàn toàn không thể tiếp cận không gian xanh.

---

## Liên kết liên quan

- [Dữ liệu đầu vào](du-lieu-dau-vao.md): Xem yêu cầu chuẩn bị 5 nhóm dữ liệu.
- [Cài đặt tham số](tham-so.md): Xem ý nghĩa các tham số $R_{max}$, $C_{min}$, ngưỡng SAVI và diện tích lọc.
- [Chạy phân tích](chay-phan-tich.md): Xem hướng dẫn từng bước thực thi Plugin và nạp kết quả.

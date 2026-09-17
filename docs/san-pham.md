# Sản phẩm và chỉ tiêu thống kê

Sau khi hoàn tất quy trình phân tích, Plugin **Green Space Evaluator** sẽ tạo ra **5 sản phẩm đầu ra chính**, tùy chọn **bảng báo cáo thống kê** (Excel/CSV) cùng hệ thống **sản phẩm trung gian** phục vụ kiểm tra và nghiên cứu chuyên sâu. Trang này cung cấp thông tin chi tiết về cấu trúc dữ liệu, ý nghĩa các trường thuộc tính thống kê và hướng dẫn diễn giải kết quả.

---

## 1. 5 sản phẩm đầu ra chính

Năm sản phẩm chính được cấu hình tại thẻ **Sản phẩm đầu ra** trên giao diện Plugin:

| STT | Tên sản phẩm | Loại dữ liệu | Định dạng hỗ trợ | Nội dung & Vai trò kỹ thuật |
|---|---|---|---|---|
| **1** | **Mảng xanh đô thị** | Raster | `.tif` (GeoTIFF) | Lớp raster thể hiện phân bố thảm thực vật mảng xanh sau khi phân loại chỉ số phổ, trừ mặt nước và lọc diện tích. |
| **2** | **Vùng phục vụ mảng xanh (R\*)** | Vector (Polygon) | `.gpkg`, `.shp`, `.geojson` | Phạm vi không gian đệm bán kính lớn nhất $R^*$ thỏa điều kiện của mô hình cho từng mảng xanh độc lập. |
| **3** | **Không gian xây dựng trong vùng phục vụ** | Vector (Polygon) | `.gpkg`, `.shp`, `.geojson` | Các phần hình học công trình (`footprint-part`) nằm trong phạm vi vùng phục vụ bán kính $R^*$ của mảng xanh. |
| **4** | **Không gian xây dựng ngoài vùng phục vụ** | Vector (Polygon) | `.gpkg`, `.shp`, `.geojson` | Các phần hình học công trình (`footprint-part`) nằm ngoài phạm vi vùng phục vụ mảng xanh. |
| **5** | **Thống kê theo đơn vị hành chính** | Vector (Polygon) | `.gpkg`, `.shp`, `.geojson` | Lớp ranh giới tích hợp 10 trường chỉ số định lượng đánh giá toàn diện mức độ phục vụ mảng xanh. |

---

## 2. Chi tiết từng sản phẩm chính

### 2.1. Mảng xanh đô thị
- **Bản chất dữ liệu**: Lớp raster nhị phân (1: Mảng xanh, 0: Khác) thể hiện thảm thực vật mảng xanh tập trung sau khi bóc tách bằng chỉ số SAVI, loại trừ mặt nước (MNDWI > 0.0) và lọc bỏ các mảng nhỏ ngoài công viên (< 5000 m²). Toàn bộ mảng xanh trong ranh công viên mẫu luôn được bảo toàn nguyên vẹn.
- **Vai trò kỹ thuật**: Đóng vai trò là nguồn phát mảng xanh để xây dựng lưới khoảng cách Euclid và cung cấp diện tích $S_i$ của từng patch trong mô hình xác định bán kính phục vụ.
- **Lưu ý**: Lớp dữ liệu thể hiện hiện trạng thảm thực vật phản xạ phổ tại thời điểm chụp ảnh Sentinel-2, không đồng nhất hoàn toàn với diện tích đất cây xanh quy hoạch theo hồ sơ pháp lý.

### 2.2. Vùng phục vụ mảng xanh (R*)
- **Bản chất dữ liệu**: Lớp vector polygon vùng đệm không gian được tạo ra từ việc mở rộng từng mảng xanh độc lập với bán kính phục vụ lớn nhất $R_i^*$.
- **Đặc điểm quan trọng**:
    - Mỗi mảng xanh có một bán kính $R^*$ riêng biệt và một polygon vùng phục vụ riêng biệt gắn liền với mã định danh `PATCH_ID`.
    - Các polygon vùng phục vụ được giữ nguyên độc lập, **không thực hiện gộp (merge) hay thu nhỏ (shrink)** ngay cả khi các vùng đệm có sự chồng lấn không gian.
- **Bảng thuộc tính của lớp Vùng phục vụ R\* (`SERVICE_AREA`)**:
    - `PATCH_ID` (Integer): Mã định danh số nguyên duy nhất của từng mảng xanh.
    - `R_STAR` (Double): Bán kính phục vụ lớn nhất thỏa điều kiện tìm kiếm được (m).
    - `P_TARGET` (Double): Quy mô dân số mục tiêu được xác định từ diện tích mảng xanh và chỉ tiêu C ($P_{target,i} = S_i / C$).
    - `STATUS` (String): Trạng thái kết quả tìm kiếm bán kính (`RMAX`, `BINARY_SEARCH`, `NO_SOLUTION`).
    - `AREA_M2` (Double): Diện tích của mảng xanh ($\text{m}^2$).
    - `AREA_HA` (Double): Diện tích của mảng xanh (ha).
    - `P_RMIN` (Double): Dân số phân bổ tiếp cận tại bán kính $R = 0\text{ m}$.
    - `P_RMAX` (Double): Dân số phân bổ tiếp cận tại bán kính $R = R_{\max}$.
    - `P_AT_RSTAR` (Double): Tổng dân số được phân bổ nằm trong phạm vi khoảng cách $R^*$ của mảng xanh.
    - `ITERATIONS` (Integer): Số bước lặp của thuật toán tìm kiếm nhị phân.

### 2.3. Không gian xây dựng trong vùng phục vụ
- **Bản chất dữ liệu**: Tập hợp các đối tượng hình học công trình (`footprint-part`) nằm trong phạm vi vùng phục vụ bán kính $R^*$ của mảng xanh.
- **Cơ chế gán độc quyền (Exclusive Assignment)**: Khi một footprint-part nằm trong phạm vi giao cắt của nhiều vùng phục vụ, Plugin tự động gán đối tượng đó cho mảng xanh gần nhất. Nhờ đó, mỗi footprint-part chỉ gắn với một `PATCH_ID` duy nhất, loại bỏ hoàn toàn việc đếm lặp dân số hoặc diện tích.
- **Ý nghĩa**: Đại diện cho không gian xây dựng vật lý được tiếp cận mảng xanh theo khoảng cách Euclid và chỉ tiêu diện tích bình quân của mô hình.

### 2.4. Không gian xây dựng ngoài vùng phục vụ
- **Bản chất dữ liệu**: Tập hợp các đối tượng hình học công trình (`footprint-part`) nằm ngoài phạm vi vùng phục vụ của tất cả các mảng xanh.
- **Ý nghĩa**: Thể hiện các khu vực xây dựng còn thiếu hụt không gian xanh theo tiêu chí khoảng cách và chỉ tiêu diện tích của mô hình, hỗ trợ định hướng các vị trí ưu tiên bổ sung công viên, vườn hoa mới.

### 2.5. Thống kê theo đơn vị hành chính
- **Bản chất dữ liệu**: Lớp vector ranh giới hành chính (phường/xã) kế thừa cấu trúc hình học từ lớp ranh giới đầu vào, được tích hợp trực tiếp 10 trường chỉ số định lượng.
- **Phương pháp tổng hợp**: Tổng hợp trực tiếp từ các lớp vector kết quả hình học và số liệu phân bổ không gian.

---

## 3. Bảng báo cáo thống kê bổ sung (CSV / XLSX)

### 3.1. Bảng thống kê theo đơn vị hành chính
- **Định dạng hỗ trợ**: **Excel (`.xlsx`)** hoặc **CSV (`.csv`)**.
- **Cấu trúc dữ liệu**: Mỗi hàng tương ứng với một đơn vị hành chính và chứa đầy đủ 10 trường chỉ số định lượng.
- **Chuẩn mã hóa**: Tệp `.csv` được xuất dưới định dạng mã hóa `UTF-8-SIG`, đảm bảo mở trực tiếp bằng Microsoft Excel hiển thị tiếng Việt chuẩn xác không bị lỗi phông chữ.

### 3.2. Bảng thống kê chi tiết theo mảng xanh (`patch_service_statistics`)
Tệp thống kê chi tiết ở cấp độ từng patch (`patch_service_statistics.csv` / `.xlsx`) gồm đúng **13 trường chuẩn hóa trong mã nguồn**:
1. `PATCH_ID`: Mã định danh mảng xanh.
2. `AREA_M2`: Diện tích mảng xanh ($\text{m}^2$).
3. `AREA_HA`: Diện tích mảng xanh (ha).
4. `R_STAR_M`: Bán kính phục vụ lớn nhất thỏa điều kiện $R^*$ (m).
5. `STATUS`: Trạng thái giải bán kính (`RMAX`, `BINARY_SEARCH`, `NO_SOLUTION`).
6. `P_TARGET`: Quy mô dân số mục tiêu được xác định từ diện tích mảng xanh và chỉ tiêu C ($S_i / C$).
7. `P_AT_RSTAR`: Tổng dân số được phân bổ nằm trong phạm vi khoảng cách $R^*$ của mảng xanh.
8. `N_SERVED_PARTS`: Số lượng footprint-part được gán phục vụ độc quyền cho mảng xanh.
9. `SERVED_BUILDING_AREA_M2`: Tổng diện tích xây dựng được gán phục vụ ($\text{m}^2$).
10. `SERVED_BUILDING_AREA_HA`: Tổng diện tích xây dựng được gán phục vụ (ha).
11. `SERVED_POP`: Tổng dân số được phân bổ phục vụ độc quyền của mảng xanh.
12. `MEAN_DIST_M`: Khoảng cách trung bình từ các footprint-part được gán đến mảng xanh (m).
13. `MAX_DIST_M`: Khoảng cách lớn nhất từ các footprint-part được gán đến mảng xanh (m).

---

## 4. 10 trường thuộc tính thống kê hành chính

Bảng thuộc tính của lớp vector thống kê hành chính chứa đúng **10 trường chỉ số định lượng chuẩn hóa**:

| STT | Tên trường | Nội dung chỉ số | Đơn vị | Kiểu dữ liệu | Ý nghĩa & Bản chất khoa học |
|---|---|---|:---:|:---:|---|
| 1 | `T_DanSo` | Tổng dân số thống kê của đơn vị hành chính | người | Integer | Dân số thống kê chính thức của đơn vị hành chính (`POP_STAT`). |
| 2 | `S_MangXanh` | Tổng diện tích mảng xanh phân bổ cho đơn vị hành chính | ha | Double (3) | Tổng diện tích mảng xanh được phân bổ cho đơn vị hành chính theo tỷ trọng diện tích giao cắt của từng mảng xanh với ranh giới hành chính. |
| 3 | `S_XayDung` | Tổng diện tích không gian xây dựng | ha | Double (3) | Tổng diện tích các footprint-part nằm trong đơn vị hành chính. |
| 4 | `S_DaPhucVu` | Diện tích không gian xây dựng được phục vụ | ha | Double (3) | Tổng diện tích các footprint-part nằm trong vùng phục vụ mảng xanh. |
| 5 | `S_ThieuXanh` | Diện tích không gian xây dựng thiếu mảng xanh | ha | Double (3) | Tổng diện tích các footprint-part nằm ngoài vùng phục vụ mảng xanh. |
| 6 | `D_DaPhucVu` | Dân số phân bổ được phục vụ mảng xanh | người | Integer | Tổng dân số được phân bổ (`POP_ALLOC`) cho các footprint-part nằm trong vùng phục vụ mảng xanh. |
| 7 | `D_ThieuXanh` | Dân số phân bổ thiếu mảng xanh | người | Integer | Tổng dân số được phân bổ (`POP_ALLOC`) cho các footprint-part nằm ngoài vùng phục vụ mảng xanh. |
| 8 | `TL_DaPhucVu` | Tỷ lệ dân số phân bổ được phục vụ | % | Double (3) | Tỷ lệ phần trăm dân số phân bổ được phục vụ so với tổng dân số của đơn vị hành chính. |
| 9 | `TL_ThieuXanh` | Tỷ lệ dân số phân bổ thiếu mảng xanh | % | Double (3) | Tỷ lệ phần trăm dân số phân bổ nằm ngoài vùng phục vụ so với tổng dân số của đơn vị hành chính. |
| 10 | `N_Patch_DuocPhucVu` | Số lượng mảng xanh tham gia phục vụ | mảng | Integer | Số lượng mảng xanh độc lập có vùng phục vụ tham gia phục vụ không gian xây dựng của đơn vị hành chính. |

---

## 5. Công thức tính toán các chỉ tiêu

Các chỉ tiêu thống kê được tính toán tuần tự theo các công thức khoa học sau:

### 5.1. Cân bằng diện tích xây dựng
Tổng diện tích xây dựng của đơn vị hành chính là tổng diện tích của phần được phục vụ và phần thiếu xanh:

$$
S_{XayDung} = S_{DaPhucVu} + S_{ThieuXanh}
$$

### 5.2. Cân bằng dân số phân bổ
Tổng dân số thống kê của đơn vị hành chính bằng tổng dân số phân bổ được phục vụ và dân số phân bổ thiếu xanh:

$$
T_{DanSo} = D_{DaPhucVu} + D_{ThieuXanh}
$$

### 5.3. Tỷ lệ dân số phân bổ được phục vụ và thiếu mảng xanh

$$
TL_{DaPhucVu} = \frac{D_{DaPhucVu}}{T_{DanSo}} \times 100
$$

$$
TL_{ThieuXanh} = \frac{D_{ThieuXanh}}{T_{DanSo}} \times 100
$$

Trong đó:

$$
TL_{DaPhucVu} + TL_{ThieuXanh} = 100\%
$$

---

## 6. Sản phẩm trung gian

Khi người dùng cấu hình đường dẫn lưu tại tab **Sản phẩm trung gian** trong cửa sổ Cài đặt, Plugin có thể xuất thêm các tệp sau (nếu để trống, các tệp tạm thời được tạo trong thư mục tạm của hệ thống `tempfile.gettempdir()`):

| Sản phẩm trung gian | Định dạng | Vai trò kỹ thuật |
|---|---|---|
| **Ảnh Sentinel-2 Stack** | Raster (`.tif`) | Ảnh đa kênh ghép gồm các kênh bắt buộc và tùy chọn cắt theo ranh giới, độ phân giải 10 m. |
| **Raster MNDWI** | Raster (`.tif`) | Ảnh chỉ số khác biệt nước cải tiến. |
| **Raster mặt nước** | Raster (`.tif`) | Mặt nạ nhị phân thể hiện bề mặt nước sông, suối, hồ kênh rạch. |
| **Raster SAVI** | Raster (`.tif`) | Ảnh chỉ số thực vật điều chỉnh theo đất trên toàn cảnh. |
| **Raster SAVI (loại bỏ nước)** | Raster (`.tif`) | Chỉ số SAVI trên đất liền sau khi đã trừ mặt nạ nước. |
| **Raster thực vật thô** | Raster (`.tif`) | Mặt nạ thảm thực vật ban đầu trước khi áp dụng bộ lọc diện tích tối thiểu. |
| **Biểu đồ Histogram MNDWI & SAVI** | Ảnh (`.png`) | Biểu đồ phân bố tần suất giá trị phổ hỗ trợ đánh giá độ tin cậy của ngưỡng bóc tách. |
| **Dấu vết công trình phân bổ dân số** | Vector (`.gpkg`) | Lớp footprint-part tích hợp trường dân số phân bổ `POP_ALLOC`. |
| **Bảng thống kê mảng xanh** | File (`.xlsx`/`.csv`) | Bảng thuộc tính chi tiết 13 trường của từng patch: diện tích, bán kính $R^*$, trạng thái, dân số tiếp cận. |

---

## 7. Lưu ý khi diễn giải kết quả

!!! warning "Lưu ý phương pháp luận khi đọc kết quả"
    1. **Bản chất dân số:** Các chỉ tiêu `D_DaPhucVu`, `D_ThieuXanh`, `TL_ThieuXanh` là giá trị **dân số được phân bổ theo không gian** từ số liệu thống kê cấp hành chính dựa trên diện tích footprint công trình. Không được diễn giải đây là số lượng cư dân điều tra thực địa hay số hộ gia đình thực tế.
    2. **Khái niệm phục vụ:** Thuật ngữ "được phục vụ" hoặc "thiếu mảng xanh" phản ánh kết quả mô hình hóa không gian bằng khoảng cách hình học Euclid và chỉ tiêu diện tích bình quân đầu người của mô hình; không đồng nhất hoàn toàn với việc người dân có tiếp cận được không gian xanh trên thực tế hay không (do phụ thuộc vào mạng lưới đường đi, lối vào công viên và rào chắn thực địa).

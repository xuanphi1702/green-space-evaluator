# Chạy phân tích

Trang này hướng dẫn chi tiết từng bước thực thi quy trình phân tích trên Plugin **Green Space Evaluator**, cấu hình cửa sổ Cài đặt, theo dõi nhật ký tiến trình và kiểm tra các kết quả đầu ra trên giao diện QGIS.

---

## 1. Các bước thực hiện tổng quan

```
[BƯỚC 1: Nạp 4 nhóm dữ liệu] ➔ [BƯỚC 2: Cài đặt tham số (tùy chọn)] ➔ [BƯỚC 3: Chọn nơi lưu sản phẩm] ➔ [BƯỚC 4: Nhấn "Phân tích"]
```

---

## 2. Mở Plugin

Người dùng có thể mở giao diện Plugin trên QGIS theo một trong hai cách:

1. **Từ menu**: Vào **Plugins** → **Urban Green Space Service Evaluator** → chọn **Đánh giá mức độ phục vụ mảng xanh đô thị**.
2. **Từ thanh công cụ**: Nhấn trực tiếp vào biểu tượng **chiếc lá** trên thanh công cụ của QGIS.

Cửa sổ chính của Plugin hiển thị 3 tab chức năng:
- **Dữ liệu đầu vào**: Nạp 4 nhóm dữ liệu không gian.
- **Sản phẩm đầu ra**: Chỉ định đường dẫn lưu trữ cho 5 sản phẩm chính và bảng báo cáo bổ sung.
- **Nhật ký tiến trình**: Theo dõi tiến độ, thông báo và tổng kết thời gian thực thi.

---

## 3. Thao tác tại tab Dữ liệu đầu vào

Tại thẻ **Dữ liệu đầu vào**, nạp lần lượt 4 nhóm dữ liệu:

<div class="guide-figure" markdown>
![Tab Dữ liệu đầu vào trên giao diện chính của Plugin](images/giao_dien_chinh_plugin.png)
<div class="guide-caption"><strong>Hình 1.</strong> Giao diện nạp dữ liệu đầu vào của Plugin</div>
</div>

1. **Các kênh ảnh Sentinel-2**: Nhấn nút **`...`** để mở hộp thoại chọn kênh. Tích chọn các layer ảnh đang mở trong QGIS hoặc nhấn *Thêm file từ ổ đĩa* để nạp tối thiểu 4 kênh bắt buộc (**B03, B04, B08, B11**); có thể thêm **B02** và **SCL** nếu có.
2. **Ranh giới hành chính và dân số thống kê**:
   - Chọn layer polygon ranh giới hành chính trong danh sách thả xuống.
   - Tại ô chọn trường dân số, chọn trường thuộc tính chứa số liệu dân số thống kê chính thức của từng đơn vị hành chính. Plugin sẽ tự động chuẩn hóa nội bộ thành trường `POP_STAT`.
3. **Công viên/vườn hoa**: Chọn layer polygon công viên, vườn hoa hiện hữu làm vùng mẫu trích xuất SAVI và bảo toàn mảng xanh.
4. **Dấu vết công trình xây dựng (Footprint)**: Chọn layer polygon công trình (Google Open Buildings) đại diện cho không gian xây dựng vật lý.

---

## 4. Cửa sổ Cài đặt

Nhấn nút **Cài đặt** (biểu tượng bánh răng ở góc trên bên phải) để mở cửa sổ cấu hình gồm 2 tab:

<div class="guide-figure" markdown>
![Cửa sổ Cài đặt của Plugin](images/cai_dat_nang_cao.png)
<div class="guide-caption"><strong>Hình 2.</strong> Giao diện cấu hình trong cửa sổ Cài đặt của Plugin</div>
</div>

### Tab 1: Sản phẩm trung gian
- Cho phép người dùng chỉ định đường dẫn lưu trữ các tệp trung gian nếu có nhu cầu lưu trữ tệp riêng để phục vụ nghiên cứu hoặc kiểm tra chất lượng (ví dụ: Sentinel-2 Stack, MNDWI, SAVI, Mặt nước, SAVI loại nước, Thực vật thô, Histogram, Dấu vết công trình phân bổ dân số, Bảng thống kê mảng xanh).
- Nếu để trống, các sản phẩm trung gian sẽ được tự động tạo dưới dạng tệp tạm thời trong thư mục tạm của hệ điều hành (`tempfile.gettempdir()`) và nạp vào phiên làm việc.

### Tab 2: Tùy chỉnh nâng cao
- Thiết lập các tham số kỹ thuật: Ngưỡng MNDWI (`0.0`), Hệ số SAVI $L$ (`0.5`), Phương thức và phương pháp xác định ngưỡng SAVI (`P10`), Diện tích mảng xanh tối thiểu ngoài công viên (`5000 m²`), Bán kính phục vụ tối đa $R_{\max}$ (`300 m`), Chỉ tiêu diện tích mảng xanh bình quân đầu người $C$ (`6.0 m²/người`), Sai số hội tụ (`10 m`).
- Nhấn **OK** để lưu hoặc **Đặt lại** để quay về giá trị mặc định ban đầu.

---

## 5. Thao tác tại tab Sản phẩm đầu ra

Chuyển sang thẻ **Sản phẩm đầu ra** để chỉ định đường dẫn lưu:

<div class="guide-figure" markdown>
![Tab Sản phẩm đầu ra của Plugin](images/san_pham_dau_ra.png)
<div class="guide-caption"><strong>Hình 3.</strong> Giao diện chỉ định nơi lưu 5 sản phẩm đầu ra chính</div>
</div>

### 5 sản phẩm đầu ra chính:
1. **Mảng xanh đô thị** (định dạng `.tif`).
2. **Vùng phục vụ mảng xanh (R\*)** (định dạng `.gpkg`, `.shp` hoặc `.geojson`).
3. **Không gian xây dựng trong vùng phục vụ** (định dạng `.gpkg`, `.shp` hoặc `.geojson`).
4. **Không gian xây dựng ngoài vùng phục vụ** (định dạng `.gpkg`, `.shp` hoặc `.geojson`).
5. **Thống kê theo đơn vị hành chính** (định dạng `.gpkg`, `.shp` hoặc `.geojson`).

### Bảng báo cáo thống kê bổ sung:
- **Bảng thống kê theo đơn vị hành chính** (tùy chọn định dạng `.xlsx` hoặc `.csv`).

!!! tip "Cơ chế tạo tệp kết quả tạm thời"
    Nếu người dùng **để trống đường dẫn**, Plugin sẽ tự động tạo các tệp kết quả tạm thời trong thư mục tạm của hệ điều hành (`tempfile.gettempdir()`) và nạp trực tiếp lên bản đồ QGIS. Khuyến nghị người dùng chỉ định đường dẫn lưu mới để tránh bị khóa tệp khi ghi đè các sản phẩm cũ đang mở.

---

## 6. Chuỗi thực thi 5 Module

Khi nhấn nút **Phân tích**, thuật toán chạy ngầm qua Worker Thread không gây đơ giao diện QGIS theo chuỗi 5 module:

```
[Module 1: Chuẩn bị dữ liệu] ➔ [Module 2: Phân tách mảng xanh] ➔ [Module 3: Vùng phục vụ R*] ➔ [Module 4: Không gian xây dựng] ➔ [Module 5: Thống kê Hành chính]
```

- **Module 1 (Tiền xử lý và chuẩn hóa dữ liệu):**
  - Kiểm tra tính hợp lệ hình học các lớp vector.
  - Chuẩn hóa trường dân số thống kê thành `POP_STAT`.
  - Tiếp nhận hệ tọa độ phẳng dự chiếu tham chiếu (mét) và tạo ảnh Sentinel-2 Stack 10 m.
- **Module 2 (Chỉ số phổ và phân tách mảng xanh):**
  - Tính MNDWI và phân tách mặt nước.
  - Tính SAVI và xác định ngưỡng thực vật (tự động từ mẫu công viên hoặc thủ công).
  - Lọc bỏ mảng xanh nhỏ ngoài công viên (< 5000 m²).
  - Phân tích liên thông 8-neighbor tạo các mảng xanh độc lập, gán `PATCH_ID` và tính diện tích $S_i$.
- **Module 3 (Mô hình hóa vùng phục vụ mảng xanh):**
  - Phân tách footprint theo đơn vị hành chính tạo `footprint-part` và phân bổ dân số không gian:
    $$P_{b,u} = P_u \times \frac{A_{b,u}}{\sum_j A_{j,u}}$$
    tạo lớp `POP_ALLOC`.
  - Xây dựng ma trận khoảng cách hình học Euclid (EDT) từ biên từng mảng xanh.
  - Xác định quy mô dân số phục vụ mục tiêu $P_{target,i} = S_i / C$.
  - Tìm kiếm bán kính phục vụ lớn nhất $R^*$ trong $[0, R_{\max}]$ bằng Binary Search và xử lý các trường hợp biên (`RMAX`, `NO_SOLUTION`).
- **Module 4 (Phân tích không gian xây dựng):**
  - Phân tích vị trí không gian của từng `footprint-part` đối với vùng phục vụ bán kính $R^*$.
  - Phân loại `footprint-part` nằm trong và ngoài vùng phục vụ.
  - Áp dụng thuật toán gán độc quyền (**Exclusive Assignment**): nếu một `footprint-part` giao cắt với nhiều vùng phục vụ, nó được gán độc quyền cho mảng xanh gần nhất. Mỗi `footprint-part` chỉ thuộc tối đa một `PATCH_ID`, loại bỏ hoàn toàn đếm lặp.
- **Module 5 (Tổng hợp và thống kê kết quả):**
  - Tổng hợp trực tiếp từ các lớp vector kết quả ra 10 trường chỉ số định lượng theo từng đơn vị hành chính.
  - Tự động thực thi hệ thống 8 phép kiểm tra tính toàn vẹn dữ liệu (Balance Checks).
  - Xuất các lớp kết quả và bảng số liệu báo cáo (Excel/CSV).

---

## 7. Hệ thống 8 phép kiểm tra tính toàn vẹn dữ liệu (Balance Checks)

Plugin tích hợp cơ chế tự động kiểm tra đối soát số liệu khắt khe trước khi kết thúc:

1. **POPULATION BALANCE**: Tổng dân số phân bổ trên các footprint-part bằng đúng tổng dân số thống kê ban đầu của đơn vị hành chính ($\sum T\_DanSo = \sum D\_DaPhucVu + \sum D\_ThieuXanh$).
2. **PATCH POPULATION BALANCE**: Dân số được phân bổ cho các mảng xanh khớp với tổng dân số được phục vụ theo phường ($\sum SERVED\_POP(PATCH) = \sum D\_DaPhucVu$).
3. **WARD BUILDING AREA BALANCE**: Tổng diện tích xây dựng phục vụ và diện tích xây dựng thiếu xanh bằng đúng tổng diện tích xây dựng của phường/xã ($\sum S\_XayDung = \sum S\_DaPhucVu + \sum S\_ThieuXanh$).
4. **INPUT FOOTPRINT AREA BALANCE**: Tổng diện tích các footprint-part sau khi cắt bằng đúng diện tích footprint công trình đầu vào ($\sum AREA(input) = \sum AREA(served) + \sum AREA(outside)$).
5. **FOOTPRINT COUNT BALANCE**: Xác nhận toàn bộ footprint-part đầu vào của bước phân tích cuối được phân loại đầy đủ thành served hoặc outside ($N_{\text{input}} = N_{\text{served}} + N_{\text{outside}}$), không bị mất hoặc tạo thêm đối tượng ngoài quy trình.
6. **PATCH BUILDING AREA BALANCE**: Cân bằng diện tích xây dựng được gán cho các mảng xanh độc lập ($\sum SERVED\_BUILDING\_AREA(PATCH) = \sum AREA(cons\_in\_buffer)$).
7. **EXCLUSIVE PATCH ASSIGNMENT**: Xác nhận mỗi footprint-part chỉ được gán độc quyền cho tối đa một mảng xanh duy nhất, SERVED $\cap$ OUTSIDE = $\emptyset$.
8. **GREEN AREA BALANCE**: Tổng diện tích mảng xanh được phân bổ cho các đơn vị hành chính bằng đúng tổng diện tích mảng xanh toàn khu vực nghiên cứu.

---

## 8. Theo dõi tiến trình và nhật ký

Trong quá trình thực thi, tab **Nhật ký tiến trình** cập nhật liên tục với 5 nhóm ký hiệu chuẩn:

- `ℹ` **Khởi động / Thông tin:** Thông báo trạng thái cấu hình và thông tin kỹ thuật.
- `→` **Tiến trình:** Thông báo bước xử lý đang diễn ra.
- `✓` **Hoàn thành:** Thông báo bước xử lý hoàn tất thành công kèm số liệu định lượng.
- `⚠` **Cảnh báo:** Nhắc nhở các vấn đề cần lưu ý nhưng Plugin vẫn tiếp tục xử lý bình thường.
- `✗` **Lỗi:** Thông báo lỗi khiến tiến trình không thể tiếp tục hoặc dữ liệu không hợp lệ.

Người dùng có thể nhấn nút **Sao chép log** hoặc **Lưu log** ra tệp văn bản `.txt` để lưu lại toàn bộ tiến trình. Nếu muốn dừng khẩn cấp, nhấn nút **Hủy** để ngắt tiến trình an toàn.

---

## 9. Hoàn tất và hiển thị kết quả

Khi hoàn thành (100%):
- Hộp thoại *Thành công* hiển thị kèm bảng tổng kết thời gian thực thi của từng module.
- 5 sản phẩm đầu ra chính được tự động nạp lên danh sách lớp (Layers Panel) của QGIS.
- Các lớp được tự động áp dụng kiểu hiển thị (QML Style) chuyên nghiệp: mảng xanh màu xanh lá đậm, vùng phục vụ màu xanh rừng trong suốt, không gian xây dựng thiếu xanh màu đỏ tươi.

<div class="guide-figure" markdown>
![Các lớp kết quả được nạp tự động lên giao diện QGIS kèm kiểu dáng trực quan](images/ket_qua_qgis.png)
<div class="guide-caption"><strong>Hình 4.</strong> Các lớp kết quả được nạp tự động lên giao diện QGIS kèm kiểu dáng trực quan</div>
</div>

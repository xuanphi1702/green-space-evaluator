# Lỗi thường gặp và cách xử lý

Trang này tổng hợp các tình huống lỗi thường gặp trong quá trình chuẩn bị dữ liệu, cài đặt tham số và thực thi phân tích trên Plugin **Green Space Evaluator**, cùng nguyên nhân kỹ thuật và các bước xử lý tương ứng dựa trên mã nguồn thực tế của Plugin.

---

## 1. Lỗi không đọc/không nhận được trường dân số thống kê

### Hiện tượng
Hộp thoại cảnh báo xuất hiện thông báo lỗi hoặc trong **Nhật ký tiến trình** hiển thị:
```text
✗ Không tìm thấy trường dân số thống kê trong lớp ranh giới hành chính!
```
hoặc:
```text
✗ Dữ liệu dân số thống kê của một số đơn vị hành chính không hợp lệ (nhỏ hơn hoặc bằng 0, hoặc giá trị rỗng)!
```

### Nguyên nhân
- Người dùng chưa chọn trường thuộc tính dân số trong danh sách thả xuống trên giao diện.
- Trường được chọn chứa kiểu dữ liệu không phải dạng số (ví dụ: chuỗi văn bản Text chứa chữ cái, ký hiệu đặc biệt).
- Một số polygon đơn vị hành chính có giá trị dân số bị khuyết (NULL/NaN) hoặc giá trị $\le 0$.

### Cách xử lý
1. Mở bảng thuộc tính của lớp ranh giới hành chính trong QGIS, kiểm tra trường chứa số liệu dân số.
2. Đảm bảo trường dữ liệu có kiểu số (Integer, Real, Double) và tất cả các đơn vị hành chính đều có số dân hợp lệ (> 0).
3. Trên giao diện Plugin (tab *Dữ liệu đầu vào*), chọn chính xác tên trường dân số thống kê trong danh sách thả xuống.

---

## 2. Thiếu thư viện Python phụ thuộc

Plugin tự động kiểm tra sự tồn tại của các thư viện bắt buộc (`osgeo/gdal`, `numpy`, `scipy`, `matplotlib`) và tùy chọn (`pandas`, `openpyxl`) trước khi bắt đầu thực thi.

### Hiện tượng
Hộp thoại cảnh báo xuất hiện hoặc trong **Nhật ký tiến trình** hiển thị:
```text
✗ THIẾU THƯ VIỆN PYTHON BẮT BUỘC: [tên thư viện]!
```

### Cách xử lý
1. Đóng hoàn toàn phần mềm QGIS.
2. Mở công cụ **OSGeo4W Shell** trên Windows (tìm trong Start Menu với từ khóa *OSGeo4W Shell*).
3. Chạy lệnh cài đặt thư viện vào môi trường Python của QGIS:
   ```bash
   pip install numpy scipy matplotlib openpyxl pandas
   ```
4. Khởi động lại QGIS và mở lại Plugin để tiếp tục phân tích.

---

## 3. Thiếu kênh ảnh Sentinel-2 bắt buộc

Quy trình bóc tách mảng xanh yêu cầu tối thiểu **4 kênh ảnh bắt buộc**: **B03, B04, B08, B11**. Các kênh **B02** (tạo ảnh màu tự nhiên) và **SCL** (lọc mây và bóng mây) là tùy chọn bổ sung.

### Hiện tượng
Plugin dừng lại ở Module 1 và thông báo:
```text
✗ Thiếu các kênh Sentinel-2 bắt buộc: [B03, B04, B08, B11] (cần tối thiểu B03, B04, B08, B11 để tính MNDWI và SAVI)!
```

### Cách xử lý
- Đảm bảo chọn đủ tối thiểu 4 tệp kênh phổ của cùng một cảnh chụp Sentinel-2 Level-2A.
- Kiểm tra tên tệp hoặc tên layer trên QGIS có chứa đúng ký hiệu nhận diện kênh: `B03` (Green), `B04` (Red), `B08` (NIR), `B11` (SWIR).
- Có thể đổi tên layer trong bảng điều khiển lớp của QGIS để Plugin nhận diện chính xác kênh ảnh.

---

## 4. Lỗi không nạp được các lớp vector đầu vào

### 4.1. Lớp ranh giới hành chính
- **Hiện tượng**: `✗ Không thể nạp Vector ranh giới hành chính và dân số thống kê!`
- **Nguyên nhân**: Tệp bị hỏng, đường dẫn chứa ký tự đặc biệt hoặc lớp đang bị khóa bởi chương trình khác. Lớp chưa được chuyển về hệ tọa độ phẳng dự chiếu (Projected CRS, đơn vị mét).
- **Cách xử lý**: Đảm bảo lớp mở được trong QGIS, có hệ tọa độ phẳng phù hợp (ví dụ: VN-2000 kinh tuyến trục địa phương hoặc UTM) và cấu trúc hình học hợp lệ. Có thể dùng công cụ *Fix Geometries* trong QGIS để sửa lỗi polygon tự cắt.

### 4.2. Lớp công viên/vườn hoa
- **Hiện tượng**: `✗ Không thể nạp Vector polygon công viên/vườn hoa!`
- **Cách xử lý**: Đảm bảo lớp công viên có định dạng polygon hợp lệ và nằm trong phạm vi khu vực nghiên cứu.

### 4.3. Lớp dấu vết công trình xây dựng (Footprint)
- **Hiện tượng**: `✗ Không thể nạp Vector dấu vết công trình xây dựng!`
- **Cách xử lý**: Đảm bảo lớp footprint (Google Open Buildings / OSM) có hình học polygon hợp lệ và bao phủ không gian xây dựng trong khu vực nghiên cứu.

---

## 5. Không trích xuất được mẫu SAVI từ công viên

Khi cấu hình `Phương thức xác định ngưỡng SAVI` là `Tự động xác định`, Plugin sẽ lấy mẫu các ô pixel SAVI trên đất liền bên trong các polygon công viên để xác định ngưỡng phân tách thực vật.

### Hiện tượng
Nhật ký tiến trình báo lỗi:
```text
✗ Không trích xuất được pixel mẫu SAVI nào từ lớp Công viên/vườn hoa!
```

### Nguyên nhân
- Các polygon công viên nằm hoàn toàn ngoài phạm vi ảnh Sentinel-2 hoặc ngoài ranh giới nghiên cứu.
- Toàn bộ vùng công viên bị nhận diện nhầm là mặt nước (do MNDWI quá cao) hoặc bị che phủ bởi mây/bóng mây (kênh SCL gán NoData).

### Cách xử lý
- **Cách 1**: Kiểm tra lại vị trí các polygon công viên mẫu, đảm bảo nằm bên trong phạm vi cảnh ảnh Sentinel-2 và khu vực nghiên cứu.
- **Cách 2**: Chuyển sang chế độ **Nhập thủ công** trong cửa sổ **Cài đặt** (Tab *Tùy chỉnh nâng cao*), nhập một giá trị ngưỡng SAVI cố định (ví dụ: `0.20` hoặc `0.25`).

---

## 6. Các trường hợp biên khi xác định bán kính phục vụ R*

### 6.1. Không tìm thấy mảng xanh nào đạt ngưỡng diện tích
- **Hiện tượng**: `✗ Không tìm thấy mảng xanh nào đạt tiêu chuẩn diện tích tối thiểu!`
- **Nguyên nhân**: Ngưỡng SAVI bị đặt quá cao hoặc tham số *Diện tích mảng xanh tối thiểu* (`5000 m²`) quá lớn so với quy mô cây xanh thực tế trong khu vực.
- **Cách xử lý**: Mở cửa sổ Cài đặt, giảm diện tích mảng xanh tối thiểu hoặc giảm ngưỡng SAVI.

### 6.2. Trường hợp NO_SOLUTION ($R^* = 0\text{ m}$)
- **Hiện tượng**: Trong bảng thống kê mảng xanh hoặc nhật ký tiến trình xuất hiện một số mảng xanh có trạng thái `NO_SOLUTION` và bán kính $R^* = 0\text{ m}$.
- **Bản chất khoa học**: Đây **không phải là lỗi phần mềm**. Tình trạng này xảy ra khi mảng xanh có quy mô diện tích quá nhỏ ($S_i$ nhỏ) dẫn đến quy mô dân số phục vụ mục tiêu thấp ($P_{target,i} = S_i / C$), trong khi mật độ xây dựng và dân số phân bổ ngay sát bên mảng xanh ($R = 0\text{ m}$) đã lớn hơn $P_{target,i}$. Theo định nghĩa của mô hình, mảng xanh này không tìm thấy bán kính phục vụ thỏa điều kiện trong khoảng $[0, R_{\max}]$.
- **Cách xử lý**:
  - Không tùy tiện hạ chỉ tiêu $C$ chỉ để tạo ra bán kính phục vụ.
  - Đây là bằng chứng khoa học cho thấy mảng xanh quy mô quá nhỏ không đủ khả năng phục vụ dân số xung quanh theo chỉ tiêu quy chuẩn đề ra.

---

## 7. Lỗi khóa tệp (File Lock) khi lưu kết quả

### Hiện tượng
Plugin báo lỗi khi ghi tệp kết quả ra đĩa (đặc biệt là tệp Excel `.xlsx` hoặc GeoPackage `.gpkg`):
```text
✗ Lỗi khi lưu sản phẩm đầu ra: Permission denied / File is locked!
```

### Nguyên nhân
- Tệp kết quả cũ đang được mở bởi Microsoft Excel, QGIS hoặc một ứng dụng khác, khiến hệ điều hành khóa quyền ghi đè tệp.

### Cách xử lý
1. Đóng toàn bộ các tệp kết quả đang mở trong Excel hoặc các chương trình đọc ngoài.
2. Nếu lớp kết quả đang được nạp trong QGIS, nhấn chuột phải vào lớp và chọn *Remove Layer* trước khi chạy lại.
3. Chọn một tên tệp mới hoặc lưu sang một thư mục mới.
4. Có thể để trống đường dẫn lưu để Plugin tạo các lớp kết quả tạm thời (Temporary Layers) trong bộ nhớ QGIS.

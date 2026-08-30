# Lỗi thường gặp

Trang này tổng hợp các lỗi thường gặp trong quá trình cài đặt, chuẩn bị dữ liệu và thực thi phân tích trên Plugin **Green Space Evaluator**, cùng nguyên nhân kỹ thuật và các bước xử lý tương ứng dựa trên mã nguồn thực tế của Plugin.

---

## 1. Thiếu thư viện Python phụ thuộc

Plugin tự động kiểm tra sự tồn tại của các thư viện phụ thuộc bắt buộc (`numpy`, `osgeo/gdal`, `scipy`, `matplotlib`) và tùy chọn (`pandas`, `openpyxl`) trước khi bắt đầu thực thi thuật toán.

### Hiện tượng
Hộp thoại cảnh báo xuất hiện thông báo thiếu thư viện hoặc trong **Nhật ký tiến trình** hiển thị thông báo:
```text
❌ THIẾU THƯ VIỆN PYTHON BẮT BUỘC: [tên thư viện]!
```

### Kiểm tra
- Mở menu **Plugins** → **Python Console** trong QGIS để kiểm tra xem môi trường Python hiện tại đã cài đặt thư viện chưa.

### Cách xử lý
1. Đóng hoàn toàn phần mềm QGIS.
2. Mở công cụ **OSGeo4W Shell** trên Windows (tìm trong Start Menu với từ khóa *OSGeo4W Shell*).
3. Chạy lệnh cài đặt:
   ```bash
   pip install [tên thư viện còn thiếu]
   ```
4. Khởi động lại QGIS và mở lại Plugin để tiếp tục sử dụng.

---

## 2. Thiếu kênh ảnh Sentinel-2 bắt buộc

Quy trình tính toán chỉ số phổ MNDWI và SAVI yêu cầu tối thiểu **4 kênh ảnh bắt buộc**: **B03, B04, B08, B11**. Các kênh **B02** (tạo ảnh màu thực) và **SCL** (lọc mây và bóng mây tự động) là tùy chọn bổ sung.

### Hiện tượng
Plugin dừng ở Module 1 và báo lỗi:
```text
❌ Thiếu các kênh Sentinel-2 bắt buộc: [B03, B04, B08, B11] (cần tối thiểu B03, B04, B08, B11 để tính MNDWI và SAVI)!
```

### Kiểm tra
- Kiểm tra danh sách layer hoặc tệp ảnh Sentinel-2 đã chọn tại trường *1. Các kênh ảnh Sentinel-2*.
- Kiểm tra tên tệp hoặc tên layer có chứa đúng ký hiệu nhận diện kênh (ví dụ: `B03`, `B04`, `B08`, `B11`, `B02`, `SCL`).

### Cách xử lý
- Đảm bảo chọn đủ tối thiểu 4 tệp kênh phổ của cùng một cảnh chụp Sentinel-2 (Level-2A).
- Đảm bảo tên layer/tệp chứa đúng ký hiệu nhận diện kênh mà Plugin yêu cầu. Không nên thay đổi tên tệp gốc nếu không cần thiết; có thể đổi tên layer trong QGIS để Plugin nhận diện đúng kênh.s

---

## 3. Không nạp được dữ liệu đầu vào

### 3.1. Lớp khu vực nghiên cứu (MASK_LAYER)
- **Hiện tượng**: `❌ Không thể nạp Vector ranh giới khu vực nghiên cứu (MASK_LAYER)!`
- **Kiểm tra**: Kiểm tra đường dẫn/tệp nguồn của lớp và khả năng đọc lớp trong QGIS; đồng thời kiểm tra hình học của polygon có hợp lệ hay không.
- **Cách xử lý**: Nếu tệp hoặc đường dẫn có vấn đề, chọn lại đúng lớp dữ liệu. Nếu hình học không hợp lệ, có thể sử dụng công cụ **Fix Geometries** trong QGIS. Đồng thời đảm bảo lớp khu vực nghiên cứu đã được chuẩn hóa theo yêu cầu dữ liệu của Plugin, sử dụng Projected CRS với đơn vị mét.

### 3.2. Raster dân số WorldPop (POP_RASTER)
- **Hiện tượng**: `❌ Không thể nạp Raster dân số WorldPop (POP_RASTER)!`
- **Kiểm tra**: Tệp raster WorldPop có mở được trực tiếp trong QGIS hay không; phạm vi raster có bao trùm toàn bộ khu vực nghiên cứu hay không.
- **Cách xử lý**: Tải lại tệp GeoTIFF dân số WorldPop cho khu vực nghiên cứu hoặc kiểm tra lại đường dẫn tệp.

### 3.3. Lớp công viên/vườn hoa (VEG_VECTOR)
- **Hiện tượng**: `❌ Không thể nạp Vector polygon công viên/vườn hoa (VEG_VECTOR)!`
- **Kiểm tra**: Lớp vector công viên có định dạng polygon hợp lệ và nằm trong phạm vi khu vực nghiên cứu hay không.
- **Cách xử lý**: Kiểm tra bảng thuộc tính và không gian của lớp công viên mẫu trước khi nạp vào Plugin.

### 3.4. Lớp dấu vết công trình xây dựng (RES_VECTOR)
- **Hiện tượng**: `❌ Không thể nạp Vector dấu vết công trình xây dựng (RES_VECTOR)!`
- **Kiểm tra**: Lớp vector công trình (Open Buildings / OSM) có dữ liệu hình học hợp lệ hay không.
- **Cách xử lý**: Đảm bảo lớp công trình bao phủ phạm vi nghiên cứu và chứa các đa giác chân công trình xây dựng.

---

## 4. Không lấy được mẫu SAVI từ công viên

Khi cấu hình `SAVI_THRESHOLD = -999` (chế độ tự động), Plugin sẽ rasterize lớp công viên mẫu để trích xuất phân bố giá trị SAVI đất liền nhằm xác định ngưỡng bóc tách thực vật.

### Hiện tượng
Nhật ký tiến trình báo lỗi:
```text
❌ Không trích xuất được pixel mẫu SAVI nào từ lớp Công viên mẫu (VEG_VECTOR)!
```
hoặc:
```text
❌ Không thể mở raster mẫu công viên sau khi rasterize!
```

### Kiểm tra
- Các polygon công viên có nằm hoàn toàn ngoài phạm vi ảnh Sentinel-2 hoặc ranh giới nghiên cứu không.
- Toàn bộ vùng công viên có bị nhận diện nhầm là mặt nước (do MNDWI quá cao) hoặc bị gán NoData do mây/bóng mây hay không.

### Cách xử lý
- **Cách 1**: Kiểm tra và cập nhật lại lớp vector công viên mẫu đảm bảo nằm bên trong khu vực nghiên cứu và có phạm vi mẫu phù hợp để trích xuất giá trị SAVI..
- **Cách 2**: Chuyển sang **Ngưỡng SAVI thủ công** bằng cách mở cửa sổ **Cài đặt** (Tab *Tùy chỉnh nâng cao*), nhập giá trị ngưỡng cố định (ví dụ: `0.20` hoặc `0.25`) thay vì `-999`.

---

## 5. Không tạo được vùng phục vụ

### 5.1. Không tìm thấy pixel mảng xanh nào
- **Hiện tượng**: `❌ Không tìm thấy pixel mảng xanh nào để tạo vùng phục vụ!`
- **Kiểm tra**: Kiểm tra xem ngưỡng SAVI có bị đặt quá cao hoặc giá trị diện tích lọc tối thiểu (`MIN_GREEN_AREA`) có quá lớn so với hiện trạng cây xanh trong khu vực hay không.
- **Cách xử lý**: Giảm ngưỡng SAVI hoặc giảm diện tích lọc mảng xanh tối thiểu trong phần Cài đặt.

### 5.2. Quá tải ngay tại nguồn 
- **Hiện tượng**:
  ```text
  ❌ Trường hợp biên: C(Rmin) < Cmin (C(0) < Cmin)
  -> KHÔNG TỒN TẠI BÁN KÍNH PHỤC VỤ THỎA MÃN ĐIỀU KIỆN!
  ```
- **Kiểm tra**: Sức tải mảng xanh tại bán kính $R = 0\text{ m}$ ($C(0) = S_{UGS} / P(0)$) đã nhỏ hơn chỉ tiêu sức tải mục tiêu ($C_{min}$).  Điều này cho biết sức tải tại bán kính 0 m chưa đạt ngưỡng mục tiêu, do tương quan giữa diện tích mảng xanh và dân số tại phạm vi này chưa đáp ứng điều kiện đặt ra.
- **Cách xử lý**:
  - Chỉ thay đổi $C_{min}$ khi có cơ sở phù hợp với mục tiêu phân tích; không nên giảm ngưỡng chỉ để Plugin tạo được một bán kính phục vụ.
  - Kiểm tra lại lớp dân số WorldPop hoặc ngưỡng trích xuất mảng xanh để đảm bảo không bị thiếu hụt diện tích cây xanh thực tế.

---

## 6. Lỗi sản phẩm đầu ra

### Hiện tượng
Plugin báo lỗi khi ghi file raster/vector kết quả hoặc khi xuất báo cáo thống kê Excel/CSV.

### Kiểm tra
- Tệp kết quả cũ có đang được mở bằng phần mềm khác (ví dụ: mở file Excel `.xlsx` hoặc tệp GeoPackage `.gpkg` trong một phần mềm khác) làm tệp bị khóa ghi đè (file lock).
- Thư mục lưu trữ đầu ra có quyền ghi (write permission) hoặc ổ đĩa bị đầy dung lượng hay không.

### Cách xử lý
- Đóng tất cả các tệp kết quả đang mở trong Excel hoặc các phần mềm ngoài trước khi bấm **Phân tích**.
- Chọn đường dẫn lưu ở một thư mục mới hoặc để trống đường dẫn để Plugin tự động tạo các lớp kết quả tạm thời (Temporary Layers) trong QGIS.

---

## 7. Plugin dừng trong quá trình xử lý

Nếu tiến trình phân tích bị gián đoạn giữa chừng mà không rõ nguyên nhân:

1. Chuyển sang thẻ **Nhật ký tiến trình** trên giao diện chính của Plugin.
2. Rà soát các dòng nhật ký cuối cùng để xác định module mà quá trình phân tích dừng lại.
3. Đọc thông điệp cảnh báo có biểu tượng ❌ hoặc ⚠️ để biết chính xác tham số hoặc dữ liệu cần hiệu chỉnh.
4. Sử dụng nút **Sao chép log** để lưu lại toàn bộ tiến trình phân tích phục vụ tra cứu.

---

## Khi cần báo lỗi

Nếu bạn gặp phải lỗi chưa được đề cập ở trên hoặc nghi ngờ lỗi từ mã nguồn Plugin, vui lòng chuẩn bị các thông tin sau để gửi báo cáo hỗ trợ:

- **Phiên bản QGIS**: Phiên bản QGIS đang sử dụng (ví dụ: QGIS 3.28 LTR, 3.34 LTR hoặc 3.44).
- **Thông điệp lỗi chi tiết**: Nội dung lỗi hiển thị trong khung *Nhật ký tiến trình* hoặc Python Error Dialog.
- **Tệp nhật ký xử lý**: Nhấn nút **Lưu log** trong Plugin để xuất file `.txt`.
- **Cấu hình tham số**: Giá trị các tham số đã thiết lập trong cửa sổ Cài đặt ($R_{max}$, $C_{min}$, ngưỡng SAVI,...).
- **Mô tả dữ liệu đầu vào**: Hệ tọa độ, định dạng và phạm vi của 5 nhóm dữ liệu đầu vào đang sử dụng.
- **Ảnh chụp màn hình**: Ảnh chụp giao diện và vị trí phát sinh lỗi (nếu có).

---

## Liên kết liên quan

- [Cài đặt Plugin](cai-dat.md): Kiểm tra môi trường và cài đặt thư viện phụ thuộc.
- [Dữ liệu đầu vào](du-lieu-dau-vao.md): Hướng dẫn chuẩn hóa dữ liệu trước khi chạy.
- [Cài đặt tham số](tham-so.md): Giải thích chi tiết các ngưỡng kỹ thuật và tham số mô hình.
- [Chạy phân tích](chay-phan-tich.md): Quy trình thực thi phân tích từng bước.

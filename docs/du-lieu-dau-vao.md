# Dữ liệu đầu vào

Trang này hướng dẫn chi tiết về **danh mục dữ liệu đầu vào cần thiết** và **cách chuẩn bị dữ liệu trước khi đưa vào Plugin Green Space Evaluator**.

---

## 1. Plugin cần những dữ liệu nào?

Để thực hiện toàn diện quy trình phân tích, Plugin yêu cầu **5 nhóm dữ liệu đầu vào**:

| Nhóm dữ liệu | Dữ liệu sử dụng | Vai trò | Yêu cầu chuẩn bị |
|---|---|---|---|
| **1. Khu vực nghiên cứu** | Vector Polygon ranh giới khu vực nghiên cứu | Khung tham chiếu không gian (CRS, phạm vi) cho toàn bộ quy trình xử lý | Hệ tọa độ phẳng (Projected CRS, đơn vị mét), hình học hợp lệ, bao trùm phạm vi nghiên cứu |
| **2. Sentinel-2** | Raster các kênh phổ ảnh Sentinel-2 (L2A) | Tính toán chỉ số phổ MNDWI, SAVI để bóc tách mặt nước và mảng xanh | Đầy đủ các kênh bắt buộc (**B03, B04, B08, B11**); tùy chọn **B02, SCL**; định dạng GeoTIFF |
| **3. WorldPop** | Raster mật độ dân số WorldPop | Ước tính số lượng và phân bố dân số trong khu vực và vùng phục vụ | Định dạng GeoTIFF (số người/pixel), bao phủ trọn vẹn khu vực nghiên cứu |
| **4. Công viên tham chiếu** | Vector Polygon công viên, vườn hoa mẫu | Cung cấp vùng mẫu để trích xuất phân bố SAVI xác định ngưỡng và bảo toàn mảng xanh công viên | Vector polygon ranh giới các công viên thực tế trong khu vực |
| **5. Google Open Buildings** | Vector Polygon dấu vết chân công trình xây dựng | Đại diện cho không gian xây dựng; hỗ trợ phân tích dân số ước tính theo không gian xây dựng và phân loại công trình theo vùng phục vụ | Vector polygon dấu vết công trình (Open Buildings, OSM hoặc tương đương) |

---

## 2. Dữ liệu phải được chuẩn bị như thế nào trước khi đưa vào Plugin?

### 2.1. Khu vực nghiên cứu (Ranh giới nghiên cứu)

!!! warning "Lớp khu vực nghiên cứu cần được chuẩn hóa trước khi đưa vào Plugin"
    Lớp ranh giới khu vực nghiên cứu đóng vai trò là **khung tham chiếu không gian** (spatial reference) cho toàn bộ quy trình: Plugin sử dụng trực tiếp hệ tọa độ và phạm vi không gian của lớp này để cắt (clip), tái chiếu (reproject) và chuẩn hóa độ phân giải lưới (10m) cho các dữ liệu raster khác (Sentinel-2, WorldPop).

Các yêu cầu kỹ thuật cần đảm bảo:

- **Hệ tọa độ (CRS)**: Lớp khu vực nghiên cứu **phải sử dụng hệ tọa độ phẳng (Projected CRS)** có đơn vị đo lường bằng mét (ví dụ: hệ tọa độ VN-2000 kinh tuyến trục địa phương hoặc UTM tương ứng). Việc này giúp các phép tính khoảng cách Euclid (m) và diện tích (m², ha) diễn ra chính xác. Plugin **không tự động xác định hay áp đặt một hệ tọa độ tổng quát** cho khu vực nghiên cứu bất kỳ.
- **Phạm vi không gian (Extent)**: Đa giác ranh giới cần thể hiện đúng và đủ phạm vi địa lý của khu vực nghiên cứu hoặc đơn vị hành chính cần đánh giá.
- **Tính hợp lệ của hình học (Geometry)**: Lớp vector cần có cấu trúc hình học hợp lệ, không bị lỗi tự cắt (self-intersection) hoặc hở đỉnh.
- **Khung tham chiếu cho xử lý tiếp theo**: Dữ liệu lớp này được dùng làm chuẩn không gian cho mọi bước xử lý phía sau.

---

### 2.2. Ảnh viễn thám Sentinel-2

Plugin xử lý ảnh phản xạ bề mặt Sentinel-2 để nhận diện mặt nước và thảm thực vật mảng xanh:

- **Loại sản phẩm**: Sentinel-2 **Level-2A (L2A)** (sản phẩm đã hiệu chỉnh khí quyển Bottom of Atmosphere - BOA).
- **Các kênh phổ bắt buộc**:
    - **B03** (Green - Xanh lục) và **B11** (SWIR - Hồng ngoại sóng ngắn): Dùng để tính toán chỉ số khác biệt nước cải tiến **MNDWI**:

        $$
        \text{MNDWI} = \frac{\text{B03} - \text{B11}}{\text{B03} + \text{B11}}
        $$

    - **B04** (Red - Đỏ) và **B08** (NIR - Cận hồng ngoại): Dùng để tính toán chỉ số thực vật điều chỉnh theo đất **SAVI**:

        $$
        \text{SAVI} = \frac{\text{B08} - \text{B04}}{\text{B08} + \text{B04} + L} \times (1 + L)
        $$

        *(với hệ số hiệu chỉnh nền đất mặc định $L = 0{,}5$)*

- **Các kênh phổ tùy chọn**:
    - **B02** (Blue - Xanh lam): Hỗ trợ ghép kênh tạo tổ hợp ảnh màu thực True Color (RGB) trong tệp ảnh stack.
    - **SCL** (Scene Classification Layer): Kênh phân loại cảnh quan dùng để tạo mặt nạ lọc tự động mây, bóng mây trước khi tính chỉ số phổ.

**Checklist chuẩn bị ảnh Sentinel-2:**

- [x] Có đầy đủ các kênh phổ bắt buộc (**B03, B04, B08, B11**); bổ sung **B02** và **SCL** nếu có nhu cầu
- [x] Các tệp kênh ảnh thuộc cùng một cảnh chụp (scene) hoặc cùng thời điểm thu nhận thích hợp
- [x] Tệp raster định dạng GeoTIFF đọc được trực tiếp trong QGIS
- [x] Phạm vi ảnh bao trùm toàn bộ khu vực nghiên cứu

---

### 2.3. Dữ liệu dân số WorldPop

Dữ liệu dân số được sử dụng để định lượng số dân được phục vụ và số dân thiếu hụt mảng xanh:

- **Loại dữ liệu**: Raster mật độ dân số ước tính không gian (WorldPop Individual countries / UN-adjusted Population Count, đơn vị: số người/pixel).
- **Quy trình chuẩn hóa tự động**: Trong quá trình thực thi, Plugin sẽ tự động cắt raster theo ranh giới nghiên cứu, chuyển đổi hệ tọa độ và tái mẫu (resample) về kích thước ô lưới 10m đồng bộ với ảnh Sentinel-2, đồng thời tính toán hệ số tỷ lệ để **bảo toàn tổng dân số** của vùng nghiên cứu.
- **Phân biệt bản chất dữ liệu**:

!!! note "Phân biệt dân số ước tính và dân số kiểm kê thực tế"
    Dữ liệu dân số từ WorldPop là **dân số ước tính** theo mô hình phân bố không gian dựa trên ảnh vệ tinh và các biến phụ trợ, không phải là **dân số kiểm kê thực tế** từ số liệu điều tra hộ khẩu hoặc thống kê hành chính. Các chỉ tiêu tính toán trong Plugin thể hiện tương quan phân bố không gian của dân cư.

---

### 2.4. Dữ liệu dấu vết công trình (Google Open Buildings)

Dữ liệu công trình được sử dụng để xác định ranh giới không gian xây dựng:

- **Bản chất dữ liệu**: Google Open Buildings là tập dữ liệu nhận diện hình học **dấu vết chân công trình xây dựng (building footprints)** từ ảnh vệ tinh có độ phân giải cao.
- **Vai trò trong Plugin**: 
    1. Đại diện cho **không gian xây dựng** để trích xuất lớp raster dân số phân bố trong các khối nhà/công trình.
    2. Dùng để **phân loại các công trình xây dựng nằm trong vùng phục vụ (Intersects) và nằm ngoài vùng phục vụ (Disjoint)** tương ứng với bán kính phục vụ $R^*$.
    3. Hỗ trợ tính toán tổng diện tích không gian xây dựng và tỷ lệ dân cư thiếu mảng xanh theo từng đơn vị hành chính.
- **Nguồn dữ liệu phù hợp**: Google Open Buildings (vector polygon), OpenStreetMap (OSM building footprints) hoặc lớp dữ liệu hiện trạng công trình xây dựng tương đương.

!!! warning "Open Buildings không phải dữ liệu dân cư"
    Tập dữ liệu Google Open Buildings đại diện cho **dấu vết chân công trình xây dựng (building footprints)**. Đây **không phải là dữ liệu dân cư, không phải dữ liệu nhà ở dân sinh hay dữ liệu kiểm kê hộ tịch**, mà đóng vai trò đại diện cho **không gian xây dựng** trong mô hình phân tích.

---

### 2.5. Lớp công viên tham chiếu

- **Vai trò trong Plugin**: Lớp vector polygon công viên, vườn hoa, mảng xanh công cộng hiện hữu đảm nhận 2 vai trò kỹ thuật:
    1. **Tập mẫu trích xuất SAVI**: Khi chạy ở chế độ tự động (`SAVI_THRESHOLD = -999`), Plugin trích xuất phân bố giá trị SAVI bên trong các đa giác này để tính toán ngưỡng phân tách thực vật khách quan (theo P10, Mean - Std, Median...).
    2. **Bảo toàn mảng xanh công viên**: Các pixel thực vật được nhận diện nằm bên trong ranh giới công viên sẽ được giữ lại nguyên vẹn, không bị loại bỏ bởi bộ lọc diện tích tối thiểu ngoài công viên.
- **Phân biệt rõ ràng**:

!!! info "Phân biệt công viên tham chiếu và mảng xanh đô thị kết quả"
    Lớp công viên tham chiếu **không đồng nhất với lớp mảng xanh đô thị đầu ra**. Công viên tham chiếu chỉ đóng vai trò là tập mẫu và ranh bảo toàn; mảng xanh đô thị cuối cùng là kết quả bóc tách từ ảnh viễn thám trên toàn bộ vùng nghiên cứu (kết hợp cả mảng xanh trong công viên và các dải cây xanh ngoài công viên đã qua bước lọc diện tích tối thiểu).

---

## Checklist trước khi chạy Plugin

Trước khi nhấn nút **Phân tích**, hãy rà soát lại các mục sau:

- [ ] **Khu vực nghiên cứu đã được chuẩn hóa**: Sử dụng hệ tọa độ phẳng (Projected CRS, đơn vị mét) phù hợp, hình học polygon hợp lệ, bao trùm đầy đủ phạm vi phân tích.
- [ ] **Các lớp dữ liệu cần thiết đã đầy đủ**: Đã chọn đủ 5 nhóm dữ liệu trong tab *Dữ liệu đầu vào*.
- [ ] **Sentinel-2 có đủ các band cần thiết**: Tối thiểu gồm các kênh B03, B04, B08, B11; bổ sung B02 và SCL nếu có.
- [ ] **Raster dân số có thể đọc được**: Tệp raster WorldPop hợp lệ và bao phủ trọn vẹn khu vực nghiên cứu.
- [ ] **Lớp Open Buildings đã sẵn sàng**: Dữ liệu vector polygon công trình xây dựng hợp lệ, đại diện cho không gian xây dựng.
- [ ] **Lớp công viên tham chiếu đã sẵn sàng**: Dữ liệu vector polygon ranh giới công viên mẫu đã được nạp.
- [ ] **Không gian lưu kết quả không gây xung đột với sản phẩm cũ**: Đường dẫn lưu kết quả không trùng với sản phẩm đang tồn tại hoặc đang được chương trình khác sử dụng.

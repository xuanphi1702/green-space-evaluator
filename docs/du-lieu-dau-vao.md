# Dữ liệu đầu vào

Trang này hướng dẫn chi tiết về **danh mục 4 nhóm dữ liệu đầu vào cần thiết** và **cách chuẩn bị dữ liệu chuẩn xác trước khi đưa vào Plugin Green Space Evaluator**.

---

## 1. Danh mục 4 nhóm dữ liệu đầu vào

Để thực hiện toàn diện quy trình phân tích, Plugin yêu cầu **4 nhóm dữ liệu đầu vào**:

| STT | Nhóm dữ liệu | Dạng dữ liệu | Vai trò kỹ thuật | Yêu cầu chuẩn bị |
|---|---|---|---|---|
| **1** | **Ảnh Sentinel-2** | Raster đa kênh (`.tif`, `.jp2`) | Tính toán chỉ số phổ MNDWI, SAVI để bóc tách mặt nước và mảng xanh thực vật | Đầy đủ các kênh bắt buộc (**B03, B04, B08, B11**); tùy chọn **B02, SCL**; định dạng GeoTIFF hoặc JPEG2000 |
| **2** | **Ranh giới hành chính và dân số thống kê** | Vector Polygon | Khung tham chiếu không gian (CRS, phạm vi) và cung cấp số liệu dân số thống kê chính thức của từng đơn vị hành chính | Hệ tọa độ phẳng (Projected CRS, đơn vị mét), hình học hợp lệ; chứa trường thuộc tính dân số thống kê |
| **3** | **Công viên/vườn hoa** | Vector Polygon | Vùng mẫu trích xuất phân bố SAVI xác định ngưỡng thực vật và bảo toàn mảng xanh công viên | Vector polygon phạm vi công viên/vườn hoa trong khu vực nghiên cứu |
| **4** | **Dấu vết công trình xây dựng (Footprint)** | Vector Polygon | Đại diện cho không gian xây dựng vật lý; cơ sở phân bổ dân số thống kê theo không gian (`POP_ALLOC`) | Vector polygon dấu vết chân công trình xây dựng (Google Open Buildings, OSM hoặc tương đương) |

---

## 2. Chi tiết chuẩn bị từng nhóm dữ liệu

### 2.1. Ảnh viễn thám Sentinel-2

Plugin xử lý ảnh phản xạ bề mặt Sentinel-2 để nhận diện mặt nước và thảm thực vật mảng xanh:

- **Loại sản phẩm**: Sentinel-2 **Level-2A (L2A)** (sản phẩm đã hiệu chỉnh khí quyển Bottom of Atmosphere - BOA).
- **Các kênh phổ bắt buộc**:
    - **B03** (Green - Xanh lục, 10 m) và **B11** (SWIR - Hồng ngoại sóng ngắn, 20 m): Dùng để tính toán chỉ số khác biệt nước cải tiến **MNDWI**:

        $$
        \text{MNDWI} = \frac{\text{B03} - \text{B11}}{\text{B03} + \text{B11}}
        $$

    - **B04** (Red - Đỏ, 10 m) và **B08** (NIR - Cận hồng ngoại, 10 m): Dùng để tính toán chỉ số thực vật điều chỉnh theo đất **SAVI**:

        $$
        \text{SAVI} = \frac{\text{B08} - \text{B04}}{\text{B08} + \text{B04} + L} \times (1 + L)
        $$

        *(với hệ số hiệu chỉnh nền đất mặc định $L = 0.5$)*

- **Các kênh phổ tùy chọn**:
    - **B02** (Blue - Xanh lam, 10 m): Hỗ trợ ghép kênh tạo tổ hợp ảnh màu thực True Color (RGB) trong tệp ảnh Sentinel-2 Stack.
    - **SCL** (Scene Classification Layer, 20 m): Kênh phân loại cảnh quan dùng để tạo mặt nạ lọc tự động mây và bóng mây trước khi tính chỉ số phổ.

**Checklist chuẩn bị ảnh Sentinel-2:**
- [x] Có đầy đủ các kênh phổ bắt buộc (**B03, B04, B08, B11**); bổ sung **B02** và **SCL** nếu có nhu cầu.
- [x] Các tệp kênh ảnh thuộc cùng một cảnh chụp (scene) hoặc thời điểm thu nhận thích hợp, ít mây.
- [x] Tệp raster định dạng GeoTIFF đọc được trực tiếp trong QGIS.
- [x] Phạm vi ảnh bao trùm toàn bộ khu vực nghiên cứu.

---

### 2.2. Ranh giới hành chính và dân số thống kê

Lớp ranh giới hành chính đóng vai trò là **khung tham chiếu không gian** và nguồn cung cấp **số liệu dân số thống kê chính thức**:

- **Bản chất dữ liệu**: Lớp vector polygon phân chia ranh giới các đơn vị hành chính (ví dụ: phường, xã, thị trấn hoặc quận, huyện) trong phạm vi nghiên cứu.
- **Hệ tọa độ (CRS)**: **Bắt buộc sử dụng hệ tọa độ phẳng dự chiếu (Projected CRS)** có đơn vị đo lường bằng mét (ví dụ: VN-2000 kinh tuyến trục địa phương hoặc UTM tương ứng). Việc này đảm bảo tính toán khoảng cách Euclid (m) và diện tích (m², ha) diễn ra chính xác. Plugin lấy CRS của lớp này làm chuẩn không gian cho toàn bộ quy trình.
- **Dân số thống kê (`POP_STAT`)**:
    - Trên giao diện Plugin, người dùng chỉ định trường thuộc tính chứa số liệu dân số thống kê chính thức của từng đơn vị hành chính.
    - Plugin tự động đọc và chuẩn hóa dữ liệu sang trường chuẩn nội bộ `POP_STAT`.
    - Dữ liệu này là số liệu thống kê chính thức của chính quyền địa phương hoặc cơ quan thống kê, đảm bảo tính pháp lý và tin cậy cao.

---

### 2.3. Lớp công viên/vườn hoa

- **Bản chất dữ liệu**: Lớp vector polygon thể hiện phạm vi các công viên, vườn hoa hoặc mảng xanh công cộng hiện hữu.
- **Vai trò trong Plugin**: Đảm nhận 2 vai trò kỹ thuật quan trọng:
    1. **Vùng mẫu trích xuất SAVI**: Khi chạy ở chế độ tự động, Plugin trích xuất phân bố giá trị SAVI bên trong các đa giác công viên mẫu để tính toán ngưỡng phân tách thực vật khách quan (theo P10, Mean - Std, Median...).
    2. **Bảo toàn mảng xanh công viên**: Các pixel thực vật được nhận diện nằm bên trong ranh giới công viên sẽ được bảo toàn nguyên vẹn, không bị loại trừ bởi bộ lọc diện tích tối thiểu.

!!! info "Phân biệt công viên mẫu và kết quả mảng xanh đô thị"
    Lớp công viên/vườn hoa chỉ đóng vai trò là **vùng mẫu trích xuất ngưỡng và vùng bảo toàn**. Kết quả "Mảng xanh đô thị" đầu ra là toàn bộ thảm thực vật được bóc tách trên khắp khu vực nghiên cứu (bao gồm cả mảng xanh trong công viên và các mảng xanh độc lập ngoài công viên đạt quy mô diện tích tối thiểu).

---

### 2.4. Dấu vết công trình xây dựng (Footprint)

- **Bản chất dữ liệu**: Google Open Buildings hoặc OpenStreetMap (OSM) là tập dữ liệu nhận diện hình học **dấu vết chân công trình xây dựng (building footprints)** từ ảnh vệ tinh độ phân giải cao.
- **Vai trò đại diện không gian xây dựng**: Footprint đại diện cho **không gian xây dựng vật lý (physical built space)**, phản ánh quy mô và vị trí các khối kết cấu xây dựng trên bề mặt đô thị.
- **Cơ chế phân tách hình học (footprint-part)**:
    - Khi một polygon công trình xây dựng cắt qua ranh giới hành chính, phần hình học của nó được phân tách thành các phần tử hình học nhỏ hơn gọi là `footprint-part` thuộc về từng đơn vị hành chính.
- **Công thức phân bổ dân số không gian**:
    - Dân số thống kê chính thức của đơn vị hành chính ($P_u$) được phân bổ cho từng `footprint-part` ($b$) thuộc đơn vị hành chính $u$ theo tỷ lệ diện tích:

        $$
        P_{b,u} = P_u \times \frac{A_{b,u}}{\sum_j A_{j,u}}
        $$

    - Trong đó:
        - $P_u$: Dân số thống kê chính thức của đơn vị hành chính $u$ (`POP_STAT`).
        - $A_{b,u}$: Diện tích của `footprint-part` $b$ nằm trong đơn vị hành chính $u$.
        - $\sum_j A_{j,u}$: Tổng diện tích của tất cả các `footprint-part` nằm trong đơn vị hành chính $u$.
        - $P_{b,u}$: Dân số được phân bổ cho `footprint-part` $b$ (lưu trong trường `POP_ALLOC`).

!!! warning "Lưu ý phương pháp luận về POP_ALLOC và Footprint"
    1. **`POP_ALLOC`** là **dân số được phân bổ theo không gian phục vụ mô hình hóa**, KHÔNG phải là dân số quan sát trực tiếp, kiểm kê hộ tịch hay điều tra thực địa tại từng công trình.
    2. **Footprint** là **dấu vết công trình xây dựng**, KHÔNG đồng nhất hoàn toàn với nhà ở dân cư, số hộ gia đình hay nơi cư trú thực tế (vì có thể bao gồm nhà xưởng, công trình công cộng, thương mại, dịch vụ).

---

## 3. Checklist trước khi phân tích

Trước khi nhấn nút **Phân tích**, hãy kiểm tra lại các mục sau:

- [ ] **Hệ tọa độ chuẩn**: Lớp ranh giới hành chính sử dụng hệ tọa độ phẳng dự chiếu (Projected CRS, đơn vị mét) phù hợp với khu vực nghiên cứu.
- [ ] **Dữ liệu vector hợp lệ**: Các lớp vector không bị lỗi tự cắt hình học (Self-intersection); có thể chạy công cụ *Fix Geometries* trong QGIS nếu cần.
- [ ] **Đầy đủ 4 nhóm dữ liệu**: Đã chọn đủ 4 nhóm trên tab *Dữ liệu đầu vào*.
- [ ] **Kênh Sentinel-2 đầy đủ**: Tối thiểu gồm các kênh B03, B04, B08, B11.
- [ ] **Trường dân số hợp lệ**: Đã chọn đúng trường thuộc tính chứa số liệu dân số thống kê dạng số dương.

# Dữ liệu đầu vào

Trang này hướng dẫn chi tiết về **danh mục 4 nhóm dữ liệu đầu vào cần thiết** và **cách chuẩn bị dữ liệu chuẩn xác trước khi đưa vào Plugin Green Space Evaluator**.

---

## 1. Danh mục 4 nhóm dữ liệu đầu vào

Để thực hiện toàn diện quy trình phân tích, Plugin yêu cầu **4 nhóm dữ liệu chuẩn**:

| STT | Nhóm dữ liệu | Dạng dữ liệu | Vai trò kỹ thuật | Yêu cầu chuẩn bị |
|---|---|---|---|---|
| **1** | **Ảnh Sentinel-2** | Raster đa kênh (`.tif`, `.jp2`) | Tính toán chỉ số phổ MNDWI, SAVI để bóc tách mặt nước và mảng xanh thực vật | Đầy đủ các kênh bắt buộc (**B03, B04, B08, B11**); tùy chọn **B02, SCL**; định dạng GeoTIFF hoặc JPEG2000 |
| **2** | **Ranh giới hành chính và dân số thống kê** | Vector Polygon | Khung tham chiếu không gian (CRS, phạm vi) và cung cấp số liệu dân số thống kê chính thức của từng đơn vị hành chính | Hệ tọa độ phẳng dự chiếu (Projected CRS, đơn vị mét); chứa trường thuộc tính dân số thống kê chính thức |
| **3** | **Dấu vết công trình xây dựng (Building Footprints)** | Vector Polygon | Đại diện cho không gian xây dựng vật lý; cơ sở phân chia thành các footprint-part và phân bổ dân số thống kê theo không gian (`POP_ALLOC`) | Vector polygon dấu vết chân công trình xây dựng (ví dụ Google Open Buildings) |
| **4** | **Các tham số phân tích** | Cấu hình tham số | Các ngưỡng quang phổ, chỉ tiêu không gian và tham số tìm kiếm nghiệm số học | Thiết lập cấu hình tại cửa sổ Cài đặt |

---

## 2. Chi tiết chuẩn bị từng nhóm dữ liệu

### 2.1. Ảnh viễn thám Sentinel-2

Plugin xử lý ảnh phản xạ bề mặt Sentinel-2 để nhận diện mặt nước và thảm thực vật mảng xanh:

- **Loại sản phẩm**: Sentinel-2 **Level-2A (L2A)** (sản phẩm đã hiệu chỉnh khí quyển Bottom of Atmosphere - BOA).
- **Cơ chế nhận diện kênh**: Plugin tự động nhận diện các kênh dựa trên chuỗi văn bản nhận diện xuất hiện trong **tên layer trên QGIS** hoặc **đường dẫn/tên tệp nguồn** (ví dụ: `B02`, `B03`, `B04`, `B08`, `B11`, `SCL`).
- **Các kênh phổ bắt buộc**:
    - **B03** (Green - Xanh lục, 10 m) và **B11** (SWIR-1 - Hồng ngoại sóng ngắn, 20 m): Dùng để tính toán chỉ số khác biệt nước cải tiến **MNDWI**:

        $$
        \text{MNDWI} = \frac{\text{B03} - \text{B11}}{\text{B03} + \text{B11}}
        $$

    - **B04** (Red - Đỏ, 10 m) và **B08** (NIR - Cận hồng ngoại, 10 m): Dùng để tính toán chỉ số thực vật điều chỉnh theo đất **SAVI**:

        $$
        \text{SAVI} = \frac{\text{B08} - \text{B04}}{\text{B08} + \text{B04} + L} \times (1 + L)
        $$

        *(với hệ số hiệu chỉnh nền đất $L = 0.50$, giá trị cấu hình chuẩn production)*

- **Các kênh phổ tùy chọn**:
    - **B02** (Blue - Xanh lam, 10 m): Hỗ trợ ghép kênh tạo tổ hợp ảnh màu tự nhiên RGB trong tệp ảnh Sentinel-2 Stack.
    - **SCL** (Scene Classification Layer, 20 m): Kênh phân loại cảnh quan dùng để tạo mặt nạ lọc tự động mây và bóng mây trước khi tính chỉ số phổ.

**Checklist chuẩn bị ảnh Sentinel-2:**
- [x] Có đầy đủ các kênh phổ bắt buộc (**B03, B04, B08, B11**); bổ sung **B02** và **SCL** nếu có nhu cầu.
- [x] Các tệp kênh ảnh thuộc cùng một cảnh chụp (scene) hoặc thời điểm thu nhận thích hợp, ít mây.
- [x] Tệp raster định dạng GeoTIFF đọc được trực tiếp trong QGIS.
- [x] Phạm vi ảnh bao trùm toàn bộ khu vực nghiên cứu.

---

### 2.2. Ranh giới hành chính và dân số thống kê

Lớp ranh giới hành chính đóng vai trò là **khung tham chiếu không gian** và nguồn cung cấp **số liệu dân số thống kê chính thức**:

- **Bản chất dữ liệu**: Lớp vector polygon phân chia ranh giới các đơn vị hành chính (phường, xã, thị trấn hoặc quận, huyện) trong phạm vi nghiên cứu.
- **Yêu cầu Hệ tọa độ (CRS)**:
    - Lớp ranh giới hành chính và dữ liệu dân số công trình **bắt buộc sử dụng hệ tọa độ phẳng dự chiếu (Projected CRS, đơn vị mét)** như VN-2000 kinh tuyến trục địa phương hoặc UTM để tính diện tích m² và khoảng cách chính xác.
    - Plugin kiểm tra và đồng bộ CRS theo yêu cầu của từng giai đoạn xử lý; riêng trong Module 4, nếu CRS của `OUT_BUFFER` khác CRS của footprint-population, hình học `OUT_BUFFER` được chuyển đổi trong bộ nhớ về CRS phân tích trước khi overlay.
- **Dân số thống kê (`POP_STAT`)**:
    - Trên giao diện Plugin, người dùng chỉ định trường thuộc tính chứa số liệu dân số thống kê chính thức của từng đơn vị hành chính.
    - Plugin tự động đọc và chuẩn hóa dữ liệu sang trường chuẩn nội bộ `POP_STAT`.
    - Dữ liệu này là số liệu thống kê thuộc tính chính thức, không lấy từ raster dân số.

---

### 2.3. Dấu vết công trình xây dựng (Footprint)

- **Bản chất dữ liệu**: Lớp polygon nhận diện hình học **dấu vết chân công trình xây dựng (building footprints)** (ví dụ từ Google Open Buildings hoặc dữ liệu đo đạc thành lập bản đồ).
- **Vai trò đại diện không gian xây dựng**: Dấu vết công trình đại diện cho **không gian xây dựng vật lý (physical built space)**, phản ánh quy mô và vị trí các khối kết cấu xây dựng trên bề mặt đô thị.
- **Cơ chế phân chia thành footprint-part**:
    - Khi một polygon công trình xây dựng cắt qua ranh giới hành chính, phần hình học của nó được phân chia thành các phần tử hình học nhỏ hơn gọi là footprint-part thuộc về từng đơn vị hành chính.
- **Công thức phân bổ dân số không gian**:
    - Dân số thống kê chính thức của đơn vị hành chính ($P_u$) được phân bổ cho từng footprint-part ($b$) thuộc đơn vị hành chính $u$ theo tỷ lệ diện tích:

        $$
        P_{b,u} = P_u \times \frac{A_{b,u}}{\sum_j A_{j,u}}
        $$

    - Trong đó:
        - $P_u$: Dân số thống kê chính thức của đơn vị hành chính $u$ (`POP_STAT`).
        - $A_{b,u}$: Diện tích của footprint-part $b$ nằm trong đơn vị hành chính $u$.
        - $\sum_j A_{j,u}$: Tổng diện tích của tất cả các footprint-part nằm trong đơn vị hành chính $u$.
        - $P_{b,u}$: Dân số được phân bổ cho footprint-part $b$ (lưu trong trường `POP_ALLOC`).

!!! warning "Lưu ý phương pháp luận về POP_ALLOC và Dấu vết công trình"
    1. **`POP_ALLOC`** là **dân số được phân bổ theo không gian phục vụ mô hình hóa**, KHÔNG phải là dân số quan sát trực tiếp, kiểm kê hộ tịch hay điều tra thực địa tại từng công trình.
    2. **Dấu vết công trình xây dựng** đại diện cho không gian xây dựng vật lý, KHÔNG đồng nhất hoàn toàn với nhà ở dân cư, số hộ gia đình hay nơi cư trú thực tế.

---

### 2.4. Các tham số phân tích

Các tham số phân tích không phải là tệp dữ liệu không gian mà là các thiết lập ngưỡng được quản lý trong cửa sổ **Cài đặt ⚙️ → Tùy chỉnh nâng cao** (gồm ngưỡng MNDWI, ngưỡng SAVI, diện tích mảng xanh tối thiểu $A_{\min}$, bán kính tối đa $R_{\max}$, chỉ tiêu $C_{\min}$, bước tìm kiếm $\Delta d$, và sai số hội tụ dân số $e$).

---

## 3. Checklist trước khi phân tích

Trước khi nhấn nút **Phân tích**, hãy kiểm tra lại các mục sau:

- [ ] **Hệ tọa độ phẳng dự chiếu**: Lớp ranh giới hành chính và footprint công trình sử dụng hệ tọa độ phẳng dự chiếu (Projected CRS, đơn vị mét) phù hợp.
- [ ] **Dữ liệu vector hợp lệ**: Các lớp vector có cấu trúc hình học hợp lệ (Valid Geometries).
- [ ] **Đầy đủ dữ liệu đầu vào**: Đã chọn đầy đủ các kênh ảnh Sentinel-2, lớp ranh giới hành chính và lớp dấu vết công trình.
- [ ] **Kênh Sentinel-2 đầy đủ**: Tối thiểu gồm các kênh bắt buộc B03, B04, B08, B11.
- [ ] **Trường dân số hợp lệ**: Đã chọn đúng trường thuộc tính chứa số liệu dân số thống kê dạng số dương.

# Hướng dẫn sử dụng Plugin Green Space Evaluator

> **Đồ án tốt nghiệp:** Xây dựng Plugin trên QGIS hỗ trợ tự động hóa đánh giá mức độ phục vụ của mảng xanh đô thị  
> **Sinh viên thực hiện:** Huỳnh Hoàng Xuân Phi  
> **Đơn vị:** Khoa Trắc địa, Bản đồ và Công trình – Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh  
> **Năm thực hiện:** 2026  

---

## 1. Giới thiệu

**Green Space Evaluator** (tên hiển thị trong QGIS: *Urban Green Space Service Evaluator*) là Plugin chạy trên nền tảng QGIS, hỗ trợ tự động hóa toàn diện quy trình trích xuất mảng xanh đô thị từ ảnh vệ tinh Sentinel-2, mô hình hóa vùng phục vụ cho từng mảng xanh độc lập theo khoảng cách Euclid và chỉ tiêu diện tích bình quân đầu người, phân tích không gian xây dựng và tổng hợp kết quả theo đơn vị hành chính.

Plugin giải quyết bài toán đánh giá phạm vi phục vụ của mảng xanh dựa trên diện tích mảng xanh, dân số được phân bổ theo không gian và khoảng cách đến mảng xanh. Toàn bộ quy trình được thực hiện tự động và khép kín từ khâu chuẩn bị dữ liệu, trích xuất thực vật, phân bổ dân số không gian, xác định bán kính phục vụ đến thống kê và kiểm tra tính toàn vẹn của kết quả.

<div class="guide-figure" markdown>
![Giao diện chính Plugin Green Space Evaluator](images/giao_dien_chinh_plugin.png)
<div class="guide-caption"><strong>Hình 1.</strong> Giao diện chính của Plugin Green Space Evaluator trên QGIS</div>
</div>

---

## 2. Quy trình phân tích qua 5 Module

Quy trình phân tích của Plugin gồm 5 module chức năng liên hoàn:

1. **Module 1 — Chuẩn bị và kiểm tra dữ liệu đầu vào:** Tiếp nhận và kiểm tra cấu trúc hình học các lớp vector; chuẩn hóa trường dân số thống kê của đơn vị hành chính thành `POP_STAT`; thiết lập hệ tọa độ phẳng dự chiếu tham chiếu và tạo ảnh Sentinel-2 Stack 10 m.
2. **Module 2 — Phân tách mảng xanh đô thị:** Tính toán các chỉ số quang học MNDWI và SAVI; loại trừ mặt nước; bóc tách thực vật; lọc bỏ mảng xanh nhỏ ngoài công viên theo diện tích tối thiểu và định danh từng mảng xanh riêng biệt (`PATCH_ID`).
3. **Module 3 — Mô hình hóa vùng phục vụ mảng xanh:** Xây dựng lưới khoảng cách Euclid từ biên mảng xanh; phân bổ dân số thống kê theo footprint công trình tạo lớp `POP_ALLOC`; xác định quy mô dân số mục tiêu ($P_{target,i} = S_i / C$) và tìm kiếm bán kính phục vụ lớn nhất $R^*$ thỏa điều kiện của mô hình cho từng mảng xanh.
4. **Module 4 — Phân tích không gian xây dựng:** Phân tách footprint công trình thành các phần hình học (`footprint-part`); xác định footprint-part nằm trong và ngoài vùng phục vụ; áp dụng thuật toán gán độc quyền (**Exclusive Assignment**) cho mảng xanh gần nhất để loại trừ hoàn toàn việc đếm lặp.
5. **Module 5 — Tổng hợp và thống kê kết quả:** Tổng hợp các chỉ tiêu định lượng theo đơn vị hành chính trực tiếp từ các lớp vector; tự động thực hiện 8 phép kiểm tra cân bằng dữ liệu (**Balance Checks**); xuất 5 sản phẩm đầu ra chính cùng các kết quả bổ sung lên QGIS và tệp lưu trữ ngoài.

---

## 3. Dữ liệu đầu vào

Plugin sử dụng **4 nhóm dữ liệu đầu vào** không gian:

| STT | Nhóm dữ liệu | Dạng dữ liệu | Yêu cầu kỹ thuật & Vai trò |
|---|---|---|---|
| 1 | **Ảnh Sentinel-2** | Raster (`.tif`, `.jp2`) | • **Bắt buộc:** Kênh B03 (Green, 10 m), B04 (Red, 10 m), B08 (NIR, 10 m), B11 (SWIR, 20 m) để tính chỉ số MNDWI, SAVI và bóc tách thực vật.<br>• **Tùy chọn:** Kênh B02 (Blue, 10 m - hỗ trợ tạo ảnh màu tự nhiên RGB); Kênh SCL (20 m - hỗ trợ lọc mây và bóng mây). |
| 2 | **Ranh giới hành chính và dân số thống kê** | Vector Polygon | Lớp ranh giới phân chia các đơn vị hành chính (phường/xã). Người dùng chọn trường thuộc tính chứa số liệu dân số thống kê trên giao diện; Plugin tự động chuẩn hóa nội bộ thành trường `POP_STAT`. Lớp này đóng vai trò khung tham chiếu không gian. |
| 3 | **Công viên/vườn hoa** | Vector Polygon | Lớp polygon thể hiện phạm vi công viên/vườn hoa được sử dụng làm vùng mẫu trong quá trình xác định mảng xanh và hỗ trợ phân loại mảng xanh. |
| 4 | **Dấu vết công trình xây dựng (Footprint)** | Vector Polygon | Lớp polygon dấu vết chân công trình xây dựng (Google Open Buildings), đại diện cho không gian xây dựng vật lý. Được sử dụng để phân bổ dân số thống kê theo không gian (`POP_ALLOC`). |

👉 Xem hướng dẫn chi tiết: [Chuẩn bị dữ liệu đầu vào](du-lieu-dau-vao.md)

---

## 4. Tham số và giá trị mặc định

Các tham số tính toán được quản lý trong cửa sổ **Cài đặt** (nút *Cài đặt* ở góc trên bên phải giao diện chính):

| Tham số trên giao diện | Giá trị mặc định | Đơn vị | Ý nghĩa khoa học & Khuyến nghị |
|---|---:|:---:|---|
| **Ngưỡng MNDWI** | `0.0` | — | Giá trị mặc định được lựa chọn trong cấu hình nghiên cứu/thử nghiệm hiện tại; pixel có MNDWI > 0.0 được phân loại là nước và loại trừ. |
| **Hệ số hiệu chỉnh nền đất SAVI (L)** | `0.5` | — | Hệ số hiệu chỉnh nền đất L = 0.5, là giá trị mặc định trong cấu hình nghiên cứu/thử nghiệm hiện tại. |
| **Phương thức xác định ngưỡng SAVI** | `Tự động xác định` | — | Tự động tính ngưỡng từ vùng mẫu công viên/vườn hoa (mặc định theo bách phân vị P10). |
| **Phương pháp xác định ngưỡng SAVI** | `Bách phân vị P10 (Mặc định)` | — | Phương pháp mặc định trong cấu hình nghiên cứu/thử nghiệm hiện tại. |
| **Diện tích mảng xanh tối thiểu** | `5000.0` | m² | Giá trị cấu hình mặc định/tham chiếu trong nghiên cứu/thử nghiệm nhằm loại bỏ các cụm thực vật nhỏ lẻ ngoài công viên. |
| **Bán kính phục vụ tối đa (Rmax)** | `300.0` | m | Giới hạn trên của miền tìm kiếm bán kính phục vụ $R^*$ cho từng mảng xanh trong nghiên cứu/thử nghiệm. |
| **Chỉ tiêu diện tích mảng xanh bình quân đầu người (C)** | `6.0` | m²/người | Chỉ tiêu diện tích mảng xanh bình quân dùng để xác định quy mô dân số mục tiêu: $P_{target,i} = S_i / C$. |
| **Sai số hội tụ khi xác định bán kính** | `10.0` | m | Điều kiện dừng sai số khoảng cách của thuật toán tìm kiếm nhị phân (Binary Search). |

!!! info "Các giá trị cấu hình mặc định trong nghiên cứu"
    Trong cấu hình thử nghiệm, các giá trị mặc định như bán kính phục vụ tối đa **300 m**, chỉ tiêu diện tích bình quân đầu người **6.0 m²/người** và diện tích lọc **5000 m²** được lựa chọn làm giá trị cấu hình tham chiếu phục vụ nghiên cứu. Đây không phải là các giới hạn quy chuẩn áp đặt cho mọi đô thị; người dùng có thể tùy chỉnh linh hoạt phù hợp với quy chuẩn địa phương và bối cảnh từng khu vực nghiên cứu.

👉 Xem hướng dẫn chi tiết: [Cài đặt tham số](tham-so.md)

---

## 5. Quy trình sử dụng nhanh

```
[Nạp 4 nhóm dữ liệu] ➔ [Cấu hình tham số (nếu cần)] ➔ [Chọn nơi lưu 5 sản phẩm] ➔ [Nhấn "Phân tích"]
```

1. **Khởi động Plugin:** Mở QGIS, nhấn vào biểu tượng chiếc lá trên thanh công cụ hoặc vào menu **Plugins** → **Urban Green Space Service Evaluator**.
2. **Nạp dữ liệu:** Tại tab **Dữ liệu đầu vào**, chọn lần lượt 4 nhóm dữ liệu (Ảnh Sentinel-2, Lớp ranh giới hành chính kèm trường dân số, Lớp công viên/vườn hoa, Lớp footprint công trình).
3. **Cài đặt tham số (tùy chọn):** Nhấn nút **Cài đặt** ở góc trên bên phải để điều chỉnh $R_{\max}$, $C$ hoặc diện tích mảng xanh tối thiểu nếu có nhu cầu riêng.
4. **Chỉ định nơi lưu sản phẩm:** Chuyển sang tab **Sản phẩm đầu ra**, chọn định dạng và thư mục lưu cho 5 sản phẩm chính (hoặc để trống để tạo lớp tạm thời).
5. **Thực thi phân tích:** Nhấn nút **Phân tích** ở góc dưới cùng. Theo dõi nhật ký tiến trình hiển thị trực tiếp. Khi hoàn tất, các lớp kết quả sẽ được tự động thêm vào bản đồ QGIS kèm kiểu dáng trực quan.

👉 Xem hướng dẫn chi tiết: [Chạy phân tích](chay-phan-tich.md)

---

## 6. 5 sản phẩm đầu ra chính

Plugin tạo ra **5 sản phẩm đầu ra chính**:

1. **Mảng xanh đô thị** (Raster GeoTIFF): Lớp phân bố thảm thực vật mảng xanh sau khi phân loại chỉ số phổ, trừ mặt nước và lọc diện tích.
2. **Vùng phục vụ mảng xanh ($R^*$)** (Vector Polygon): Phạm vi không gian đệm bán kính lớn nhất $R^*$ thỏa điều kiện của mô hình cho từng mảng xanh độc lập.
3. **Không gian xây dựng trong vùng phục vụ** (Vector Polygon): Các phần hình học công trình (`footprint-part`) được phục vụ bởi mảng xanh đô thị.
4. **Không gian xây dựng ngoài vùng phục vụ** (Vector Polygon): Các phần hình học công trình (`footprint-part`) nằm ngoài vùng phục vụ mảng xanh.
5. **Thống kê theo đơn vị hành chính** (Vector Polygon): Lớp ranh giới tích hợp 10 trường chỉ số định lượng đánh giá toàn diện về diện tích mảng xanh, diện tích xây dựng và dân số được phân bổ.

*Lưu ý:* Bảng số liệu thống kê chi tiết định dạng Excel (`.xlsx`) hoặc CSV (`.csv`) là sản phẩm báo cáo bổ sung tùy chọn theo cấu hình lưu của người dùng.

👉 Xem chi tiết cấu trúc trường và sản phẩm: [Sản phẩm & chỉ tiêu thống kê](san-pham.md)

---

## 7. Lưu ý phương pháp luận cốt lõi

!!! warning "Các nguyên tắc phương pháp luận cần lưu ý"
    - **Dân số phân bổ (`POP_ALLOC`):** Dân số được phân bổ theo tỷ lệ diện tích footprint công trình từ dân số thống kê của đơn vị hành chính. Đây là **dân số được phân bổ theo không gian** phục vụ mô hình hóa, không phải dân số quan sát hoặc kiểm kê thực tế tại từng công trình.
    - **Dấu vết công trình xây dựng (Footprint):** Đại diện cho **không gian xây dựng vật lý (built space)**, không đồng nhất hoàn toàn với nhà ở, hộ gia đình hoặc nơi cư trú.
    - **Khoảng cách Euclid:** Vùng phục vụ được tính toán theo **khoảng cách hình học Euclid**, không phải cự ly di chuyển thực tế theo mạng lưới giao thông đường bộ.
    - **Bán kính phục vụ $R^*$:** Được xác định độc lập cho từng mảng xanh dựa trên tương quan giữa diện tích mảng xanh, dân số phân bổ, chỉ tiêu $C$ và giới hạn $R_{\max}$. $R^*$ là bán kính phục vụ lớn nhất trong miền tìm kiếm thỏa mãn điều kiện của mô hình.
    - **Xử lý chồng lấn (Exclusive Assignment):** Các vùng đệm $R^*$ của từng mảng xanh được giữ nguyên độc lập. Việc xử lý chồng lấn được thực hiện ở bước gán không gian xây dựng (Module 4): mỗi footprint-part chỉ được gán độc quyền cho một mảng xanh gần nhất để loại trừ hoàn toàn việc đếm lặp.

👉 Xem đầy đủ lưu ý: [Lưu ý khi sử dụng](luu-y.md) | [Lỗi thường gặp](loi-thuong-gap.md)

# Hướng dẫn sử dụng Plugin Green Space Evaluator

> **Đồ án tốt nghiệp:** Xây dựng Plugin trên QGIS hỗ trợ tự động hóa đánh giá mức độ phục vụ của mảng xanh đô thị  
> **Sinh viên thực hiện:** Huỳnh Hoàng Xuân Phi  
> **Đơn vị:** Khoa Trắc địa, Bản đồ và Công trình – Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh  
> **Năm thực hiện:** 2026  

---

## 1. Giới thiệu

**Green Space Evaluator** (tên hiển thị trong QGIS: *Urban Green Space Service Evaluator*) là Plugin chạy trên nền tảng QGIS, hỗ trợ tự động hóa toàn diện quy trình trích xuất mảng xanh đô thị từ ảnh vệ tinh Sentinel-2, mô hình hóa vùng phục vụ cho từng mảng xanh độc lập theo bán kính phục vụ $R^*$, hợp nhất thành vùng phục vụ mảng xanh chung (`SERVICE_UNION`), phân tích không gian xây dựng và tổng hợp kết quả theo từng đơn vị hành chính.

Plugin giải quyết bài toán đánh giá mức độ phục vụ của mảng xanh dựa trên diện tích mảng xanh thực tế, dân số thống kê chính thức được phân bổ theo không gian công trình và khoảng cách hình học. Toàn bộ quy trình được thực hiện tự động và khép kín qua 5 giai đoạn: từ chuẩn bị dữ liệu, trích xuất quang phổ thực vật, phân bổ dân số không gian, xác định bán kính phục vụ khả thi, phân rã không gian xây dựng đến tổng hợp thống kê và kiểm tra 8 điều kiện cân bằng dữ liệu.

<div class="guide-figure" markdown>
![Giao diện chính Plugin Green Space Evaluator](images/giao_dien_chinh_plugin.png)
<div class="guide-caption"><strong>Hình 1.</strong> Giao diện chính của Plugin Green Space Evaluator trên QGIS</div>
</div>

---

## 2. Quy trình phân tích qua 5 Module

Quy trình phân tích của Plugin gồm 5 module chức năng:

1. **Module 1 — Chuẩn bị và kiểm tra dữ liệu đầu vào:** Tiếp nhận và kiểm tra cấu trúc hình học các lớp vector; chuẩn hóa trường dân số thống kê của đơn vị hành chính thành `POP_STAT`; thiết lập hệ tọa độ phẳng dự chiếu tham chiếu và tạo ảnh Sentinel-2 Stack 10 m.
2. **Module 2 — Phân tách mảng xanh đô thị:** Tính toán các chỉ số quang học MNDWI và SAVI ($L = 0.5$); loại trừ mặt nước; bóc tách thực vật; lọc bỏ mảng xanh nhỏ có diện tích dưới ngưỡng $A_{\min} = 5.000\text{ m}^2$, định danh từng mảng xanh riêng biệt (`PATCH_ID`) và vector hóa thành các polygon mảng xanh đô thị.
3. **Module 3 — Mô hình hóa vùng phục vụ mảng xanh ($R^*$):** Phân chia footprint công trình theo đơn vị hành chính và phân bổ dân số thống kê `POP_STAT` theo tỷ lệ diện tích để tạo `POP_ALLOC`; xây dựng chỉ mục không gian `QgsSpatialIndex`; xác định bán kính phục vụ $R^*$ cho từng mảng xanh bằng phương pháp tìm kiếm hai giai đoạn (khoanh vùng nghiệm theo bước $\Delta d = 50\text{ m}$ và tinh chỉnh bằng tìm kiếm nhị phân theo ngưỡng hội tụ dân số $e = 1.0\text{ người}$); tạo vùng phục vụ riêng `OUT_BUFFER` cho từng mảng xanh.
4. **Module 4 — Phân tích không gian xây dựng:** Hợp nhất các vùng phục vụ riêng thành vùng phục vụ mảng xanh chung (`SERVICE_UNION`, các vùng chồng lấn chỉ tính một lần); phân tích giao cắt không gian với các footprint-part; phân rã thành các phần diện tích xây dựng nằm trong vùng phục vụ (`SERVED`) và nằm ngoài vùng phục vụ (`OUTSIDE`); phân bổ dân số nguồn `SOURCE_POP_ALLOC` cho từng phần diện tích thành `POP_FRAGMENT` theo tỷ lệ diện tích thực tế.
5. **Module 5 — Tổng hợp và thống kê kết quả:** Tổng hợp các chỉ tiêu định lượng theo đơn vị hành chính trực tiếp từ các lớp vector; tự động thực hiện 8 phép kiểm tra cân bằng dữ liệu (**Balance Checks**); xuất 5 sản phẩm đầu ra chính cùng các kết quả bổ sung lên QGIS và tệp lưu trữ ngoài.

---

## 3. Dữ liệu đầu vào

Plugin sử dụng **4 nhóm dữ liệu đầu vào** chuẩn:

| STT | Nhóm dữ liệu | Dạng dữ liệu | Yêu cầu kỹ thuật & Vai trò |
|---|---|---|---|
| 1 | **Ảnh Sentinel-2** | Raster (`.tif`, `.jp2`) | • **Bắt buộc:** Kênh B03 (Green, 10 m), B04 (Red, 10 m), B08 (NIR, 10 m), B11 (SWIR-1, 20 m) để tính chỉ số MNDWI, SAVI và bóc tách thực vật.<br>• **Tùy chọn:** Kênh B02 (Blue, 10 m - hỗ trợ tạo ảnh màu tự nhiên RGB); Kênh SCL (20 m - hỗ trợ lọc mây và bóng mây). |
| 2 | **Ranh giới hành chính và dân số thống kê** | Vector Polygon | Lớp ranh giới phân chia các đơn vị hành chính (phường/xã). Người dùng chọn trường thuộc tính chứa số liệu dân số thống kê trên giao diện; Plugin tự động chuẩn hóa nội bộ thành trường `POP_STAT`. Dân số được lấy từ số liệu thống kê thuộc tính, không lấy từ raster dân số. |
| 3 | **Dấu vết công trình xây dựng (Building Footprints)** | Vector Polygon | Lớp polygon dấu vết chân công trình xây dựng, đại diện cho không gian xây dựng vật lý. Được sử dụng để phân chia thành các footprint-part và phân bổ dân số thống kê theo không gian (`POP_ALLOC`). |
| 4 | **Các tham số phân tích** | Cấu hình tham số | Các ngưỡng quang phổ, chỉ tiêu không gian và tham số tìm kiếm nghiệm số học (thiết lập tại cửa sổ Cài đặt). |

👉 Xem hướng dẫn chi tiết: [Chuẩn bị dữ liệu đầu vào](du-lieu-dau-vao.md)

---

## 4. Tham số và giá trị cấu hình

Các tham số tính toán được quản lý trong cửa sổ **Cài đặt ⚙️** (tab **Tùy chỉnh nâng cao**):

| Tham số trên giao diện | Giá trị cấu hình | Đơn vị | Ý nghĩa khoa học & Khuyến nghị |
|---|---:|:---:|---|
| **Ngưỡng MNDWI** | `0.000` | — | Giá trị cấu hình chuẩn production; pixel có $\text{MNDWI} > 0.000$ được phân loại là nước và loại trừ trước khi phân tích thực vật. |
| **Ngưỡng SAVI** | `0.165` (Thủ công) | — | Ngưỡng SAVI dùng để bóc tách thực vật sau khi loại trừ mặt nước (nhập thủ công giá trị cấu hình 0.165; có tùy chọn tự động theo mẫu nếu cần). |
| **Hệ số hiệu chỉnh nền đất SAVI ($L$)** | `0.50` | — | Hệ số hiệu chỉnh nền đất $L = 0.50$, giảm ảnh hưởng phản xạ của nền đất trong công thức SAVI. |
| **Diện tích mảng xanh tối thiểu ($A_{\min}$)** | `5000.0` | m² | Ngưỡng lọc bỏ các cụm thực vật nhỏ lẻ manh mún (thảm cỏ nhỏ, dải phân cách, bóng cây). Chỉ giữ các mảng xanh có diện tích $\ge 5000.0\text{ m}^2$. |
| **Bán kính phục vụ tối đa ($R_{\max}$)** | `300.0` | m | Giới hạn trên của miền tìm kiếm bán kính phục vụ $R^*$ cho từng mảng xanh. |
| **Chỉ tiêu diện tích mảng xanh tối thiểu ($C_{\min}$)** | `6.0` | m²/người | Chỉ tiêu diện tích mảng xanh bình quân đầu người tối thiểu dùng để xác định sức chứa dân số tối đa: $P_{\max, i} = S_i / C_{\min}$. |
| **Bước tìm kiếm bán kính ($\Delta d$)** | `50.0` | m | Bước giảm bán kính trong giai đoạn khoanh vùng nghiệm của Module 3. |
| **Ngưỡng sai số hội tụ dân số ($e$)** | `1.0` | người | Tham số hội tụ số học dùng làm điều kiện dừng của tìm kiếm nhị phân: $P(R_{\text{high}}) - P(R_{\text{low}}) \le e$. |

👉 Xem hướng dẫn chi tiết: [Cài đặt tham số](tham-so.md)

---

## 5. Quy trình sử dụng nhanh

```
[Nạp 4 nhóm dữ liệu] ➔ [Kiểm tra tham số phân tích] ➔ [Chọn nơi lưu 5 sản phẩm] ➔ [Nhấn "Phân tích"]
```

1. **Khởi động Plugin:** Mở QGIS, nhấn vào biểu tượng chiếc lá trên thanh công cụ hoặc vào menu **Plugins** → **Urban Green Space Service Evaluator**.
2. **Nạp dữ liệu:** Tại tab **Dữ liệu đầu vào**, nạp lần lượt các kênh ảnh Sentinel-2, lớp ranh giới hành chính kèm trường dân số thống kê `POP_STAT`, và lớp dấu vết công trình xây dựng.
3. **Cài đặt tham số (tùy chọn):** Nhấn nút **Cài đặt ⚙️** ở góc trên bên phải để kiểm tra các tham số phân tích (MNDWI, SAVI, $A_{\min}$, $R_{\max}$, $C_{\min}$, $\Delta d$, $e$).
4. **Chỉ định nơi lưu sản phẩm:** Chuyển sang tab **Sản phẩm đầu ra**, chọn đường dẫn lưu cho 5 sản phẩm chính (khuyến nghị định dạng GeoPackage `.gpkg`; hoặc để trống để tạo lớp tạm thời).
5. **Thực thi phân tích:** Nhấn nút **Phân tích**. Theo dõi nhật ký tiến trình hiển thị trực tiếp. Khi hoàn tất, các lớp kết quả sẽ được tự động thêm vào bản đồ QGIS kèm kiểu hiển thị chuẩn trực quan.

👉 Xem hướng dẫn chi tiết: [Chạy phân tích](chay-phan-tich.md)

---

## 6. 5 sản phẩm đầu ra chính

Plugin tạo ra **5 sản phẩm đầu ra chính**:

1. **Mảng xanh đô thị** (Raster GeoTIFF, `OUT_PARK`): Lớp phân bố thảm thực vật mảng xanh sau khi phân loại chỉ số phổ, trừ mặt nước và lọc diện tích tối thiểu.
2. **Vùng phục vụ mảng xanh** (Vector Polygon, `SERVICE_UNION`): Vùng phục vụ chung được tạo bằng cách hợp nhất các vùng phục vụ riêng của từng mảng xanh theo bán kính khả thi $R^*$. Các phần chồng lấn chỉ được tính một lần.
3. **Không gian xây dựng trong vùng phục vụ** (Vector Polygon, `SERVED`): Các phần diện tích không gian xây dựng nằm trong vùng phục vụ chung. Mỗi phần diện tích mang trường dân số `POP_FRAGMENT` được phân bổ từ `SOURCE_POP_ALLOC` theo tỷ lệ diện tích.
4. **Không gian xây dựng ngoài vùng phục vụ** (Vector Polygon, `OUTSIDE`): Các phần diện tích không gian xây dựng nằm ngoài vùng phục vụ chung trong phạm vi phân tích.
5. **Thống kê kết quả theo đơn vị hành chính** (Vector Polygon, `OUT_STATS`): Lớp ranh giới tích hợp 10 trường chỉ số định lượng đánh giá toàn diện về diện tích mảng xanh, diện tích xây dựng và dân số được phân bổ.

*Lưu ý:* Vùng phục vụ riêng của từng mảng xanh (`OUT_BUFFER`) là sản phẩm bổ sung / trung gian phục vụ kiểm tra chi tiết. Bảng số liệu thống kê chi tiết định dạng Excel (`.xlsx`) hoặc CSV (`.csv`) là sản phẩm báo cáo bổ sung tùy chọn.

👉 Xem chi tiết cấu trúc trường và sản phẩm: [Sản phẩm & chỉ tiêu thống kê](san-pham.md)

---

## 7. Lưu ý phương pháp luận

!!! warning "Các nguyên tắc phương pháp luận cần lưu ý"
    - **Dân số phân bổ (`POP_ALLOC`, `POP_FRAGMENT`):** Dân số được phân bổ theo tỷ lệ diện tích dấu vết công trình từ dân số thống kê chính thức của đơn vị hành chính. Đây là **dân số được phân bổ theo không gian** phục vụ mô hình hóa, không phải dân số quan sát hoặc kiểm kê thực tế tại từng công trình.
    - **Dấu vết công trình xây dựng (Footprint):** Đại diện cho **không gian xây dựng vật lý (built space)**, không đồng nhất hoàn toàn với nhà ở, hộ gia đình hoặc nơi cư trú.
    - **Khoảng cách hình học:** Vùng phục vụ được mô hình hóa theo **khoảng cách hình học**, không phải cự ly di chuyển thực tế theo mạng lưới giao thông đường bộ.
    - **Bán kính phục vụ $R^*$:** Được xác định độc lập cho từng mảng xanh dựa trên tương quan giữa diện tích mảng xanh, dân số phân bổ, chỉ tiêu diện tích mảng xanh tối thiểu $C_{\min}$ và giới hạn $R_{\max}$. Nghiệm bán kính khả thi được chọn là $R^* = R_{\text{low}}$ (cận dưới khả thi của khoảng nghiệm đảm bảo dân số phục vụ không vượt quá $P_{\max, i}$).
    - **Vùng phục vụ chung (`SERVICE_UNION`):** Các vùng phục vụ riêng `OUT_BUFFER` được hợp nhất thành vùng phục vụ chung `SERVICE_UNION`. Các phần diện tích chồng lấn giữa nhiều vùng phục vụ riêng được gộp lại và chỉ tính đúng một lần, loại bỏ hoàn toàn hiện tượng đếm lặp dân số và diện tích phục vụ.

👉 Xem đầy đủ lưu ý: [Lưu ý khi sử dụng](luu-y.md) | [Lỗi thường gặp](loi-thuong-gap.md)

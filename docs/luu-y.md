# Lưu ý khi sử dụng

Trang này tổng hợp các lưu ý phương pháp luận cốt lõi, phạm vi ứng dụng và các giả định khoa học của Plugin **Green Space Evaluator**. Người dùng cần nắm rõ các điểm này để diễn giải và sử dụng kết quả đánh giá một cách chuẩn xác và khoa học.

---

## 1. Bản chất dữ liệu dân số phân bổ (`POP_ALLOC`, `POP_FRAGMENT`)

!!! info "Dân số phân bổ theo không gian"
    1. **Nguồn gốc số liệu:** Trường `POP_STAT` trong Plugin được lấy từ số liệu dân số thống kê chính thức của đơn vị hành chính (phường/xã). Dân số sau đó được phân bổ xuống từng footprint-part theo tỷ trọng diện tích:
    
        $$P_{b,u} = P_u \times \frac{A_{b,u}}{\sum_j A_{j,u}}$$
        
    2. **Phân rã phần diện tích:** Khi một footprint-part bị ranh giới vùng phục vụ chung `SERVICE_UNION` phân cắt, dân số của phần diện tích xây dựng (`POP_FRAGMENT`) được bảo toàn và phân bổ theo tỷ lệ diện tích từ `SOURCE_POP_ALLOC`.
    3. **Ý nghĩa khoa học:** `POP_ALLOC` và `POP_FRAGMENT` là **dân số được phân bổ theo không gian phục vụ mô hình hóa**, KHÔNG phải là dân số quan sát, kiểm kê hộ tịch hay điều tra thực địa tại từng công trình cụ thể.

---

## 2. Bản chất dữ liệu dấu vết công trình xây dựng (Footprint)

!!! info "Đại diện cho không gian xây dựng vật lý"
    1. **Dữ liệu dấu vết chân công trình:** Là tập dữ liệu nhận diện hình học chân công trình xây dựng từ ảnh vệ tinh độ phân giải cao bằng mô hình học máy (ví dụ Google Open Buildings) hoặc dữ liệu đo đạc địa chính.
    2. **Không đồng nhất với nhà ở:** Dữ liệu dấu vết công trình đại diện cho **không gian xây dựng vật lý (built space)**, không được đồng nhất hoàn toàn với dữ liệu nhà ở dân sinh, nơi cư trú, số căn hộ hay số hộ gia đình (vì có thể bao gồm nhà xưởng, cơ quan, trường học, bệnh viện, thương mại dịch vụ).

---

## 3. Bản chất mảng xanh trích xuất từ ảnh vệ tinh

!!! info "Lớp phủ thực vật bề mặt"
    1. **Hiện trạng lớp phủ phổ:** Mảng xanh bóc tách từ ảnh Sentinel-2 bằng chỉ số SAVI phản ánh hiện trạng lớp phủ thực vật bề mặt tại thời điểm vệ tinh chụp ảnh sau khi đã trừ mặt nước (MNDWI > 0.000) và lọc quy mô diện tích tối thiểu $A_{\min} = 5000\text{ m}^2$.
    2. **Không đồng nhất với đất cây xanh quy hoạch:** Mảng xanh bề mặt có thể bao gồm cây xanh tư nhân, thảm cây bụi ven rạch, vườn cây ăn trái... Lớp này không đồng nhất hoàn toàn với diện tích đất cây xanh sử dụng công cộng theo hồ sơ pháp lý quy hoạch đô thị.

---

## 4. Mô hình hóa khoảng cách hình học

!!! warning "Khoảng cách hình học so với khoảng cách mạng lưới giao thông"
    1. **Khoảng cách hình học:** Vùng phục vụ của mảng xanh được tính toán dựa trên khoảng cách hình học tính từ biên mảng xanh ra xung quanh.
    2. **Giới hạn ứng dụng:** Khoảng cách này không phản ánh cự ly đi bộ thực tế dọc theo mạng lưới đường giao thông đô thị và chưa xét đến các rào cản vật lý như tường rào, sông ngòi chia cắt hoặc cổng vào công viên.

---

## 5. Ý nghĩa bán kính phục vụ khả thi R* và giới hạn R_max

!!! note "Bán kính phục vụ khả thi của mảng xanh"
    1. **Tính độc lập của từng mảng xanh:** Mỗi mảng xanh có dân số phục vụ tối đa riêng $P_{\max, i} = S_i / C_{\min}$ và được xác định một bán kính phục vụ $R_i^*$ riêng trong khoảng $[0, R_{\max}]$.
    2. **Các yếu tố quyết định $R^*$:** Bán kính phục vụ $R^*$ phụ thuộc đồng thời vào diện tích mảng xanh ($S_i$), phân bố không gian của các công trình xây dựng xung quanh, quy mô dân số phân bổ (`POP_ALLOC`), chỉ tiêu diện tích mảng xanh tối thiểu ($C_{\min}$), giới hạn trên ($R_{\max}$), bước tìm kiếm $\Delta d$ và sai số hội tụ dân số $e$.
    3. **Bản chất khoa học của $R^*$:** Nghiệm bán kính khả thi được chọn là $R^* = R_{\text{low}}$ (cận dưới khả thi của khoảng nghiệm đảm bảo dân số phục vụ thực tế không vượt quá $P_{\max, i}$).
    4. **Giới hạn $R_{\max}$:** Mặc định tham chiếu $300\text{ m}$ đóng vai trò là cận trên để khống chế phạm vi tìm kiếm của thuật toán, không phải là quy chuẩn áp đặt cho mọi không gian xanh.

---

## 6. Xử lý vùng phục vụ chung (SERVICE_UNION)

!!! tip "Loại bỏ hoàn toàn đếm lặp ở cấp độ vùng phục vụ chung"
    1. **Hợp nhất không gian dịch vụ:** Các vùng phục vụ riêng `OUT_BUFFER` của từng mảng xanh được hợp nhất bằng phép Unary Union thành vùng phục vụ mảng xanh chung `SERVICE_UNION`.
    2. **Loại bỏ đếm lặp:** Các phần diện tích chồng lấn giữa nhiều vùng phục vụ riêng được hợp nhất và chỉ tính đúng một lần, loại bỏ hoàn toàn việc tính lặp dân số hoặc diện tích phục vụ.
    3. **Phân rã phần diện tích:** Công trình nằm trên biên vùng phục vụ chung được cắt thành các phần diện tích xây dựng nằm trong vùng phục vụ (`SERVED`) và ngoài vùng phục vụ (`OUTSIDE`), dân số được phân bổ tương ứng theo tỷ lệ diện tích thực tế.

---

## 7. Diễn giải kết quả phân tích

!!! warning "Khuyến nghị diễn giải"
    Khái niệm "trong vùng phục vụ" hay "ngoài vùng phục vụ" là kết quả mô hình hóa không gian dựa trên các tham số của mô hình ($C_{\min}$, $R_{\max}$, khoảng cách hình học). Không nên diễn giải đây là số liệu kiểm kê thực địa hoặc khẳng định người dân hoàn toàn không thể tiếp cận cây xanh trên thực tế. Kết quả của Plugin đóng vai trò là công cụ hỗ trợ ra quyết định và quy hoạch đô thị.

---

## Thông tin tác giả

**Huỳnh Hoàng Xuân Phi**  
Khoa Trắc địa, Bản đồ và Công trình  
Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh  
Năm 2026

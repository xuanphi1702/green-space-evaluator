# Lưu ý khi sử dụng

Trang này tổng hợp các lưu ý phương pháp luận cốt lõi, phạm vi ứng dụng và các giả định khoa học của Plugin **Green Space Evaluator**. Người dùng cần nắm rõ các điểm này để diễn giải và sử dụng kết quả đánh giá một cách chuẩn xác và khoa học.

---

## 1. Bản chất dữ liệu dân số phân bổ (`POP_ALLOC`)

!!! info "Dân số phân bổ theo không gian"
    1. **Nguồn gốc số liệu:** Trường `POP_STAT` trong Plugin được lấy từ số liệu dân số thống kê chính thức của đơn vị hành chính (phường/xã). Dân số sau đó được phân bổ xuống từng phần hình học công trình xây dựng (`footprint-part`) theo tỷ trọng diện tích:
    
        $$P_{b,u} = P_u \times \frac{A_{b,u}}{\sum_j A_{j,u}}$$
        
    2. **Ý nghĩa khoa học:** `POP_ALLOC` là **dân số được phân bổ theo không gian phục vụ mô hình hóa**, KHÔNG phải là dân số quan sát, kiểm kê hộ tịch hay điều tra thực địa tại từng công trình cụ thể.

---

## 2. Bản chất dữ liệu dấu vết công trình xây dựng (Footprint)

!!! info "Đại diện cho không gian xây dựng vật lý"
    1. **Dữ liệu Google Open Buildings / OSM:** Đây là tập dữ liệu nhận diện hình học chân công trình xây dựng từ không gian viễn thám.
    2. **Không đồng nhất với nhà ở:** Dữ liệu footprint đại diện cho **không gian xây dựng vật lý (built space)**, không được đồng nhất hoàn toàn với dữ liệu nhà ở dân sinh, nơi cư trú, số căn hộ hay số hộ gia đình (vì có thể bao gồm nhà xưởng, cơ quan, trường học, bệnh viện, thương mại dịch vụ).

---

## 3. Bản chất mảng xanh trích xuất từ ảnh vệ tinh

!!! info "Lớp phủ thực vật bề mặt"
    1. **Hiện trạng lớp phủ phổ:** Mảng xanh bóc tách từ ảnh Sentinel-2 bằng chỉ số SAVI phản ánh hiện trạng lớp phủ thực vật bề mặt tại thời điểm vệ tinh chụp ảnh.
    2. **Không đồng nhất với đất cây xanh quy hoạch:** Mảng xanh bề mặt có thể bao gồm cây xanh tư nhân, thảm cây bụi ven rạch, vườn cây ăn trái... Lớp này không đồng nhất hoàn toàn với diện tích đất cây xanh sử dụng công cộng theo hồ sơ pháp lý quy hoạch đô thị.

---

## 4. Mô hình hóa khoảng cách Euclid

!!! warning "Khoảng cách Euclid so với khoảng cách mạng lưới giao thông"
    1. **Khoảng cách đường chim bay:** Vùng phục vụ của mảng xanh được tính toán dựa trên **khoảng cách hình học Euclid (Euclidean Distance)** tính từ biên mảng xanh ra xung quanh.
    2. **Giới hạn ứng dụng:** Khoảng cách này không phản ánh cự ly đi bộ thực tế dọc theo mạng lưới đường giao thông đô thị và chưa xét đến các rào cản vật lý như tường rào, sông ngòi chia cắt hoặc cổng vào công viên.

---

## 5. Ý nghĩa bán kính phục vụ $R^*$ và giới hạn $R_{\max}$

!!! note "Bán kính phục vụ lớn nhất thỏa điều kiện của mô hình"
    1. **Tính độc lập của từng mảng xanh:** Mỗi mảng xanh có một quy mô dân số phục vụ mục tiêu riêng $P_{target,i} = S_i / C$ và được xác định một bán kính phục vụ $R_i^*$ riêng trong khoảng $[0, R_{\max}]$.
    2. **Các yếu tố quyết định $R^*$:** Bán kính phục vụ $R^*$ phụ thuộc đồng thời vào:
       - Diện tích mảng xanh ($S_i$);
       - Phân bố không gian của các công trình xây dựng xung quanh;
       - Quy mô dân số phân bổ (`POP_ALLOC`);
       - Chỉ tiêu diện tích mảng xanh bình quân đầu người ($C$);
       - Giới hạn trên của miền tìm kiếm ($R_{\max}$).
    3. **Bản chất khoa học của $R^*$:** $R^*$ là **bán kính phục vụ lớn nhất trong miền tìm kiếm thỏa mãn điều kiện của mô hình**.
    4. **Giới hạn $R_{\max}$:** Mặc định $300\text{ m}$ đóng vai trò là cận trên để khống chế phạm vi tìm kiếm của thuật toán, không phải là quy chuẩn áp đặt cho mọi không gian xanh.

---

## 6. Xử lý chồng lấn bằng thuật toán gán độc quyền (Exclusive Assignment)

!!! tip "Loại bỏ hoàn toàn đếm lặp ở cấp độ không gian xây dựng"
    1. **Giữ nguyên vùng đệm mảng xanh:** Vùng phục vụ bán kính $R^*$ của từng mảng xanh được giữ nguyên vẹn độc lập, không gộp lại để bảo toàn tính độc lập của từng mảng xanh.
    2. **Gán độc quyền cho công trình:** Khi một phần công trình (`footprint-part`) nằm trong vùng phục vụ của nhiều mảng xanh, nó được gán cho mảng xanh gần nhất. Mỗi `footprint-part` cuối cùng chỉ thuộc về tối đa một mảng xanh, bảo toàn nguyên tắc cân bằng dữ liệu và không đếm lặp dân số.

---

## 7. Diễn giải kết quả phân tích

!!! warning "Khuyến nghị diễn giải"
    Khái niệm "trong vùng phục vụ" hay "thiếu mảng xanh" là kết quả mô hình hóa không gian dựa trên các tham số của mô hình ($C$, $R_{\max}$, khoảng cách Euclid). Không nên diễn giải đây là số liệu kiểm kê thực địa hoặc khẳng định người dân hoàn toàn không thể tiếp cận cây xanh trên thực tế. Kết quả của Plugin đóng vai trò là công cụ hỗ trợ ra quyết định và quy hoạch đô thị.

---

## Thông tin tác giả

**Huỳnh Hoàng Xuân Phi**  
Khoa Trắc địa, Bản đồ và Công trình  
Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh  
Năm 2026

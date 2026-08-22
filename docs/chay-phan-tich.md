# Chạy phân tích

## Quy trình thực hiện

1. Nạp đầy đủ 5 nhóm dữ liệu trong tab **Dữ liệu đầu vào**.
2. Mở **Cài đặt** nếu cần thay đổi tham số hoặc đường dẫn sản phẩm trung gian.
3. Chuyển sang tab **Sản phẩm đầu ra** và chọn nơi lưu các sản phẩm cần thiết.
4. Nhấn **Phân tích**.
5. Theo dõi tiến trình trong tab **Nhật ký tiến trình**.
6. Sau khi hoàn tất, kiểm tra các lớp kết quả được nạp vào QGIS và các file đã được lưu tại đường dẫn đã chọn.

## Cách xác định bán kính phục vụ

Trong Module 3, Plugin tính ma trận khoảng cách Euclid từ mảng xanh và đánh giá sức tải theo bán kính. Hai biên của miền tìm kiếm được kiểm tra trước; khi cần thiết, thuật toán tìm kiếm nhị phân được sử dụng để xác định bán kính lớn nhất mà sức tải vẫn đạt ngưỡng mục tiêu.

Nếu không tồn tại bán kính thỏa điều kiện trong miền tìm kiếm, thông tin tương ứng được ghi trong **Nhật ký tiến trình**.

<div class="guide-figure" markdown>
![Kết quả phân tích của Plugin](images/ket_qua_plugin_2.png)
<div class="guide-caption"><strong>Hình 3.</strong> Minh họa kết quả phân tích của Plugin</div>
</div>

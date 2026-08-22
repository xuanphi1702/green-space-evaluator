# Cài đặt tham số

Nhấn nút **Cài đặt** trên giao diện chính để mở cửa sổ cấu hình.

<div class="guide-figure" markdown>
![Cửa sổ cài đặt nâng cao của Plugin](images/cai_dat_nang_cao.png)
<div class="guide-caption"><strong>Hình 2.</strong> Cửa sổ cài đặt nâng cao của Plugin</div>
</div>

| Tham số | Giá trị mặc định | Ý nghĩa |
|---|---:|---|
| Ngưỡng MNDWI | 0,0 | Ngưỡng nhận diện pixel mặt nước |
| Hệ số L của SAVI | 0,5 | Hệ số hiệu chỉnh ảnh hưởng nền đất |
| Ngưỡng SAVI | -999 | Chế độ tự động xác định ngưỡng từ mẫu công viên |
| Phương pháp thống kê SAVI | P10 | Bách phân vị 10% được dùng làm giá trị mặc định thử nghiệm |
| Bách phân vị tùy chỉnh | 10% | Chỉ sử dụng khi chọn chế độ bách phân vị tùy chỉnh |
| Diện tích mảng xanh lọc nhiễu tối thiểu | 5000 m² | Ngưỡng kỹ thuật dùng cho các cụm mảng xanh ngoài công viên |
| Bán kính phục vụ tối đa | 300 m | Giới hạn trên của miền tìm kiếm bán kính |
| Sức tải mảng xanh mục tiêu | 6 m²/người | Ngưỡng sức tải dùng trong quá trình xác định bán kính phục vụ |
| Sai số hội tụ | 10 m | Điều kiện dừng của quá trình tìm kiếm nhị phân |

## Các giá trị tham chiếu QCVN trong thử nghiệm của đồ án

- **300 m** được sử dụng làm bán kính phục vụ tối đa, có tham chiếu quy định về bán kính phục vụ của vườn hoa, sân chơi trong nhóm nhà ở.
- **6 m²/người** được sử dụng làm ngưỡng sức tải tham chiếu ở cấp đô thị.
- **5000 m²** được sử dụng làm ngưỡng kỹ thuật thử nghiệm để lọc các cụm mảng xanh nhỏ, có tham chiếu quy mô tối thiểu của công viên, vườn hoa trong đơn vị ở.

Người dùng có thể thay đổi các giá trị này khi áp dụng Plugin cho mục đích hoặc khu vực nghiên cứu khác.

## Sản phẩm trung gian

Người dùng có thể chỉ định đường dẫn lưu các sản phẩm trung gian như raster MNDWI, SAVI, mặt nước, thực vật, raster dân số đã chuẩn hóa và các biểu đồ histogram. Nếu không cần lưu, có thể để trống các trường này.

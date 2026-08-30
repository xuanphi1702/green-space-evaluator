# Green Space Evaluator

**Green Space Evaluator (Urban Green Space Service Evaluator)** là Plugin chạy trên nền tảng QGIS, hỗ trợ tự động hóa quy trình xử lý ảnh Sentinel-2, trích xuất mảng xanh đô thị, mô hình hóa vùng phục vụ theo khoảng cách Euclid, phân tích không gian xây dựng và tổng hợp kết quả thống kê theo đơn vị hành chính.

<div class="guide-figure" markdown>
![Giao diện chính Plugin Green Space Evaluator](images/giao_dien_chinh_plugin.png)
<div class="guide-caption"><strong>Hình 1.</strong> Giao diện chính của Plugin Green Space Evaluator</div>
</div>

## Chức năng chính

Quy trình của Plugin gồm 5 module:

1. Tiền xử lý dữ liệu Sentinel-2 và WorldPop.
2. Tính MNDWI, SAVI và trích xuất mảng xanh đô thị.
3. Tính khoảng cách Euclid và xác định bán kính phục vụ lớn nhất thỏa điều kiện sức tải.
4. Phân tích không gian xây dựng nằm trong và ngoài vùng phục vụ.
5. Thống kê kết quả theo đơn vị hành chính và xuất bảng dữ liệu.

## Bắt đầu sử dụng

[**Cài đặt Plugin**](cai-dat.md){ .md-button .md-button--primary }
[**Chuẩn bị dữ liệu đầu vào**](du-lieu-dau-vao.md){ .md-button }

[**Cài đặt tham số**](tham-so.md){ .md-button }
[**Chạy phân tích**](chay-phan-tich.md){ .md-button }

[**Xem sản phẩm đầu ra**](san-pham.md){ .md-button }

!!! info "Các giá trị mặc định trong thử nghiệm"
    Trong thử nghiệm tại khu vực Biên Hòa, một số giá trị mặc định được lựa chọn có **tham chiếu QCVN 01:2021/BXD**, gồm bán kính phục vụ tối đa **300 m**, sức tải mục tiêu **6 m²/người** và ngưỡng lọc diện tích **5000 m²**. Đây là các giá trị mặc định phục vụ thử nghiệm, không giới hạn Plugin chỉ sử dụng cho QCVN 01:2021/BXD.

---

**Đồ án tốt nghiệp:** Xây dựng Plugin trên QGIS hỗ trợ tự động hóa đánh giá mức độ phục vụ của mảng xanh đô thị  
**Sinh viên thực hiện:** Huỳnh Hoàng Xuân Phi  
**Đơn vị:** Khoa Trắc địa, Bản đồ và Công trình – Trường Đại học Tài nguyên và Môi trường TP. Hồ Chí Minh  
**Năm thực hiện:** 2026

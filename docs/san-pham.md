# Sản phẩm và chỉ tiêu thống kê

## Sản phẩm trung gian

Các sản phẩm trung gian có thể gồm ảnh Sentinel-2 đã chuẩn hóa, raster dân số, MNDWI, SAVI, mặt nước, thực vật, histogram và raster dân số trong không gian xây dựng.

## Sản phẩm chính

Trong tab **Sản phẩm đầu ra**, người dùng có thể chỉ định đường dẫn lưu 5 sản phẩm chính:

1. Raster mảng xanh đô thị.
2. Vector vùng phục vụ tương ứng với bán kính xác định được.
3. Vector không gian xây dựng nằm trong vùng phục vụ.
4. Vector không gian xây dựng nằm ngoài vùng phục vụ.
5. Vector thống kê theo đơn vị hành chính.

Nếu để trống đường dẫn, Plugin sử dụng đầu ra tạm thời trong phiên làm việc QGIS.

## Các trường thống kê

| Trường | Nội dung |
|---|---|
| `T_DanSo` | Tổng dân số ước tính của đơn vị hành chính |
| `S_MangXanh` | Diện tích mảng xanh đô thị (ha) |
| `S_XayDung` | Diện tích không gian xây dựng (ha) |
| `S_ThieuXanh` | Diện tích không gian xây dựng nằm ngoài vùng phục vụ (ha) |
| `D_ThieuXanh` | Dân số ước tính nằm ngoài vùng phục vụ (người) |
| `TL_ThieuXanh` | Tỷ lệ dân số ước tính nằm ngoài vùng phục vụ (%) |

Plugin hỗ trợ xuất bảng thống kê dưới định dạng **Excel (.xlsx)** hoặc **CSV (.csv)**.

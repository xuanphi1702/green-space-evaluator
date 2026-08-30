# Cài đặt Plugin

Trang này hướng dẫn yêu cầu môi trường và các bước cài đặt Plugin **Green Space Evaluator** (tên hiển thị trong QGIS: *Urban Green Space Service Evaluator*) trên nền tảng QGIS.

## Yêu cầu môi trường

Trước khi cài đặt, hãy đảm bảo môi trường làm việc đáp ứng các điều kiện sau:

- **Phiên bản QGIS**: QGIS *3.44.12*  trở lên (khuyến nghị sử dụng phiên bản QGIS LTR phù hợp với yêu cầu của Plugin).
- **Môi trường Python**: Python 3 (tự động tích hợp sẵn trong bộ cài đặt chuẩn của QGIS).
- **Thư viện phụ thuộc**:
    - **Bắt buộc**: `GDAL/OGR`, `NumPy`, `Matplotlib`, `SciPy`. Các thư viện này thông thường đã được tích hợp sẵn trong môi trường Python của bộ cài đặt QGIS Desktop chuẩn.
    - **Tùy chọn**: `openpyxl`, `pandas` (hỗ trợ xuất trực tiếp bảng báo cáo dưới định dạng Excel `.xlsx`; nếu chưa có, Plugin sẽ tự động chuyển sang định dạng `.csv`).

---

## Cài đặt Plugin

Người dùng có thể cài đặt Plugin theo một trong hai cách dưới đây:

### Cài đặt từ file ZIP

Phương pháp thuận tiện và nhanh chóng nhất:

1. Khởi động phần mềm **QGIS**.
2. Trên thanh bảng chọn, vào **Plugins** → **Manage and Install Plugins...** (hoặc **Tiện ích** → **Quản lý và cài đặt tiện ích...**).
3. Trong cửa sổ xuất hiện, chọn thẻ **Install from ZIP** ở cột bên trái.
4. Nhấn nút duyệt (**...**) tại mục **ZIP file** và trỏ đến tệp `.zip` chứa Plugin `green_space_evaluator`.
5. Nhấn **Install Plugin** để tiến hành cài đặt.
6. Sau khi cài đặt hoàn tất, chuyển sang thẻ **Installed** và đảm bảo ô chọn bên cạnh **Urban Green Space Service Evaluator** đã được tích bật.

### Cài đặt thủ công

Nếu cài đặt trực tiếp từ thư mục mã nguồn:

1. Chuẩn bị thư mục mã nguồn của Plugin có tên `green_space_evaluator` (đảm bảo bên trong chứa trực tiếp các tệp của Plugin như `metadata.txt`, `plugin.py`, `green_space_analysis.py`,...).
2. Sao chép toàn bộ thư mục `green_space_evaluator` vào thư mục chứa Plugin của QGIS theo đường dẫn tương ứng với hệ điều hành:
    - **Windows**:  
      `C:\Users\<Tên_người_dùng>\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\`  
      *(Mẹo mở nhanh trong QGIS: chọn **Settings** → **User Profiles** → **Open Active Profile Folder**, sau đó mở tiếp thư mục `python/plugins`)*
    - **Linux**:  
      `~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/`
    - **macOS**:  
      `~/Library/Application Support/QGIS/QGIS3/profiles/default/python/plugins/`
3. Khởi động lại phần mềm QGIS (hoặc mở lại hộp thoại **Manage and Install Plugins...**).
4. Tại mục **Installed**, tìm kiếm và tích chọn để kích hoạt Plugin.

---

## Kiểm tra sau khi cài đặt

Sau khi cài đặt và kích hoạt thành công, hãy kiểm tra nhanh theo danh sách sau:

- [x] **Plugin xuất hiện trong QGIS**: Biểu tượng chiếc lá hiển thị trên thanh công cụ và trong menu **Plugins**.
- [x] **Plugin có thể mở được**: Cửa sổ giao diện chính xuất hiện bình thường khi nhấn vào biểu tượng hoặc menu.
- [x] **Giao diện chính hiển thị đầy đủ**: Các tab chức năng (*Dữ liệu đầu vào*, *Sản phẩm đầu ra*, *Nhật ký tiến trình*) hiển thị đầy đủ.
- [x] **Các trường điều khiển sẵn sàng**: Các trường chọn dữ liệu đầu vào và các nút chức năng (*Cài đặt*, *Phân tích*) sẵn sàng thao tác.

# Cài đặt tham số

Trang này hướng dẫn chi tiết ý nghĩa khoa học, vai trò thuật toán và cách thiết lập các tham số trong cửa sổ **Cài đặt** của Plugin **Green Space Evaluator**.

<div class="guide-figure" markdown>
![Cửa sổ cài đặt nâng cao của Plugin](images/cai_dat_nang_cao.png)
<div class="guide-caption"><strong>Hình 2.</strong> Giao diện cấu hình tham số nâng cao trong Plugin</div>
</div>

---

## 1. Tổng quan bảng tham số

Người dùng có thể mở cửa sổ cấu hình bằng cách nhấn nút **Cài đặt** (biểu tượng bánh răng ở góc trên bên phải giao diện chính) và chọn tab **Tùy chỉnh nâng cao**:

| Nhóm | Tham số trên giao diện | Giá trị mặc định | Đơn vị | Ý nghĩa khoa học & Vai trò thuật toán |
|---|---|---:|:---:|---|
| **Chỉ số phổ** | **Ngưỡng MNDWI** | `0.0` | — | Giá trị mặc định được lựa chọn trong cấu hình nghiên cứu/thử nghiệm hiện tại. Pixel có $\text{MNDWI} > 0.0$ được xác định là nước và loại trừ trước khi tính SAVI. |
| **Chỉ số phổ** | **Hệ số hiệu chỉnh nền đất SAVI (L)** | `0.5` | — | Hệ số hiệu chỉnh nền đất L = 0.5, là giá trị mặc định trong cấu hình nghiên cứu/thử nghiệm hiện tại. |
| **Trích xuất mảng xanh** | **Phương thức xác định ngưỡng SAVI** | `Tự động xác định` | — | Lựa chọn giữa `Tự động xác định` (trích xuất từ vùng mẫu công viên) và `Nhập thủ công` (người dùng chỉ định ngưỡng cố định). |
| **Trích xuất mảng xanh** | **Phương pháp xác định ngưỡng SAVI** | `Bách phân vị P10 (Mặc định)` | — | Phương pháp mặc định trong cấu hình nghiên cứu/thử nghiệm hiện tại áp dụng trên tập mẫu công viên khi chọn chế độ tự động. |
| **Trích xuất mảng xanh** | **Bách phân vị tùy chỉnh P (%)** | `10.0` | `%` | Giá trị bách phân vị thiết lập khi chọn phương pháp "Bách phân vị tùy chỉnh". |
| **Trích xuất mảng xanh** | **Diện tích mảng xanh tối thiểu** | `5000.0` | m² | Giá trị cấu hình mặc định/tham chiếu trong nghiên cứu/thử nghiệm nhằm loại bỏ các mảng thực vật nhỏ lẻ ngoài công viên. Mảng xanh trong công viên luôn được bảo toàn. |
| **Vùng phục vụ** | **Bán kính phục vụ tối đa (Rmax)** | `300.0` | m | Giới hạn trên của miền tìm kiếm bán kính phục vụ $[0, R_{\max}]$ cho từng mảng xanh độc lập trong cấu hình nghiên cứu/thử nghiệm. |
| **Vùng phục vụ** | **Chỉ tiêu diện tích mảng xanh bình quân đầu người (C)** | `6.0` | m²/người | Chỉ tiêu diện tích bình quân dùng để xác định quy mô dân số mục tiêu: $P_{target,i} = S_i / C$. |
| **Thuật toán** | **Sai số hội tụ khi xác định bán kính** | `10.0` | m | Điều kiện dừng sai số khoảng cách của thuật toán tìm kiếm nhị phân (Binary Search). |

---

## 2. Tham số trích xuất mảng xanh và ngưỡng SAVI

### 2.1. Phương thức xác định ngưỡng SAVI

- **Chế độ tự động xác định (Mặc định)**: Plugin tự động lấy mẫu các ô pixel SAVI trên đất liền nằm trong phạm vi các polygon công viên/vườn hoa (`VEG_VECTOR`) và áp dụng phương pháp thống kê được chọn để tính ngưỡng phân tách thực vật khách quan.
- **Chế độ nhập thủ công**: Người dùng chủ động nhập trực tiếp một giá trị số thực cố định (ví dụ: `0.20` hoặc `0.25`) theo kinh nghiệm chuyên gia hoặc khảo sát thực địa mà không phụ thuộc vào lớp công viên mẫu.

### 2.2. Các phương pháp thống kê mẫu công viên

Khi chọn chế độ tự động, Plugin cung cấp 6 phương pháp tính ngưỡng:

1. **Bách phân vị P10 (Mặc định thử nghiệm)**: Lấy giá trị bách phân vị thứ 10 của phân bố SAVI trong tập mẫu công viên. Đây là phương pháp mặc định được lựa chọn trong cấu hình nghiên cứu/thử nghiệm hiện tại, giúp giữ lại hầu hết các dạng thảm thực vật công viên trong khi loại trừ các điểm dị biệt mặt lát.
2. **Mean - 1.0 * Std**: Ngưỡng bằng giá trị trung bình trừ 1 lần độ lệch chuẩn ($\mu - 1.0\sigma$).
3. **Mean - 0.5 * Std**: Ngưỡng bằng giá trị trung bình trừ 0.5 lần độ lệch chuẩn ($\mu - 0.5\sigma$).
4. **Median (Trung vị)**: Lấy giá trị trung vị P50 của phân bố SAVI trong công viên.
5. **Mean (Trung bình)**: Lấy giá trị trung bình cộng $\mu$ của phân bố SAVI.
6. **Bách phân vị tùy chỉnh**: Cho phép người dùng nhập trực tiếp bách phân vị $P$ bất kỳ từ `0%` đến `100%`.

---

## 3. Mô hình hóa vùng phục vụ mảng xanh

Vùng phục vụ của từng mảng xanh được mô hình hóa độc lập dựa trên diện tích mảng xanh, dân số được phân bổ theo không gian và khoảng cách Euclid.

### 3.1. Quy mô dân số mục tiêu ($P_{target,i}$)

Với mỗi mảng xanh độc lập $i$ có diện tích $S_i$, quy mô dân số mục tiêu được xác định từ diện tích mảng xanh và chỉ tiêu $C$ theo công thức:

$$
P_{target,i} = \frac{S_i}{C}
$$

Trong đó:
- $S_i$: Diện tích mảng xanh $i$ ($\text{m}^2$).
- $C$: **Chỉ tiêu diện tích mảng xanh bình quân đầu người** (mặc định tham chiếu $6.0\text{ m}^2/\text{người}$).
- $P_{target,i}$: Quy mô dân số mục tiêu được xác định từ diện tích mảng xanh và chỉ tiêu $C$ ($P_{target,i} = S_i / C$).

### 3.2. Giới hạn trên của miền tìm kiếm ($R_{\max}$)

- **$R_{\max}$ (Bán kính phục vụ tối đa)**: Đóng vai trò là **giới hạn trên của miền tìm kiếm** $[0, R_{\max}]$ trong thuật toán (mặc định tham chiếu $300\text{ m}$).
- Khoảng cách được mô hình hóa theo **khoảng cách hình học Euclid**, không đại diện cho cự ly đi bộ thực tế theo mạng lưới giao thông.
- $R_{\max}$ không phải là kết quả cố định hay quy chuẩn phổ quát cho mọi mảng xanh, mà là cận trên để khống chế không gian tìm kiếm phù hợp với phạm vi nghiên cứu.

### 3.3. Xác định bán kính phục vụ lớn nhất thỏa điều kiện ($R^*$)

Với mỗi mảng xanh $i$, bán kính phục vụ $R_i^*$ được xác định là **bán kính lớn nhất trong miền tìm kiếm $[0, R_{\max}]$ thỏa điều kiện tổng dân số phân bổ tiếp cận không vượt quá quy mô phục vụ mục tiêu**:

$$
R_i^* = \max \left\{ R \in [0, R_{\max}] \mid P_i(R) \le P_{target,i} \right\}
$$

Trong đó $P_i(R)$ là tổng dân số được phân bổ (`POP_ALLOC`) nằm trong phạm vi khoảng cách hình học Euclid $\le R$ tính từ biên của mảng xanh $i$.

### 3.4. Thuật toán tìm kiếm và 3 trạng thái kết quả

Thuật toán kiểm tra điều kiện và áp dụng tìm kiếm nhị phân (**Binary Search**) khi cần thiết:

1. **Trường hợp `NO_SOLUTION` ($R^* = 0\text{ m}$)**:
   - Xảy ra khi ngay tại khoảng cách $R = 0\text{ m}$, tổng dân số được phân bổ nằm trong phạm vi của mảng xanh đã lớn hơn quy mô mục tiêu:
     $$P_i(0) > P_{target,i}$$
   - Khi đó, không tồn tại bán kính thỏa điều kiện trong miền tìm kiếm $[0, R_{\max}]$. Bán kính phục vụ được gán bằng $0\text{ m}$. Trạng thái này phản ánh mối quan hệ giữa diện tích mảng xanh, dân số được phân bổ và phân bố không gian xây dựng quanh mảng xanh.
2. **Trường hợp `RMAX` ($R^* = R_{\max}$)**:
   - Nếu tại giới hạn trên $R = R_{\max}$ vẫn thỏa điều kiện $P_i(R_{\max}) \le P_{target,i}$, thuật toán gán bán kính phục vụ $R^* = R_{\max}$ và trạng thái kết quả là RMAX.
3. **Trường hợp `BINARY_SEARCH`**:
   - Khi $P_i(0) \le P_{target,i}$ nhưng $P_i(R_{\max}) > P_{target,i}$, thuật toán tìm kiếm nhị phân sẽ thu hẹp dần khoảng cách $[R_{left}, R_{right}]$ cho đến khi đạt sai số hội tụ (mặc định $10\text{ m}$). Kết quả trả về bán kính phục vụ lớn nhất thỏa điều kiện.

!!! note "Bản chất khái niệm bán kính phục vụ R*"
    Trong tài liệu và báo cáo khoa học, $R^*$ được gọi chuẩn xác là **"bán kính phục vụ lớn nhất thỏa điều kiện"** hoặc **"bán kính phục vụ lớn nhất trong miền tìm kiếm thỏa điều kiện của mô hình"**.

---

## 4. Tham số kỹ thuật khác

- **Ngưỡng phân tách nước MNDWI (`0.0`)**: Giá trị mặc định được lựa chọn trong cấu hình nghiên cứu/thử nghiệm hiện tại. Các pixel có chỉ số $\text{MNDWI} > 0.0$ được xác định là mặt nước và được loại trừ hoàn toàn trước khi tính chỉ số SAVI.
- **Hệ số SAVI $L$ (`0.5`)**: Hệ số hiệu chỉnh nền đất L = 0.5, là giá trị mặc định trong cấu hình nghiên cứu/thử nghiệm hiện tại.
- **Diện tích mảng xanh tối thiểu (`5000 m²`)**: Giá trị cấu hình mặc định/tham chiếu trong nghiên cứu áp dụng cho các mảng xanh độc lập **ngoài công viên** (tương đương 50 pixel $10\text{m} \times 10\text{m}$) nhằm lọc bỏ cây xanh vườn nhà nhỏ lẻ, dải phân cách hẹp hoặc bóng cây trên đường. Toàn bộ mảng xanh nằm trong ranh công viên luôn được bảo toàn nguyên vẹn.
- **Sai số hội tụ (Tolerance = `10 m`)**: Điều kiện dừng của Binary Search, đồng bộ với độ phân giải lưới 10 m của ảnh viễn thám Sentinel-2.

---

## 5. Khi nào nên thay đổi tham số?

### 🟢 Có thể chủ động thay đổi
- **Bán kính phục vụ tối đa ($R_{\max}$)**: Khi cần mở rộng hoặc thu hẹp miền tìm kiếm phù hợp với cự ly nghiên cứu của đề tài.
- **Chỉ tiêu diện tích mảng xanh bình quân đầu người ($C$)**: Khi áp dụng các chỉ tiêu quy chuẩn khác nhau (ví dụ: $4.0$, $6.0$, $8.0\text{ m}^2/\text{người}$) hoặc kịch bản nghiên cứu so sánh.
- **Diện tích lọc mảng xanh tối thiểu**: Khi cần điều chỉnh quy mô mảng xanh ngoài công viên phù hợp với hiện trạng khu vực.

### 🟡 Nên cân nhắc kỹ
- **Phương pháp thống kê SAVI**: Khi kết quả trích xuất tự động theo `P10` có dấu hiệu thiếu thực vật hoặc thừa thực vật do đặc thù công viên mẫu.
- **Bách phân vị tùy chỉnh**: Khi muốn thử nghiệm các mức ngưỡng phân tách chặt chẽ hơn ($P_{15}$, $P_{20}$) hoặc bao quát hơn ($P_5$).

### 🔵 Nên giữ mặc định
- **Ngưỡng MNDWI (`0.0`)**: Giá trị mặc định được lựa chọn trong cấu hình nghiên cứu/thử nghiệm hiện tại.
- **Hệ số $L$ (`0.5`)**: Hệ số hiệu chỉnh nền đất L = 0.5, là giá trị mặc định trong cấu hình nghiên cứu/thử nghiệm hiện tại.
- **Sai số hội tụ (`10 m`)**: Tương ứng với kích thước 1 pixel 10 m của ảnh Sentinel-2.

---

## 6. Tính lan truyền của tham số

!!! warning "Lưu ý tính liên hoàn giữa các module"
    Các tham số trong Plugin có mối liên hệ chuỗi chặt chẽ:
    1. Thay đổi ngưỡng **MNDWI** hoặc **SAVI** sẽ thay đổi diện tích mảng xanh $S_i$.
    2. Diện tích $S_i$ thay đổi làm thay đổi quy mô dân số mục tiêu $P_{target,i} = S_i / C$, từ đó trực tiếp làm thay đổi bán kính $R_i^*$.
    3. Bán kính $R^*$ thay đổi sẽ làm thay đổi kết quả phân loại không gian xây dựng (trong/ngoài vùng phục vụ) và toàn bộ các chỉ tiêu thống kê hành chính ở Module 5.

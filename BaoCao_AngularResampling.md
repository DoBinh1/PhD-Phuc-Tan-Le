# BÁO CÁO BÀI THỰC HÀNH

## Lấy mẫu lại trong miền góc và ứng dụng chẩn đoán hư hỏng bánh răng

| | |
|---|---|
| Lớp / Nhóm | ITD-Lab — Nhóm NCS |
| Học viên | *(điền tên)* |
| Giảng viên hướng dẫn | TS. Trọng Du |
| Bài thực hành | Angular Resampling và Pipeline Chẩn đoán Bánh răng |
| Ngày nộp | 29/04/2026 |
| Mã nguồn tham khảo | Matania O. *et al.*, *“Signal Processing for the Condition-Based Maintenance of Rotating Machines via Vibration Analysis: A Tutorial”*, **MDPI Sensors** 24(2), 454 (2024). |

---

## 1. Đặt vấn đề và mục tiêu của bài thực hành

### 1.1 Bối cảnh kỹ thuật

Trong các hệ thống cơ khí truyền động bằng bánh răng, sự xuống cấp của răng do mỏi tiếp xúc, mòn bề mặt hay nứt gãy cục bộ là nguyên nhân hàng đầu dẫn đến hỏng hóc đột ngột và làm gián đoạn vận hành dây chuyền sản xuất. Việc phát hiện sớm những bất thường ở cấp độ vi mô — khi hư hỏng mới ở giai đoạn ủ và chưa biểu hiện rõ trên các đại lượng vận hành thông thường như nhiệt độ, dòng điện hay tải động cơ — là mục tiêu cốt lõi của bảo trì dựa trên trạng thái (Condition-Based Maintenance, CBM). Trong cách tiếp cận này, tín hiệu rung động đo trên vỏ hộp số đóng vai trò một dạng "tín hiệu sinh học" của máy: nó mang theo các dấu hiệu rất sớm về hư hỏng nội tại, song đồng thời cũng bị che phủ bởi vô số thành phần không mang thông tin chẩn đoán như rung nền của các trục lân cận, nhiễu của cảm biến, hay sự biến thiên tốc độ trục theo điều kiện tải.

### 1.2 Mục tiêu cụ thể

Bài thực hành này được thiết kế nhằm cung cấp một trải nghiệm thực tế với pipeline xử lý tín hiệu cổ điển nhưng vẫn giữ nguyên giá trị tham khảo cho ứng dụng công nghiệp hiện đại, bao gồm bốn khối nối tiếp:

- **Lấy mẫu lại trong miền góc (angular resampling)** — đưa tín hiệu không dừng về tín hiệu dừng theo nghĩa thống kê.
- **Trung bình đồng bộ (synchronous averaging)** — triệt tiêu các thành phần phi đồng bộ với trục.
- **Trích xuất tín hiệu hiệu (difference signal)** — loại bỏ rung "bình thường" của ăn khớp răng để cô lập bất thường cục bộ.
- **Xây dựng chỉ số sức khỏe tổng hợp (Health Indicator)** — thu gọn thông tin chẩn đoán về một đại lượng duy nhất.

Cụ thể, học viên triển khai và phân tích hai chương trình MATLAB do nhóm tác giả Matania và cộng sự (2024) công bố — `demo_angular_resampling.m` minh họa cô đọng nguyên lý chuyển miền, và `demo_gear_diagnosis.m` thực thi pipeline chẩn đoán đầy đủ trên một bộ dữ liệu thực được tổ chức theo kịch bản run-to-failure. Mục tiêu của bài không chỉ dừng lại ở việc đạt được kết quả phân biệt đúng giữa tín hiệu lành và tín hiệu lỗi, mà còn ở việc làm rõ ý nghĩa vật lý đằng sau từng phép biến đổi và đánh giá phạm vi áp dụng cũng như giới hạn của phương pháp.

---

## 2. Tổ chức mã nguồn và quy trình triển khai

### 2.1 Cấu trúc tệp và vai trò của từng thành phần

Pipeline chẩn đoán được tổ chức xoay quanh script `demo_gear_diagnosis.m` đóng vai trò bộ điều phối, tuần tự gọi ba hàm con — `angular_resampling.m`, `calc_sa.m` và `calc_difference_signal.m` — vốn đảm nhiệm ba phép biến đổi tín hiệu cốt lõi của pipeline. Sự phân tách mạch lạc giữa script điều phối và các hàm xử lý phản ánh một thực hành kỹ nghệ phần mềm hợp lý, bởi mỗi hàm con có thể được kiểm thử, hiệu chỉnh hay thay thế một cách độc lập mà không ảnh hưởng đến phần còn lại của pipeline.

### 2.2 Tổ chức của tệp dữ liệu .mat

Toàn bộ trạng thái dữ liệu của bài toán được nén trong một tệp duy nhất `demo_gear_diagnosis.mat`, chứa năm biến:

- `sigs_healthy_t` — ma trận tín hiệu rung của các bản ghi lành (mỗi cột là một bản ghi).
- `sigs_faulty_t` — ma trận tín hiệu rung của các bản ghi lỗi, tổ chức theo cùng quy ước.
- `speed_healthy`, `speed_faulty` — hai ma trận véc-tơ tốc độ tức thời tương ứng (đơn vị: vòng/giây).
- `dt` — bước thời gian lấy mẫu chung cho toàn bộ dữ liệu.

Cấu trúc lưu trữ với mỗi cột tương ứng một bản ghi cho phép script duyệt qua từng tín hiệu một cách có hệ thống thông qua `size(., 2)`, và việc đóng gói toàn bộ dữ liệu trong một tệp nhị phân duy nhất phản ánh một thực hành nghiên cứu có lợi cho khả năng tái lập kết quả. Vai trò của tệp `.mat` không thuần túy là cung cấp số liệu đầu vào — quan trọng hơn, nó cung cấp đồng thời tín hiệu rung và tín hiệu tốc độ tham chiếu, yếu tố tiên quyết để bước angular resampling có thể vận hành. Trong thực tế công nghiệp, tín hiệu tốc độ này thường được đo bằng cảm biến tachometer hoặc encoder gắn trực tiếp trên trục, và việc bộ dữ liệu mẫu mô phỏng đầy đủ cả hai kênh là điều khiến nó phù hợp cho mục đích tutorial.

### 2.3 Luồng thực thi tổng thể

Khi script chính được khởi chạy, dữ liệu được nạp một lần duy nhất, sau đó hai vòng lặp lớn được thực thi nối tiếp. Vòng lặp thứ nhất duyệt qua các bản ghi lành nhằm hai mục đích đồng thời: trích xuất véc-tơ đặc trưng bốn chiều của từng bản ghi, và thông qua đó ước lượng các tham số thống kê tham chiếu — cụ thể là véc-tơ trung bình $\boldsymbol{\mu}$ và véc-tơ độ lệch chuẩn $\boldsymbol{\sigma}$ của không gian đặc trưng. Vòng lặp thứ hai duyệt qua các bản ghi lỗi để tính chỉ số sức khỏe HI của từng bản ghi dựa trên các tham số ước lượng ấy. Điểm đáng lưu ý về mặt phương pháp luận là toàn bộ "kiến thức về hư hỏng" trong pipeline này không đến từ nhãn lỗi mà chỉ đến từ mô hình thống kê của tập lành. Đây thực chất là một dạng phát hiện bất thường (anomaly detection) thuần nhất, một cách tiếp cận đặc biệt phù hợp với thực tiễn công nghiệp nơi dữ liệu lỗi gắn nhãn thường khan hiếm hoặc không tồn tại.

---

## 3. Lấy mẫu lại trong miền góc — bản chất và phân tích triển khai

### 3.1 Hiện tượng spectral smearing và động cơ của phương pháp

Khi máy quay vận hành trong điều kiện tốc độ biến thiên — dù chỉ là biến thiên nhỏ do dao động tải hay biến thiên lớn trong các pha khởi động và dừng — tín hiệu rung động trở thành một quá trình ngẫu nhiên không dừng theo nghĩa thống kê. Mỗi thành phần điều hòa của trục, vốn có dạng $A\cos(2\pi k\,\varphi(t))$ với $\varphi(t)=\int_0^t\omega(\tau)\,d\tau$ là pha tích lũy của trục, biểu hiện trên trục thời gian như một tín hiệu chirp với tần số tức thời tỉ lệ thuận với vận tốc góc tức thời. Hệ quả là khi áp biến đổi Fourier rời rạc lên cửa sổ tín hiệu, năng lượng của thành phần đó không tập trung tại một tần số duy nhất mà bị trải dọc theo toàn bộ dải tần mà tốc độ trục đã đi qua trong cửa sổ quan sát — hiện tượng được gọi là *spectral smearing*. Trên một phổ bị trải như vậy, các đỉnh đặc trưng cho hư hỏng cục bộ — vốn dĩ đã yếu hơn nhiều so với các thành phần ăn khớp răng — gần như mất khả năng phát hiện vì biên độ phổ của chúng bị "loãng" trên một dải rộng.

### 3.2 Tham số hóa lại theo góc — ý tưởng cốt lõi

Lấy mẫu lại trong miền góc, về bản chất, là một phép tham số hóa lại tín hiệu, thay tham số "thời gian" bằng tham số "góc quay tích lũy". Một cách trực quan, thay vì lấy mẫu cách đều mỗi $\Delta t$ giây như khi sử dụng cảm biến rung tiêu chuẩn, ta lấy mẫu cách đều mỗi $\Delta\varphi$ vòng quay. Nhờ phép biến đổi này, các thành phần điều hòa khoá pha với trục — vốn là các chirp trên trục thời gian — trở thành các sinusoid hoàn hảo trên trục góc, tức tín hiệu được "khôi phục" tính dừng. Phép biến đổi Fourier áp lên tín hiệu trong miền góc cho ra cái thường được gọi là *phổ bậc* (order spectrum), trong đó trục hoành đo bằng số chu kỳ trên một vòng quay và độc lập với RPM. Trên phổ bậc, mọi đỉnh đặc trưng cho cấu trúc cơ khí xuất hiện ở vị trí cố định bất kể máy đang chạy ở tốc độ nào: đỉnh ở bậc một biểu thị mất cân bằng trục, đỉnh ở bậc bằng số răng biểu thị tần số ăn khớp, các đỉnh phụ (sidebands) hai bên biểu thị hiện tượng điều biến biên độ. Đây chính là điều kiện tiên quyết để mọi phép phân tích phổ tiếp theo trở nên có ý nghĩa — đặc biệt là các bước trung bình đồng bộ và lọc bỏ thành phần nền vốn yêu cầu nghiêm ngặt rằng tín hiệu phải có chu kỳ chính xác trong miền xử lý.

### 3.3 Phân tích triển khai trong `angular_resampling.m`

Đoạn mã cốt lõi của hàm `angular_resampling.m` được trình bày dưới đây cùng với phân tích tương ứng:

```matlab
function [sig_cyc, cyc_fs, sample_pnts] = angular_resampling(t, speed, sig_t)

dt = t(2) - t(1);                                                 % (1)
cumulative_phase = cumsum(speed*dt);                              % (2)
cumulative_phase = cumulative_phase - cumulative_phase(1);        % (3)

cyc_fs = ceil(1/dt/min(speed));                                   % (4)

constant_phase_intervals = linspace(0, max(cumulative_phase), ...
                  round(cyc_fs*max(cumulative_phase)))';          % (5)
times_of_constant_phase_intervals = ...
    interp1(cumulative_phase, t, constant_phase_intervals, 'linear');  % (6)

sig_cyc = interp1(t, sig_t, times_of_constant_phase_intervals, 'spline'); % (7)
```

Dòng **(1)** xác định bước thời gian từ chính véc-tơ thời gian đầu vào, ngầm định rằng lưới thời gian là đều — một giả thiết hoàn toàn hợp lý với dữ liệu từ ADC chuẩn. Dòng **(2)** thực hiện phép tích phân số của tốc độ để thu được pha tích lũy, với `cumsum(speed*dt)` đóng vai trò xấp xỉ Riemann tiến của $\int\omega\,d\tau$; do `speed` được biểu diễn theo đơn vị vòng/giây, kết quả pha tích lũy có đơn vị tự nhiên là *vòng*, một lựa chọn thuận tiện cho mọi phép tính tiếp theo. Dòng **(3)** chuẩn hóa pha tích lũy về 0 ở điểm khởi đầu — một thao tác không ảnh hưởng đến kết quả cuối cùng nhưng giúp đơn giản hóa việc xây dựng lưới góc đều ở bước sau.

Dòng **(4)** thiết lập tần số lấy mẫu trong miền góc theo công thức $F_{cyc}=\lceil 1/(\Delta t\cdot\omega_{min})\rceil$. Việc chọn $\omega_{min}$ làm vận tốc tham chiếu xuất phát từ một quan sát quan trọng về điều kiện Nyquist: tại điểm tốc độ thấp nhất trong cửa sổ quan sát, tỉ số "số mẫu thời gian trên một vòng quay" đạt giá trị lớn nhất, do vậy nếu sử dụng tỉ số này làm số mẫu/vòng đầu ra thì toàn bộ tín hiệu trong miền góc — kể cả tại các đoạn tốc độ cao hơn — đều thỏa điều kiện Nyquist mà không cần ngoại suy. Dòng **(5)** sinh ra lưới góc cách đều với số điểm phù hợp với $F_{cyc}$ và phạm vi pha tổng cộng đã đi qua.

Hai dòng **(6)** và **(7)** thực hiện phép chuyển miền thực sự thông qua hai phép nội suy nối tiếp nhau:

- Dòng **(6)** sử dụng `interp1` với phương pháp `'linear'` để thực hiện *nội suy ngược* trên ánh xạ $\varphi\mapsto t$. Vì $\varphi(t)$ là một hàm đơn điệu tăng (do $\omega>0$), việc đảo vai trò đối số và giá trị của `interp1` cho phép tìm được thời điểm $t_m$ tương ứng với từng góc $\varphi_m$ trên lưới góc đều.
- Dòng **(7)** lấy mẫu tín hiệu rung gốc tại các thời điểm $t_m$ vừa tìm được, sử dụng nội suy `'spline'`. Lựa chọn spline thay vì linear ở bước này không phải là tùy tiện: spline bảo toàn tốt hơn nhiều các thành phần tần số cao của tín hiệu, một yếu tố đặc biệt quan trọng vì tín hiệu rung của hộp số mang theo các xung va đập có phổ rộng do hư hỏng cục bộ — mất các thành phần tần số cao này đồng nghĩa với mất luôn các dấu hiệu chẩn đoán nhạy nhất.

Một quan sát đáng lưu ý là hàm này hoạt động theo mô hình hai bước nội suy "linear-then-spline" — linear cho ánh xạ pha-thời gian (bản chất là đơn điệu, ít rủi ro overshoot) và spline cho lấy mẫu tín hiệu rung (cần độ chính xác cao). Phân biệt này phản ánh một nguyên tắc tổng quát trong xử lý tín hiệu: chọn phương pháp nội suy theo đặc tính của hàm cần nội suy, không theo "thói quen mặc định".

---

## 4. Các khối xử lý bổ trợ và xây dựng chỉ số sức khỏe

### 4.1 Trung bình đồng bộ — `calc_sa.m`

Sau khi tín hiệu đã được chuyển sang miền góc đều, hàm `calc_sa.m` thực thi phép trung bình đồng bộ với một thiết kế ngắn gọn đến mức tối giản:

```matlab
function [sa] = calc_sa(sig_cyc, sa_len)

num_sgmnts = floor(length(sig_cyc)/sa_len);
sig_cyc = sig_cyc(1:num_sgmnts*sa_len);
sigs_mtrx = reshape(sig_cyc, sa_len, num_sgmnts);

sa = mean(sigs_mtrx, 2);
```

Hàm này trước tiên xác định số đoạn nguyên dài đúng `sa_len` mẫu (tương ứng đúng một vòng quay sau angular resampling) chứa được trong tín hiệu đầu vào, cắt bỏ phần đuôi không đủ một vòng, rồi `reshape` phần còn lại thành một ma trận hai chiều mà mỗi cột là một vòng. Phép `mean(..., 2)` trên chiều thứ hai thực chất là cộng theo từng vị trí mẫu trong vòng và lấy trung bình qua tất cả các vòng.

Về mặt xử lý tín hiệu, phép trung bình này hoạt động như một *bộ lọc lược (comb filter) lý tưởng* trong miền tần số bậc: các thành phần có tần số bậc nguyên — tức các thành phần đồng bộ với trục — được giữ nguyên, trong khi các thành phần phi đồng bộ như rung của các trục khác trong hộp số, nhiễu trắng, hay các tần số ngoại lai bị triệt tiêu với tốc độ tỉ lệ với căn bậc hai số vòng tham gia trung bình. Khi số vòng đủ lớn, tín hiệu thu được phản ánh gần như thuần khiết hành vi rung của riêng trục đang quan tâm. Cần lưu ý rằng tính đúng đắn của phép này phụ thuộc hoàn toàn vào việc bước angular resampling đã đảm bảo `sa_len` là chính xác số mẫu trên một vòng — đây chính là lý do tại sao hai khối phải đi đôi với nhau và không thể tách rời.

### 4.2 Tín hiệu hiệu — `calc_difference_signal.m`

Trên tín hiệu trung bình đồng bộ, năng lượng vẫn bị thống trị bởi các đỉnh ăn khớp răng (gear mesh) tại bậc bằng số răng và các sidebands xung quanh — đây là rung "bình thường", có biên độ lớn nhưng kém nhạy với khuyết tật cục bộ. Hàm `calc_difference_signal.m` xử lý vấn đề này theo nguyên tắc của Stewart (1977):

```matlab
function difference_sig = calc_difference_signal(sa, gear_mesh, num_sidebands)

sa_len    = length(sa);
max_order = floor(sa_len/2);

orders_2_remove = [];
num_of_gear_mesh_harmonics = floor(max_order./gear_mesh);
for ii = 1:num_of_gear_mesh_harmonics
    gear_mesh_harmonic_order = gear_mesh*ii;
    orders_2_remove = [orders_2_remove, ...
        (gear_mesh_harmonic_order-num_sidebands):(gear_mesh_harmonic_order+num_sidebands)];
end
orders_2_remove(orders_2_remove > max_order) = [];

orders_2_remove_positive_inds = orders_2_remove + 1;
orders_2_remove_negative_inds = sa_len - orders_2_remove_positive_inds + 2;
orders_2_remove_inds = sort([orders_2_remove_positive_inds, orders_2_remove_negative_inds]);

sa_order = fft(sa, sa_len);
sa_order(orders_2_remove_inds) = 0;
difference_sig = real(ifft(sa_order, sa_len));
```

Logic của hàm có thể được phân thành ba giai đoạn rõ rệt. Giai đoạn đầu (vòng `for`) xây dựng tập các bậc cần loại $\{lz-K,\dots,lz+K\}$ với $l=1,2,\dots$ là chỉ số hài, $z$ là số răng (tham số `gear_mesh`) và $K$ là số sidebands cần loại — hợp với nhau qua mọi hài tạo thành một dải các bậc xung quanh mỗi bội của tần số ăn khớp răng. Phép cắt `orders_2_remove > max_order = []` đảm bảo không vượt quá giới hạn Nyquist trong miền bậc.

Giai đoạn thứ hai xử lý một chi tiết kỹ thuật tinh tế nhưng *trọng yếu*: do FFT của tín hiệu thực có tính đối xứng liên hợp $X[N-k]=X^*[k]$, các bậc được loại bỏ phải xuất hiện đối xứng ở cả hai nửa phổ — nửa "dương" và nửa "âm" theo quy ước một chiều của FFT. Hai dòng tính `..._positive_inds` và `..._negative_inds` chính là làm việc đó: với MATLAB indexing 1-based, bậc $k$ ở nửa dương ứng với chỉ số $k+1$, và bản đối xứng ở nửa âm ứng với chỉ số $N-k+2$. Nếu chỉ xóa một nửa, IFFT sẽ cho ra tín hiệu phức và phá vỡ ý nghĩa vật lý.

Giai đoạn thứ ba thực hiện phép biến đổi: `fft → set 0 → ifft → real`. Lệnh `real(...)` ở cuối là một biện pháp an toàn để loại bỏ phần ảo cỡ $10^{-15}$ phát sinh từ sai số làm tròn dấu chấm động, không phải để "ép" một tín hiệu phức về thực — nếu xóa đối xứng đã đúng, phần ảo lý thuyết phải đúng bằng 0.

Tín hiệu hiệu thu được do đó đóng vai trò như một "kính lúp" tập trung vào phần năng lượng còn lại sau khi đã loại bỏ rung nền, và chính phần năng lượng này — dù nhỏ — lại nhạy bén bậc nhất với các xung va đập do răng nứt hoặc mẻ.

### 4.3 Trích đặc trưng và xây dựng HI trong `demo_gear_diagnosis.m`

Để thu gọn thông tin chẩn đoán thành một đại lượng đơn dễ giám sát, script chính trích xuất bốn đặc trưng cho mỗi bản ghi và tổ hợp chúng theo công thức:

```matlab
sa_rms              = rms(sa);
sa_kurtosis         = kurtosis(sa);
difference_rms      = rms(difference_sig);
difference_skewness = skewness(difference_sig);
sig_features = [sa_rms, sa_kurtosis, difference_rms, difference_skewness].';
```

Bốn đại lượng này không độc lập một cách ngẫu nhiên mà được lựa chọn theo một ý đồ rõ ràng:

| Đặc trưng | Tín hiệu nguồn | Ý nghĩa cơ học |
|---|---|---|
| `rms(sa)` | Trung bình đồng bộ | Năng lượng tổng thể của rung đồng bộ với trục |
| `kurtosis(sa)` | Trung bình đồng bộ | Mức độ "đuôi nặng" — nhạy với xung va đập cục bộ |
| `rms(difference_sig)` | Tín hiệu hiệu | Năng lượng còn lại sau khi loại gear mesh |
| `skewness(difference_sig)` | Tín hiệu hiệu | Tính bất đối xứng — nhạy với xung một chiều |

Hai đặc trưng đầu mô tả năng lượng và hình dạng phân bố ở cấp độ tổng thể của tín hiệu trung bình đồng bộ, hai đặc trưng sau cô lập các bất thường cục bộ trên tín hiệu hiệu vốn dễ bị che lấp khi nhìn trên tín hiệu thô. Sau khi đã có véc-tơ đặc trưng cho mọi bản ghi lành, $\boldsymbol{\mu}$ và $\boldsymbol{\sigma}$ được ước lượng theo từng chiều, và chỉ số sức khỏe của một bản ghi bất kỳ được tính theo công thức:

$$\mathrm{HI} = \frac{1}{4}\sum_{i=1}^{4}\frac{|f_i-\mu_i|}{\sigma_i}.$$

Triển khai MATLAB tương ứng:

```matlab
hi = mean(abs(sig_features - healthy_features_average) ./ healthy_features_std);
```

Về bản chất, công thức này là một biến thể đơn giản hóa của khoảng cách Mahalanobis với giả thiết ma trận hiệp phương sai chéo — mỗi chiều được chuẩn hóa độc lập theo độ phân tán của riêng nó, sau đó các chiều được tổng hợp bằng trung bình cộng (thay vì tổng bình phương như Mahalanobis chuẩn). Một giá trị HI nhỏ phản ánh tín hiệu thuộc về phân phối lành; HI lớn chỉ thị mức độ lệch khỏi phân phối lành theo nhiều chiều đặc trưng đồng thời, qua đó cảnh báo bất thường.

---

## 5. Phân tích kết quả thực nghiệm

### 5.1 Khảo sát nguyên lý qua `demo_angular_resampling.m`

Việc khảo sát trước tiên `demo_angular_resampling.m` trên một tín hiệu chirp tổng hợp với tốc độ tăng tuyến tính từ 10 đến 30 vòng/giây trong khoảng thời gian 3 giây cho phép quan sát trực quan và tách bạch hiệu ứng của phép chuyển miền.

![Kết quả `demo_angular_resampling.m`: tín hiệu trên trục thời gian (trên trái), phổ tần số (trên phải), tín hiệu trên trục góc (dưới trái) và phổ bậc (dưới phải).](demo_angular_resampling.png)

Trên trục thời gian, tín hiệu thể hiện đặc trưng chirp điển hình với mật độ dao động tăng dần theo thời gian; phổ Fourier tương ứng không có đỉnh sắc mà trải thành một dải năng lượng phân bố tương đối đều trong khoảng 10–30 Hz, hoàn toàn trùng khớp với dự đoán lý thuyết về spectral smearing. Sau khi áp angular resampling, tín hiệu trong miền vòng quay biểu hiện mật độ dao động đều trên toàn bộ trục góc — tức đã trở thành tín hiệu dừng — và phổ bậc tương ứng cô đặc lại thành duy nhất một đỉnh sắc nét tại bậc 1, phản ánh đúng bản chất thực sự của tín hiệu là một sinusoid khoá pha với trục.

Sự tương phản trực tiếp giữa hai phổ — một phổ trải rộng trên trục tần số và một phổ tập trung trên trục bậc — chính là minh chứng thực nghiệm thuyết phục nhất cho sự cần thiết của bước tiền xử lý này trong mọi pipeline chẩn đoán có liên quan đến tốc độ biến thiên. Đáng lưu ý là phổ tần số ở panel trên-phải không phải là một dải hoàn toàn phẳng mà có một số dao động nhẹ ở hai biên — điều này phản ánh rằng tốc độ không thực sự tăng đều một cách "lý tưởng" ở các điểm đầu và cuối cửa sổ, song hiệu ứng đó hoàn toàn biến mất sau khi chuyển sang miền góc, khẳng định rằng angular resampling không chỉ làm "gọn" phổ mà còn loại bỏ được các artifacts bậc cao do biến thiên tốc độ gây ra.

### 5.2 Pipeline đầy đủ trên dữ liệu thực qua `demo_gear_diagnosis.m`

Khi áp pipeline đầy đủ lên dữ liệu thực, chỉ số sức khỏe được tính cho 100 bản ghi lành và 100 bản ghi lỗi liên tiếp, và kết quả được hiển thị trên cùng một đồ thị.

![Chỉ số sức khỏe HI của tập lành (xanh lục, các bản ghi 1–100) và tập lỗi (đỏ, các bản ghi 101–200).](demo_gear_diagnosis.png)

Đường HI của tập lành dao động hẹp xung quanh giá trị trung bình xấp xỉ 1.0, với độ lệch chuẩn ước lượng vào khoảng 0.25. Tính nhất quán này không chỉ là một quan sát mô tả mà còn là một bằng chứng thực nghiệm có ý nghĩa phương pháp luận: nó cho thấy bốn đặc trưng được lựa chọn thực sự ổn định khi máy vận hành ở chế độ bình thường, qua đó các tham số $\boldsymbol{\mu}$ và $\boldsymbol{\sigma}$ ước lượng từ chính tập lành đủ chính xác để đóng vai trò chuẩn mực tham chiếu cho các phép so sánh tiếp theo. Trên tập lỗi, đường HI biểu hiện một quỹ đạo đặc trưng của kịch bản run-to-failure có thể chia thành ba pha rõ rệt:

- **Pha tiềm ẩn (bản ghi 101–140):** HI vẫn dao động trong dải phân phối của tập lành, tương ứng với giai đoạn hư hỏng còn nhẹ và rung động chưa kịp biểu hiện rõ rệt trên bốn đặc trưng được giám sát.
- **Pha chuyển tiếp (bản ghi 140–170):** đường đỏ tách dần khỏi đường xanh và HI tăng đều đặn lên giá trị xấp xỉ 10 — đây là vùng *phát hiện được* bởi bất kỳ ngưỡng cảnh báo hợp lý nào.
- **Pha bùng nổ (bản ghi 170–200):** HI tăng theo dạng gần như hàm mũ, đẩy giá trị lên đến cực đại khoảng 53 ở bản ghi cuối cùng.

Tỉ lệ giữa HI cực đại của tập lỗi và HI trung bình của tập lành đạt khoảng 50:1 — một biên phân biệt đặc biệt rộng, đủ để áp dụng các ngưỡng cảnh báo đơn giản, chẳng hạn $\mathrm{HI}_{th}=\mu+4\sigma\approx 2$, mà vẫn duy trì được độ nhạy phát hiện cao đồng thời tỉ lệ báo động giả thấp.

> **Lưu ý:** các giá trị $\mu\approx 1.0$ và $\sigma\approx 0.25$ là ước lượng đọc trực tiếp từ đồ thị. Để có giá trị chính xác từ workspace MATLAB sau khi chạy script, có thể bổ sung đoạn:
>
> ```matlab
> fprintf('Healthy: mean=%.4f, std=%.4f\n', ...
>         mean(healthy_hi_vctr), std(healthy_hi_vctr));
> fprintf('Faulty : mean=%.4f, std=%.4f, max=%.4f\n', ...
>         mean(hi_faulty_vctr), std(hi_faulty_vctr), max(hi_faulty_vctr));
> ```

### 5.3 Vai trò tương đối của bốn đặc trưng

Phân rã đóng góp của từng đặc trưng vào HI cho thấy bốn đại lượng bổ sung lẫn nhau theo những cách có thể giải thích được về mặt cơ học, và mỗi đặc trưng có một "vùng nhạy" riêng dọc theo quỹ đạo run-to-failure. Giá trị hiệu dụng `sa_rms` đo năng lượng tổng thể của rung động đồng bộ và do đó tăng đều cùng với mức độ hư hỏng, đóng vai trò chính trong giai đoạn nặng khi biên độ rung tăng mạnh. Chỉ số `sa_kurtosis`, vốn nhạy với các xung va đập cục bộ vì kurtosis phản ánh phần "đuôi" của phân bố biên độ, có xu hướng tăng sớm hơn rms ngay từ giai đoạn chuyển tiếp khi răng mẻ bắt đầu tạo ra các xung rời rạc trên nền rung trơn. Đại lượng `difference_rms` cô lập phần năng lượng còn lại sau khi loại các thành phần ăn khớp răng và do đó khuếch đại các bất thường vốn bị che lấp dưới năng lượng nền lớn của gear mesh. Cuối cùng, `difference_skewness` đo tính bất đối xứng của tín hiệu hiệu — một đặc trưng đặc biệt có ý nghĩa vì xung va đập do răng mẻ thường có tính một chiều theo hướng chuyển động, làm phân bố tín hiệu lệch khỏi tính đối xứng quanh giá trị 0.

Sự kết hợp giữa hai loại đại lượng "định lượng" (rms) đo độ lớn và "định tính" (kurtosis, skewness) đo hình dạng phân bố đảm bảo rằng HI vừa nhạy với *mức độ* hư hỏng vừa nhạy với *kiểu* hư hỏng — một đặc tính khó đạt được với bất kỳ một đặc trưng đơn lẻ nào, và là lý do tại sao biên phân biệt 50:1 có thể đạt được mà không cần đến các kỹ thuật học máy phức tạp.

---

## 6. Thảo luận và đánh giá phương pháp

### 6.1 Ưu điểm

Kết quả thu được khẳng định tính đúng đắn về mặt giả thiết và tính khả thi về mặt thực tiễn của pipeline chẩn đoán dựa trên xử lý tín hiệu cổ điển. Đáng chú ý hơn cả là việc pipeline phát hiện được giai đoạn chuyển tiếp từ trạng thái lành sang trạng thái lỗi từ rất sớm — vào khoảng bản ghi 150 — tức trước khi hư hỏng trở nên nghiêm trọng. Điều này có giá trị thực tiễn lớn vì nó cho phép dự báo thời điểm bảo trì trước khi sự cố thực sự xảy ra, qua đó chuyển từ mô hình bảo trì phản ứng (bị động sửa chữa khi hỏng) sang mô hình bảo trì dự báo (chủ động ngăn ngừa). Một quan sát không kém phần quan trọng là pipeline hoàn toàn không sử dụng nhãn hư hỏng trong giai đoạn huấn luyện — toàn bộ "kiến thức" về điều gì là bất thường được trừu xuất chỉ từ phân phối thống kê của tập lành. Đặc tính này mở ra khả năng triển khai phương pháp trong các môi trường công nghiệp nơi dữ liệu lỗi gắn nhãn không tồn tại hoặc quá khan hiếm để huấn luyện một mô hình phân loại có giám sát.

### 6.2 Hạn chế và đề xuất cải tiến

Tuy vậy, pipeline cũng bộc lộ một số hạn chế đáng cân nhắc cho các nghiên cứu tiếp theo, được trình bày ở đây theo thứ tự từ cấp độ ứng dụng đến cấp độ thuật toán:

- **Yêu cầu tín hiệu tốc độ tham chiếu.** Phương pháp đòi hỏi phải có sẵn tín hiệu tốc độ tức thời — một yêu cầu không phải lúc nào cũng được đáp ứng trong môi trường công nghiệp, đặc biệt với các thiết bị cũ chưa được trang bị tachometer hoặc encoder. Trong những trường hợp đó, cần sử dụng các kỹ thuật *tacho-less* ước lượng tốc độ trực tiếp từ chính tín hiệu rung, ví dụ qua việc theo dấu tần số tức thời bằng biến đổi Hilbert hoặc các phương pháp time-frequency.
- **Giả thiết hiệp phương sai chéo trong công thức HI.** Công thức hiện tại giả thiết các đặc trưng độc lập với nhau — một giả thiết có thể không chính xác bởi rms và kurtosis có thể tương quan trong các chế độ vận hành cụ thể. Việc thay thế bằng khoảng cách Mahalanobis đầy đủ, hoặc sử dụng các phương pháp học một lớp như One-Class SVM hoặc Isolation Forest, có tiềm năng cải thiện độ phân biệt trong các bài toán phức tạp hơn.
- **Tập đặc trưng cố định.** Bốn đặc trưng được lựa chọn — dù có cơ sở cơ học vững — vẫn là một tập cố định và có thể bỏ sót thông tin chẩn đoán giàu có hơn ở miền tần số con (sub-band envelope analysis) hoặc miền thời gian-tần số (wavelet, EMD, spectral kurtosis).
- **Sai số tích lũy của xấp xỉ tích phân pha.** Việc sử dụng `cumsum` để xấp xỉ tích phân pha là một xấp xỉ Euler tiến với sai số $\mathcal{O}(\Delta t)$. Ở chế độ tốc độ biến thiên mạnh, sai số này có thể tích lũy đáng kể qua hàng nghìn mẫu; thay bằng `cumtrapz` (xấp xỉ trapezoidal, sai số $\mathcal{O}(\Delta t^2)$) sẽ cải thiện độ chính xác mà chi phí tính toán không tăng đáng kể.

---

## 7. Kết luận

Bài thực hành đã giúp xác lập một hiểu biết toàn diện và có chiều sâu về một trong những pipeline kinh điển nhất của lĩnh vực chẩn đoán bằng phân tích rung động: chuỗi xử lý angular resampling — synchronous averaging — difference signal — health indicator. Trên dữ liệu mẫu của Matania và cộng sự (2024), pipeline chứng minh được khả năng tách biệt rõ rệt giữa tín hiệu lành và tín hiệu lỗi với tỉ lệ HI lên đến 50:1 ở giai đoạn hư hỏng nặng, đồng thời phát hiện được giai đoạn chuyển tiếp từ rất sớm — đặc tính có ý nghĩa thiết yếu cho các ứng dụng bảo trì dự báo. Hơn cả các kết quả số học cụ thể, bài thực hành cung cấp một bài học phương pháp luận sâu sắc: rằng hiệu quả của xử lý tín hiệu không nằm ở độ phức tạp của các thuật toán mà ở sự thấu hiểu bản chất vật lý của hiện tượng và ở việc lựa chọn miền biểu diễn phù hợp với cấu trúc nội tại của tín hiệu. Việc chuyển từ miền thời gian sang miền góc, ở phương diện toán học, đơn giản chỉ là một phép tham số hóa lại; nhưng ở phương diện chẩn đoán, nó là điều kiện tiên quyết khiến mọi phép phân tích tiếp theo trở nên có ý nghĩa.

Trong các nghiên cứu tiếp theo, hướng phát triển tiềm năng bao gồm việc thay thế các bước cố định trong pipeline bằng các thành phần học được — chẳng hạn sử dụng mạng nơ-ron tích chập một chiều thay cho difference signal, hoặc autoencoder thay cho công thức HI — và tích hợp pipeline cổ điển này với các mô hình sinh dữ liệu (đặc biệt là mô hình diffusion áp lên dữ liệu CWRU) như một cách bổ sung dữ liệu lỗi tổng hợp, qua đó mở rộng phạm vi áp dụng sang các kịch bản dữ liệu lỗi khan hiếm vốn rất phổ biến trong thực tế công nghiệp.

---

## 8. Tài liệu tham khảo

1. Matania O., Bachar L., Bechhoefer E., Bortman J. *Signal Processing for the Condition-Based Maintenance of Rotating Machines via Vibration Analysis: A Tutorial.* Sensors **24**(2), 454 (2024). https://www.mdpi.com/1424-8220/24/2/454
2. Mã nguồn của bài tutorial: https://github.com/omriMatania/sp_for_cbm_of_rotating_machines_using_vibration_analysis_tutorial
3. Randall R. B. *Vibration-based Condition Monitoring*, 2nd ed., Wiley, 2021 — Chương 3 (Angular resampling), Chương 5 (Synchronous average).
4. Antoni J. *Cyclic spectral analysis in practice.* Mechanical Systems and Signal Processing **21**(2), 597–630 (2007).
5. Stewart R. M. *Some Useful Data Analysis Techniques for Gearbox Diagnostics.* Report MHM/R/10/77, Machine Health Monitoring Group, ISVR, University of Southampton, 1977.

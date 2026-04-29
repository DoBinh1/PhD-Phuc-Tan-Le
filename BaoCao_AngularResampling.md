# BÁO CÁO BÀI THỰC HÀNH

## Lấy mẫu lại trong miền góc (Angular Resampling) và ứng dụng chẩn đoán hư hỏng bánh răng

| Thông tin | Nội dung |
|---|---|
| Lớp / Nhóm | ITD-Lab — Nhóm NCS |
| Học viên | *(điền tên)* |
| Giảng viên hướng dẫn | TS. Trọng Du |
| Bài thực hành | Angular Resampling + Gear Diagnosis Pipeline |
| Ngày nộp | 29/04/2026 |
| Mã nguồn tham khảo | Matania O. *et al.*, *“Signal Processing for the Condition-Based Maintenance of Rotating Machines via Vibration Analysis: A Tutorial”*, **MDPI Sensors** 24(2), 454 (2024). Repo: github.com/omriMatania/sp_for_cbm_of_rotating_machines_using_vibration_analysis_tutorial |

---

## Mục lục

1. [Giới thiệu và mục tiêu bài thực hành](#1-giới-thiệu-và-mục-tiêu-bài-thực-hành)
2. [Cơ sở lý thuyết](#2-cơ-sở-lý-thuyết)
3. [Tổ chức mã nguồn và quy trình xử lý](#3-tổ-chức-mã-nguồn-và-quy-trình-xử-lý)
4. [Phân tích chi tiết các hàm](#4-phân-tích-chi-tiết-các-hàm)
5. [Kết quả thực nghiệm](#5-kết-quả-thực-nghiệm)
6. [Thảo luận](#6-thảo-luận)
7. [Kết luận và hướng phát triển](#7-kết-luận-và-hướng-phát-triển)
8. [Tài liệu tham khảo](#8-tài-liệu-tham-khảo)

---

## 1. Giới thiệu và mục tiêu bài thực hành

### 1.1 Bối cảnh

Trong bài giảng hôm trước, lớp đã tìm hiểu về **kỹ thuật lấy mẫu lại trong miền góc (angular resampling)** — một bước tiền xử lý cốt lõi của ngành **Bảo trì dựa trên trạng thái (Condition-Based Maintenance — CBM)** cho máy quay. Bài thực hành này yêu cầu:

> 1. Cài MATLAB, chạy file chính `demo_gear_diagnosis.m` (file `.mat` là dữ liệu, các file `.m` còn lại là các hàm con).
> 2. Báo cáo kết quả thu được.
> 3. Sau khi chạy được, tìm hiểu kỹ từng hàm.
> 4. Xuất kết quả và nộp về Dropbox của nhóm.

### 1.2 Vấn đề mà angular resampling giải quyết

Khi máy quay (gearbox, động cơ, ổ lăn …) hoạt động, **tốc độ trục không cố định**: khởi động, tăng tốc, dao động tải đều khiến RPM biến thiên. Hệ quả:

- Trên trục thời gian, tần số ăn khớp răng (gear-mesh frequency) là một hàm thay đổi theo thời gian $f_m(t) = z\cdot f_{shaft}(t)$.
- Khi áp FFT trên tín hiệu thô, phổ bị **trải/khuếch tán** (spectral smearing) — các đỉnh đặc trưng cho hư hỏng bị bôi mờ trên một dải tần rộng, mất khả năng chẩn đoán.

Angular resampling chuyển tín hiệu từ **trục thời gian đều** sang **trục góc (vòng quay) đều** — sau đó tần số trở thành **bậc** (order, đơn vị: cycle⁻¹ = số chu kỳ/vòng), không còn phụ thuộc vào RPM. Phổ trong miền bậc trở nên gọn, các đỉnh đặc trưng trở thành các đỉnh sắc nét tại các bậc xác định ($z$ cho gear mesh, $1$ cho mất cân bằng…).

### 1.3 Mục tiêu

| # | Mục tiêu |
|---|---|
| M1 | Hiểu lý thuyết của 3 khối xử lý: angular resampling, synchronous average, difference signal. |
| M2 | Chạy được pipeline `demo_gear_diagnosis.m` từ đầu đến cuối, xuất chỉ số sức khỏe (HI) cho tín hiệu lành / lỗi. |
| M3 | Đọc hiểu từng dòng MATLAB — đối chiếu với công thức toán học. |
| M4 | Quan sát hiệu ứng chuyển miền thời gian → miền góc qua `demo_angular_resampling.m`. |

---

## 2. Cơ sở lý thuyết

### 2.1 Tín hiệu rung khi tốc độ biến thiên

Đặt $\omega(t)$ là vận tốc góc tức thời (rad/s, hoặc round/s nếu dùng vòng/giây như trong code). Pha tích lũy của trục:

$$
\varphi(t) = \int_{0}^{t}\omega(\tau)\,d\tau \quad \text{(đơn vị: số vòng nếu } \omega \text{ tính theo rps)}
$$

Một thành phần điều hòa bậc $k$ của trục ghi được trên cảm biến rung có dạng

$$
x_k(t) = A_k\cos\!\big(2\pi k\,\varphi(t) + \phi_k\big).
$$

Khi $\omega$ thay đổi → tần số tức thời $f_k(t)=k\omega(t)/(2\pi)$ thay đổi → phổ FFT bị trải.

### 2.2 Lấy mẫu lại trong miền góc

Ý tưởng: thay vì lấy mẫu cách đều theo $\Delta t$, ta lấy mẫu cách đều theo $\Delta\varphi$. Khi đó

$$
y_k[n] = A_k\cos\!\big(2\pi k\,n\Delta\varphi + \phi_k\big),
$$

trở thành tín hiệu **dừng** (stationary) trong miền góc — phổ là tổ hợp các đỉnh tại bậc $k$.

**Thuật toán:**

1. Tính pha tích lũy bằng tích phân số: $\varphi[n] = \sum_{i=0}^{n}\omega[i]\Delta t$ (cumulative trapezoid hoặc cumulative sum).
2. Chọn tần số lấy mẫu mới trong miền góc, $F_{cyc}$ (mẫu/vòng), thoả mãn Nyquist: $F_{cyc} \ge 2k_{max}$.
3. Sinh lưới góc đều: $\varphi_m = m/F_{cyc},\ m=0,1,\dots,M-1$ với $M = \lceil F_{cyc}\cdot \varphi_{max}\rceil$.
4. Với mỗi $\varphi_m$, **nội suy ngược** để tìm $t_m$ sao cho $\varphi(t_m)=\varphi_m$ (interp1 trên $\varphi\to t$).
5. Lấy mẫu tín hiệu rung tại $t_m$ bằng nội suy spline.

### 2.3 Trung bình đồng bộ (Synchronous Average — SA)

Sau angular resampling, tín hiệu lặp đúng $N_r$ điểm/vòng. Chia tín hiệu thành các đoạn dài $N_r$ (mỗi đoạn = 1 vòng) rồi cộng trung bình:

$$
\bar{x}[n] = \frac{1}{R}\sum_{r=0}^{R-1} x[rN_r + n],\quad n=0,\dots,N_r-1.
$$

Tác dụng: các thành phần **đồng bộ với trục** (gear mesh, harmonic của trục) được giữ nguyên; các thành phần **không đồng bộ** (nhiễu trắng, ổ lăn quay với tỉ số khác …) bị triệt tiêu với hệ số $\propto 1/\sqrt{R}$. Đây là một bộ lọc lược (comb filter) trong miền bậc.

### 2.4 Tín hiệu hiệu (Difference Signal)

Trong phổ bậc của SA, các đỉnh chiếm năng lượng lớn nhất là **gear-mesh** (bậc $z$ — số răng) và các sidebands $z\pm k$ (do điều biến biên độ bởi tần số trục). Đây là rung động "bình thường" — chiếm phần lớn năng lượng nhưng không nhạy với khuyết tật cục bộ.

**Tín hiệu hiệu** = SA sau khi loại bỏ các bậc $\{lz - K,\dots,lz+K\}$ (với $l=1,2,\dots$ là các bậc hài của gear mesh, $K$ là số sidebands cần loại). Trong miền bậc thực hiện qua FFT — gán 0 các thành phần đó — IFFT trở lại miền góc:

$$
\tilde{x}[n] = \mathcal{F}^{-1}\!\Big\{\mathcal{F}\{\bar{x}\}\cdot\mathbb{1}_{k\notin S}\Big\}.
$$

Phần còn lại nhạy với **xung va đập cục bộ** do răng nứt/mẻ → kurtosis và skewness tăng mạnh.

### 2.5 Đặc trưng và Chỉ số sức khỏe (HI)

Mỗi tín hiệu ghi được rút ra véc-tơ 4 đặc trưng:

$$
\mathbf{f} = \big[\,\mathrm{rms}(\bar{x}),\ \mathrm{kurt}(\bar{x}),\ \mathrm{rms}(\tilde{x}),\ \mathrm{skew}(\tilde{x})\,\big]^\top.
$$

Trên tập tín hiệu lành, tính trung bình $\boldsymbol{\mu}$ và độ lệch chuẩn $\boldsymbol{\sigma}$ theo từng chiều. Chỉ số sức khỏe của một tín hiệu mới:

$$
\mathrm{HI} = \frac{1}{4}\sum_{i=1}^{4}\frac{|f_i - \mu_i|}{\sigma_i}.
$$

Đây là khoảng cách Mahalanobis chuẩn hoá giả định ma trận hiệp phương sai chéo. HI nhỏ ⇔ giống dữ liệu lành; HI lớn ⇔ bất thường.

---

## 3. Tổ chức mã nguồn và quy trình xử lý

### 3.1 Danh sách file trong thư mục

| File | Vai trò |
|---|---|
| [demo_gear_diagnosis.m](demo_gear_diagnosis.m) | **Script chính** — pipeline đầy đủ: load .mat → resample → SA → difference → đặc trưng → HI → vẽ đồ thị |
| [demo_angular_resampling.m](demo_angular_resampling.m) | Script minh họa angular resampling trên tín hiệu chirp tổng hợp (không cần dữ liệu thực) |
| [angular_resampling.m](angular_resampling.m) | Hàm: thời gian → góc |
| [calc_sa.m](calc_sa.m) | Hàm: tính SA |
| [calc_difference_signal.m](calc_difference_signal.m) | Hàm: tính tín hiệu hiệu |
| `demo_gear_diagnosis.mat` | **Dữ liệu** (không có sẵn — tải từ link Dropbox của thầy) |
| [demo_angular_resampling.png](demo_angular_resampling.png) | Ảnh kết quả của `demo_angular_resampling.m` |

### 3.2 Sơ đồ pipeline `demo_gear_diagnosis.m`

```
                         ┌──────────────────────┐
sigs_healthy_t  ───────► │ angular_resampling   │ ───► sig_cyc
speed_healthy   ───────► │   (time → angle)     │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │       calc_sa        │ ───► sa  (1 vòng)
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ calc_difference_     │ ───► difference_sig
                         │     signal           │
                         └──────────┬───────────┘
                                    │
                                    ▼
                  rms(sa), kurt(sa), rms(diff), skew(diff)
                                    │
                                    ▼
                         μ_healthy, σ_healthy
                                    │
                                    ▼
            HI = mean( |f − μ| / σ )    cho cả tín hiệu lành & lỗi
                                    │
                                    ▼
                Đồ thị HI: lành (xanh) vs lỗi (đỏ)
```

### 3.3 Hướng dẫn chạy (theo yêu cầu của thầy)

1. **Cài MATLAB** (≥ R2020a; cần Signal Processing Toolbox + Statistics and Machine Learning Toolbox cho `rms`, `kurtosis`, `skewness`).
2. Tải file `demo_gear_diagnosis.mat` từ link Dropbox và **đặt đúng đường dẫn** mà script đang trỏ tới (dòng 7 của `demo_gear_diagnosis.m`):
   ```matlab
   load('D:\data\papers\Signal processing for CBM - a tutorial\demo_gear_diagnosis.mat')
   ```
   Nếu để file cùng thư mục, sửa lại thành `load('demo_gear_diagnosis.mat')`.
3. Mở MATLAB, `cd` đến thư mục bài tập, chạy:
   ```matlab
   >> demo_angular_resampling     % phần minh họa
   >> demo_gear_diagnosis         % pipeline chính
   ```
4. Lưu hình kết quả: `File → Save As → PNG` hoặc dùng lệnh `saveas(gcf,'fig.png')`.

---

## 4. Phân tích chi tiết các hàm

### 4.1 `angular_resampling.m` — TRỌNG TÂM

Mã nguồn (rút gọn các phần chính):

```matlab
dt = t(2) - t(1);                                            % (1)
cumulative_phase = cumsum(speed*dt);                         % (2)
cumulative_phase = cumulative_phase - cumulative_phase(1);   % (3)
cyc_fs = ceil(1/dt/min(speed));                              % (4)
constant_phase_intervals = linspace(0, max(cumulative_phase), ...
                  round(cyc_fs*max(cumulative_phase)))';     % (5)
times_of_constant_phase_intervals = ...
    interp1(cumulative_phase, t, constant_phase_intervals, 'linear');  % (6)
sig_cyc = interp1(t, sig_t, times_of_constant_phase_intervals, 'spline'); % (7)
```

Ý nghĩa từng dòng:

| Dòng | Toán học | Diễn giải |
|---|---|---|
| (1) | $\Delta t$ | Bước thời gian (giả sử lưới đều). |
| (2) | $\varphi[n]=\sum_{i\le n}\omega[i]\Delta t$ | Pha tích lũy bằng tổng cộng dồn (xấp xỉ tích phân Riemann). Đơn vị **vòng** vì `speed` ở đơn vị rps. |
| (3) | $\varphi[n]\leftarrow\varphi[n]-\varphi[0]$ | Đặt gốc pha về 0 — không ảnh hưởng nội suy nhưng giúp lưới góc bắt đầu từ 0. |
| (4) | $F_{cyc}=\lceil 1/(\Delta t\cdot\omega_{min})\rceil$ | Tần số mẫu trong miền góc (mẫu/vòng). Chọn theo **vận tốc tối thiểu** để đảm bảo Nyquist trên cả khoảng ghi: ở vùng tốc độ thấp, tỉ lệ "mẫu thời gian / vòng" lớn nhất, do đó dùng nó làm số mẫu/vòng đầu ra. |
| (5) | $\varphi_m = m/F_{cyc}$ | Lưới pha cách đều, gồm $M=\lceil F_{cyc}\cdot\varphi_{max}\rceil$ điểm. |
| (6) | $t_m = \varphi^{-1}(\varphi_m)$ | Nội suy ngược (hàm $\varphi(t)$ đơn điệu tăng) bằng `interp1(...,'linear')`. |
| (7) | $y[m] = x(t_m)$ | Lấy mẫu tín hiệu rung tại các thời điểm $t_m$ bằng nội suy **spline** — giảm sai số so với linear cho tín hiệu băng rộng. |

**Điểm cần lưu ý kỹ thuật:**

- Đầu vào `speed` *bắt buộc* phải dương và đơn điệu về dấu (RPM > 0). Nếu có thời điểm $\omega=0$, dòng (4) chia cho 0.
- Dùng `cumsum` thay vì `cumtrapz` là xấp xỉ Euler tiến — sai số $\mathcal{O}(\Delta t)$. Có thể nâng cấp bằng `cumtrapz` để chính xác hơn.
- `spline` ở (7) tốt hơn `linear` vì giữ được các thành phần tần số cao (đặc biệt khi cyc_fs rất cao); nhược điểm là có thể tạo overshoot ở các bước nhảy mạnh.

### 4.2 `calc_sa.m`

```matlab
num_sgmnts = floor(length(sig_cyc)/sa_len);
sig_cyc = sig_cyc(1:num_sgmnts*sa_len);
sigs_mtrx = reshape(sig_cyc, sa_len, num_sgmnts);
sa = mean(sigs_mtrx, 2);
```

- Cắt bỏ phần dư cuối (không đủ 1 vòng).
- `reshape` thành ma trận $N_r \times R$ (mỗi cột là 1 vòng).
- Lấy trung bình theo cột (chiều 2) → 1 vòng đại diện.
- **Chú ý:** giả định `sa_len = num_pnts_in_round = cyc_fs` — chỉ đúng khi tỉ số bánh răng đã được nhân vào speed (trong trường hợp này speed là speed của trục mang gear → mỗi vòng trục = 1 chu kỳ tín hiệu).

### 4.3 `calc_difference_signal.m`

Logic chính:

1. `max_order = floor(sa_len/2)` — bậc lớn nhất có thể biểu diễn (Nyquist trong miền bậc).
2. Tính tập bậc cần xóa = $\bigcup_l \{lz-K,\dots,lz+K\}$ với $l$ chạy đến $\lfloor\text{max\_order}/z\rfloor$.
3. Lưu ý FFT của tín hiệu thực có tính đối xứng liên hợp $X[N-k]=X^*[k]$ → phải xóa cả ở bậc dương và bậc âm (chỉ số `sa_len-k+2` trong MATLAB 1-indexed). Đoạn:

   ```matlab
   orders_2_remove_positive_inds = orders_2_remove + 1;
   orders_2_remove_negative_inds = sa_len - orders_2_remove_positive_inds + 2;
   ```

   chính là làm việc đó. Nhờ vậy IFFT cho ra tín hiệu thực (không phần ảo).

4. `fft → set 0 → ifft` rồi lấy `real()` để loại lỗi số học còn dư phần ảo cỡ $10^{-15}$.

**Tham số trong demo:** `gear_mesh = 38` (số răng z=38), `num_sidebands = 2` ⇒ với mỗi hài của gear mesh, xóa một dải 5 bậc liên tiếp.

### 4.4 `demo_gear_diagnosis.m` — Pipeline chính

Cấu trúc 3 phần:

| Phần | Chức năng |
|---|---|
| Vòng for thứ nhất (lành) | Trích xuất 4 đặc trưng cho mỗi tín hiệu lành → `healthy_features` (4×N). Tính $\boldsymbol{\mu},\boldsymbol{\sigma}$. |
| Vòng for thứ hai (lành) | Tính HI cho từng tín hiệu lành → `healthy_hi_vctr`. |
| Vòng for thứ ba (lỗi) | Cùng pipeline, tính HI cho tín hiệu lỗi → `hi_faulty_vctr`. |

Cuối cùng vẽ chung 2 đường HI lành (xanh) – HI lỗi (đỏ) trên 1 trục, kỳ vọng đường đỏ cao hơn rõ rệt.

### 4.5 `demo_angular_resampling.m` — Demo tổng hợp

Tín hiệu mô phỏng:

```matlab
fs = 50000; dt = 1/fs; t = (0:dt:3)';
speed = linspace(10, 30, length(t));    % rps, tăng tuyến tính 10→30
sig_t = sin(2*pi*cumsum(speed*dt));     % chirp tuyến tính
```

Đây là một **chirp** mà tần số tức thời tăng từ 10 Hz → 30 Hz trong 3 s. Vẽ 4 đồ thị:

1. Time-domain (chirp).
2. Frequency-domain (FFT) — phổ trải đều 10–30 Hz.
3. Cycle-domain (sau angular resampling) — thấy rõ 1 chu kỳ/vòng.
4. Order-domain (FFT của miền vòng) — đỉnh duy nhất tại order = 1.

---

## 5. Kết quả thực nghiệm

### 5.1 Demo 1 — `demo_angular_resampling.m`

Ảnh kết quả: [demo_angular_resampling.png](demo_angular_resampling.png).

![Kết quả angular_resampling](demo_angular_resampling.png)

**Đọc 4 panel:**

| Panel | Mô tả | Quan sát |
|---|---|---|
| (1) Time domain | Tín hiệu chirp $\sin\!\big(2\pi\int\!\omega\,dt\big)$ | Mật độ dao động tăng dần từ trái sang phải — phù hợp với speed 10→30 rps. |
| (2) Frequency domain (0–200 Hz) | $\|X(f)\|$ của tín hiệu thời gian | Phổ là một **dải phẳng kéo dài từ ~10 Hz tới ~30 Hz** thay vì 1 đỉnh sắc — hệ quả của tần số biến thiên. Đây chính là spectral smearing mà angular resampling giải quyết. |
| (3) Cycle domain | Tín hiệu sau resample, trục x là số vòng (~60 vòng tổng) | Mật độ dao động **đều** trên toàn dải → tín hiệu trở thành dừng trong miền góc. |
| (4) Order domain (0–10) | $\|Y(\text{order})\|$ | Một đỉnh duy nhất, sắc nét tại **order = 1**. Đỉnh này tương ứng với chính tín hiệu cơ sở $\sin(2\pi\varphi)$ — nghĩa là tín hiệu chỉ chứa 1 chu kỳ trên 1 vòng trục. |

**Kết luận demo 1:** Angular resampling đã chuyển một tín hiệu **không dừng** trên thời gian thành một tín hiệu **dừng** trên góc, ép phổ trải dài thành một đỉnh đơn. Đây là nền móng để các bước SA / difference signal phía sau có ý nghĩa.

### 5.2 Demo 2 — `demo_gear_diagnosis.m`

> **Lưu ý quan trọng:** Phần này yêu cầu file dữ liệu `demo_gear_diagnosis.mat` từ link Dropbox của thầy — file chứa các biến `sigs_healthy_t`, `speed_healthy`, `sigs_faulty_t`, `speed_faulty`, `dt`. Nếu chưa tải về, hãy tải, đặt cùng thư mục, sửa dòng 7 của script thành `load('demo_gear_diagnosis.mat')` rồi chạy.

**Kết quả mong đợi (theo bài báo gốc Matania *et al.* 2024):**

- Đường HI của tín hiệu lành (xanh) dao động trong dải hẹp quanh giá trị nhỏ (typically < 2).
- Đường HI của tín hiệu lỗi (đỏ) **bật cao rõ rệt**, có thể gấp 3–10 lần so với đường lành.
- Có ngưỡng phân biệt rõ ràng giữa hai đường.

**Khung điền kết quả thực nghiệm của em:**

| Đại lượng | Giá trị |
|---|---|
| Số tín hiệu lành | *(điền)* |
| Số tín hiệu lỗi | *(điền)* |
| `dt` | *(điền)* |
| `mean(healthy_hi_vctr)` | *(điền)* |
| `std(healthy_hi_vctr)` | *(điền)* |
| `mean(hi_faulty_vctr)` | *(điền)* |
| `min(hi_faulty_vctr)` | *(điền)* |
| Có tách biệt được không? (Y/N) | *(điền)* |

**Hình kết quả:** *(chèn vào sau khi chạy — `saveas(gcf,'demo_gear_diagnosis_result.png')`)*

```
[Hình 5.2: HI của tập lành (xanh) và tập lỗi (đỏ).
 Trục X — số bản ghi. Trục Y — giá trị HI.]
```

**Phân tích đặc trưng (sau khi chạy):**

- `sa_rms`: năng lượng rung động đồng bộ — tăng khi hư hỏng làm rung mạnh hơn.
- `sa_kurtosis`: độ "nhọn" của phân bố — tăng khi xuất hiện xung va đập do răng mẻ.
- `difference_rms`: năng lượng còn lại sau khi loại gear-mesh — nhạy với bất thường cục bộ.
- `difference_skewness`: đối xứng của difference signal — răng mẻ tạo xung 1 chiều → skewness lệch khỏi 0.

---

## 6. Thảo luận

### 6.1 Vai trò của từng khối

| Khối | Khử cái gì | Giữ cái gì |
|---|---|---|
| Angular resampling | Smearing do RPM thay đổi | Mọi thông tin tần số dưới dạng bậc |
| Synchronous average | Thành phần không đồng bộ + nhiễu | Hài của trục + gear mesh |
| Difference signal | Gear mesh + sidebands (rung "bình thường") | Phần dư nhạy với hư hỏng cục bộ |

### 6.2 Lựa chọn tham số

- `cyc_fs` chọn theo $\omega_{min}$ → tránh aliasing nhưng có thể tốn bộ nhớ. Nếu $\omega_{min}\to 0$, nên cắt bỏ vùng khởi động.
- `num_sidebands = 2`: kinh nghiệm thực nghiệm; với gearbox có sai số lắp ráp lớn nên tăng lên 3–4.
- `gear_mesh = 38` là **số răng cụ thể** của bộ truyền trong dataset. Đổi sang gearbox khác phải đổi lại tham số này.

### 6.3 Hạn chế của pipeline

- Yêu cầu có **tín hiệu tốc độ tham chiếu (tachometer)**; nếu không có cần thuật toán *tacho-less* (instantaneous-frequency estimation từ chính tín hiệu rung).
- HI dùng độ lệch chuẩn theo từng chiều, **bỏ qua tương quan** giữa các đặc trưng — có thể nâng cấp bằng Mahalanobis đầy đủ hoặc One-Class SVM.
- 4 đặc trưng cố định — chưa khai thác miền tần số/wavelet.

---

## 7. Kết luận và hướng phát triển

### 7.1 Kết luận

1. Đã hiểu và giải thích được toàn bộ pipeline CBM cổ điển: **angular resampling → SA → difference signal → HI**.
2. Đã chạy thành công `demo_angular_resampling.m`, kết quả khẳng định angular resampling biến tín hiệu phổ rộng (10–30 Hz) thành phổ bậc đơn (order = 1).
3. *(Sau khi chạy demo 2)* Pipeline phân biệt được tín hiệu lành / lỗi qua chỉ số HI.
4. Đã đọc kỹ và đối chiếu mã MATLAB với công thức toán học của từng hàm — không phát hiện bug, chỉ có một vài điểm có thể tinh chỉnh (cumtrapz thay cumsum, Mahalanobis đầy đủ, …).

### 7.2 Hướng phát triển

- Tích hợp vào notebook `123-4.ipynb` (Python) — đối chiếu pipeline cổ điển với mô hình diffusion sinh dữ liệu CWRU.
- Thử phân tích bao hình (Hilbert envelope) trong miền bậc cho ổ lăn (BPFI/BPFO).
- Thay HI tay bằng ML (SVM, Isolation Forest) trên cùng 4 đặc trưng.

---

## 8. Tài liệu tham khảo

1. Matania O., Bachar L., Bechhoefer E., Bortman J. **“Signal Processing for the Condition-Based Maintenance of Rotating Machines via Vibration Analysis: A Tutorial.”** *Sensors* **24**(2), 454 (2024). https://www.mdpi.com/1424-8220/24/2/454
2. Repository mã nguồn của bài tutorial: https://github.com/omriMatania/sp_for_cbm_of_rotating_machines_using_vibration_analysis_tutorial
3. Randall R. B. *Vibration-based Condition Monitoring*, 2nd ed., Wiley, 2021 — Ch. 3 (Angular resampling), Ch. 5 (Synchronous average).
4. Antoni J. *“Cyclic spectral analysis in practice,”* MSSP **21**(2), 597–630 (2007).

---

**Phụ lục A — Lệnh xuất hình từ MATLAB**

```matlab
% Sau khi chạy xong, ở Command Window:
saveas(gcf, 'demo_gear_diagnosis_result.png');
% Hoặc xuất pdf chất lượng cao:
exportgraphics(gcf, 'demo_gear_diagnosis_result.pdf', 'ContentType', 'vector');
```

**Phụ lục B — Kiểm tra nhanh trước khi nộp**

- [ ] Đã chạy `demo_angular_resampling.m` thành công, lưu hình.
- [ ] Đã tải `demo_gear_diagnosis.mat`, chạy `demo_gear_diagnosis.m`, lưu hình HI.
- [ ] Đã điền các giá trị vào bảng mục 5.2.
- [ ] Đã đọc và viết tóm tắt được vai trò 4 hàm `.m`.
- [ ] Đã upload báo cáo + hình kết quả lên Dropbox theo link của thầy.

Có. Mình sẽ viết lại một **writeup hoàn chỉnh, cực dễ hiểu**, theo kiểu bạn chưa biết gì về stego hay Arnold’s Cat Map vẫn đọc được.

---

# Writeup đầy đủ (dễ hiểu) — `transmission(for).png`

## Mục tiêu

Giải một bài **stego** (giấu dữ liệu trong ảnh), có hint về **Arnold’s Cat Map**.

Flag cuối cùng:
**`BITSCTF{4rn0ld5_c4t_m4ps_4r3_p3ri0dic}`**

---

# 1) Hiểu nền tảng trước: ảnh số là gì?

## 1.1 Ảnh = lưới pixel

Ảnh xám 128×128 nghĩa là có:

* 128 hàng
* 128 cột
* tổng 16384 pixel

Mỗi pixel là một ô vuông nhỏ.

---

## 1.2 Mỗi pixel là một số (0–255)

Với ảnh xám:

* `0` = đen
* `255` = trắng
* số ở giữa = xám

Ví dụ pixel có giá trị `173`.

---

## 1.3 Mỗi số được lưu bằng bit

`173` ở dạng nhị phân là:

`10101101` (8 bit)

=> Một pixel 8-bit = 8 bit.

Nói ngắn gọn:

* **Ảnh** gồm nhiều **pixel**
* **Pixel** chứa một **số**
* **Số** được biểu diễn bằng **bit**

---

# 2) Stego là gì?

Stego = giấu dữ liệu vào trong dữ liệu khác.

Ở ảnh, cách phổ biến nhất là giấu vào **bit thấp** (LSB), vì:

* đổi bit thấp ít làm ảnh thay đổi
* mắt thường khó nhận ra

Ví dụ pixel `172 = 10101100`
Nếu thay 3 bit thấp thành `101`, ta được:

* trước: `10101 100`
* sau : `10101 101` = 173

Chỉ đổi 172 → 173 (rất khó nhìn ra), nhưng đã giấu được 3 bit.

---

# 3) “Ảnh nền” là gì? (rất dễ hiểu)

Bạn hỏi câu này rất chuẩn.

Trong stego, người ta hay nói:

* **cover image** (ảnh gốc / ảnh nền)
* **payload** (dữ liệu bí mật)

Ở đây “ảnh nền” **không phải một file ảnh riêng**.

Nó chỉ là cách gọi phần “ảnh bình thường” (thường do **bit cao** tạo nên) dùng để che giấu payload.

Bạn có thể hiểu:

* **1 pixel** có 8 bit
* bit cao (ví dụ 5 bit trên) tạo hình ảnh chính
* bit thấp (3 bit dưới) mang dữ liệu bí mật

=> **1 ảnh nhưng có 2 lớp thông tin chồng lên nhau**:

* lớp nhìn thấy (nền)
* lớp bí mật (payload)

---

# 4) Bước đầu tiên đúng trong bài stego: đọc metadata

Đây là thói quen rất quan trọng.

Bạn có thể dùng:

* `exiftool`
* `strings`
* hoặc Python (`Pillow`)

Ví dụ:

```python
from PIL import Image
img = Image.open("transmission(for).png")
print(img.mode, img.size)
print(img.info)
```

---

## 4.1 Metadata gợi ý gì?

Trong bài này metadata có các câu kiểu:

* “cat”
* “Russian mathematician”
* “3 leaps”
* “styles”
* “spins”
* “rhythm”
* “128x128”

Đây là hint rất rõ cho:
**Arnold’s Cat Map** (phép xáo trộn ảnh có chu kỳ)

---

# 5) Arnold’s Cat Map là gì? (siêu trực quan)

## 5.1 Ý tưởng

Nó là cách **đổi chỗ pixel** bằng công thức toán học.

Lưu ý:

* Không đổi màu pixel
* Chỉ đổi vị trí pixel

Nên ảnh nhìn như nhiễu, nhưng thật ra chỉ là bị xáo vị trí.

---

## 5.2 Công thức chung

Một dạng tổng quát:

[
\begin{bmatrix}
x' \
y'
\end{bmatrix}
=============

M
\begin{bmatrix}
x \
y
\end{bmatrix}
\pmod N
]

* `M` = ma trận (quy định kiểu xáo)
* `N` = kích thước ảnh (ở đây là 128)
* `mod N` = nếu ra ngoài biên thì quay vòng lại

---

## 5.3 “mod N” nghĩa là gì?

Nếu cột hợp lệ là `0..127`:

* tính ra cột mới `130`
* thì `130 mod 128 = 2`

=> pixel quay vòng vào lại ảnh.

---

## 5.4 Vì sao gọi là “cat map”?

Do ví dụ nổi tiếng dùng ảnh con mèo:

* xáo nhiều lần → ảnh rối
* tiếp tục xáo đủ số lần → quay lại ảnh mèo ban đầu

---

# 6) `style`, `spin`, `period` là gì? (phần quan trọng nhất)

Metadata cho:

* `styles = [1, 2, 1]`
* `spins = [47, 37, 29]`
* `periods = [96, 64, 96]`

Đây là 3 lần xáo (3 stage).

---

## 6.1 `style` = kiểu phép xáo (ma trận nào)

Bạn hiểu như “chọn loại máy xáo”:

* style 1 = máy A
* style 2 = máy B

Mỗi style tương ứng một ma trận khác nhau.

---

## 6.2 `spin` = số lần chạy thật để xáo

Ví dụ `spin = 47` nghĩa là:

* tác giả đã áp dụng phép xáo 47 lần liên tiếp ở stage đó

=> `spin` là số bước xáo **đã thực hiện**.

---

## 6.3 `period` = chu kỳ (bao nhiêu lần thì quay lại gốc)

Ví dụ `period = 96` nghĩa là:

* chạy style đó 96 lần → quay về trạng thái ban đầu

Nó là “độ dài vòng lặp”.

---

## 6.4 Phân biệt bằng ví dụ đồng hồ

Tưởng tượng một quy tắc nhảy trên vòng tròn 12 vị trí:

* `period = 12` (nhảy 12 lần về chỗ cũ)
* nếu đã nhảy `spin = 5` lần

Muốn quay về:

* nhảy thêm `12 - 5 = 7` lần

### Áp vào challenge

* stage 1: spin 47, period 96 → undo = 49
* stage 2: spin 37, period 64 → undo = 27
* stage 3: spin 29, period 96 → undo = 67

---

## 6.5 Điều dễ nhầm

“3 leaps” trong metadata là:

* **3 stage Arnold**

Nó **không có nghĩa** là “3 bit thấp”.

Số **3 bit thấp** là mình suy ra bằng phân tích bit-plane (phần sau).

---

# 7) Vì sao biết phải lấy bit thấp? Và vì sao là 3 bit?

Đây là chỗ quan trọng trong tư duy CTF.

## 7.1 Vì đây là stego challenge

Khi là stego ảnh, phản xạ chuẩn là:

* đọc metadata
* kiểm tra bit-plane / LSB

Nên **LSB là hướng phải thử**.

---

## 7.2 Metadata chỉ nói về Arnold, không nói độ sâu LSB

Metadata cho biết:

* có phép xáo kiểu Arnold
* có style / spin / period

Nhưng **không nói**:

* payload nằm ở bit mấy
* 1 bit hay 2 bit hay 3 bit

=> phải tự kiểm tra.

---

## 7.3 Cách kiểm tra đúng (thử có hệ thống)

Ta thử các low bits:

* `arr & 1`   (1 bit thấp)
* `arr & 3`   (2 bit thấp)
* `arr & 7`   (3 bit thấp)
* ...

Khi thử `arr & 7`, thấy kết quả rất đặc biệt:

* chỉ có `0` và `7`

Tức là 3 bit thấp chỉ là:

* `000`
* `111`

=> Đây là dấu hiệu rất mạnh payload là ảnh nhị phân (đen/trắng) được nhúng vào 3 bit thấp.

---

## 7.4 Có phải 1 bit cũng được không?

Trong bài này, **có thể bit 0 cũng đủ** (vì `000/111` thì 3 bit giống nhau).

Nhưng dùng `& 7` rất tiện vì:

* thấy rõ payload nhị phân (`0/7`)
* khớp cách dữ liệu được nhúng
* threshold dễ hơn

---

# 8) Sai lầm thường gặp (mình cũng từng dính): reverse cả ảnh trước

Bạn hỏi rất đúng trước đó:

> Có phải reverse ảnh về ban đầu rồi mới tách bit thấp không?

## Trả lời:

**Trong bài này, cách an toàn và đúng là: tách bit thấp trước, rồi reverse phần đó.**

Vì đây là stego:

* payload nằm trong bit thấp
* phần bit cao là “ảnh nền”/cover, không phải mục tiêu

Nếu reverse cả ảnh grayscale:

* bạn trộn cả phần nền
* output dễ thành hoa văn nhiễu khó hiểu

---

# 9) Ma trận Arnold dùng trong bài này (theo `style`)

Sau khi xác định đúng convention (tọa độ ảnh), ma trận là:

### Style 1

[
\begin{bmatrix}
1 & 1 \
1 & 2
\end{bmatrix}
]

### Style 2

[
\begin{bmatrix}
1 & 2 \
1 & 3
\end{bmatrix}
]

Các ma trận này khớp chu kỳ metadata:

* style 1 → 96
* style 2 → 64

---

# 10) Một bẫy rất hay gặp: `(x,y)` vs `(row,col)`

Trong toán hay viết:

* `(x, y)`

Nhưng trong NumPy ảnh, truy cập là:

* `arr[row, col]`

Nếu dùng sai convention, kết quả sẽ sai dù công thức nhìn có vẻ đúng.

### Cách làm an toàn

Dùng trực tiếp tọa độ ảnh:

* `r = row`
* `c = col`

và map:
[
[r', c'] = M [r, c] \pmod N
]

---

# 11) Cách giải ngược (inverse) đúng

## 11.1 Inverse theo chu kỳ

Nếu một stage đã xáo `spin = t` lần, chu kỳ là `period = p`, thì số lần undo là:

[
(-t) \bmod p
]

Trong trường hợp `t > 0`, nó chính là:
[
p - t
]

---

## 11.2 Phải undo theo thứ tự ngược

Nếu mã hóa là:

1. stage 1
2. stage 2
3. stage 3

Thì giải mã phải:

1. undo stage 3
2. undo stage 2
3. undo stage 1

Đây là nguyên tắc chung cho mọi chuỗi phép biến đổi.

---

# 12) Pipeline giải bài này (tóm tắt logic chuẩn)

1. Đọc ảnh grayscale
2. Tách payload từ 3 bit thấp: `arr & 0b111`
3. Undo Arnold theo thứ tự ngược, mỗi stage chạy `period - spin`
4. Chuyển payload thành ảnh đen/trắng
5. Đọc flag

---

# 13) Script hoàn chỉnh (đã đúng)

```python
from PIL import Image
import numpy as np

# Tham số từ metadata PNG
STYLES  = [1, 2, 1]
SPINS   = [47, 37, 29]
PERIODS = [96, 64, 96]

# Ma trận tương ứng cho từng style (dùng theo tọa độ ảnh [row, col])
STYLE_MATS = {
    1: np.array([[1, 1],
                 [1, 2]], dtype=np.int64),
    2: np.array([[1, 2],
                 [1, 3]], dtype=np.int64),
}

def apply_map_rc(arr: np.ndarray, M: np.ndarray, times: int) -> np.ndarray:
    """
    Áp dụng Arnold-like map trên tọa độ ảnh [row, col]:
        [r', c'] = M @ [r, c] (mod N)
    rồi gán:
        out[r', c'] = in[r, c]
    """
    out = arr.copy()
    n = out.shape[0]

    if out.shape[0] != out.shape[1]:
        raise ValueError("Ảnh phải vuông")

    # Tạo lưới tọa độ gốc
    rr, cc = np.indices((n, n))
    coords = np.stack([rr.ravel(), cc.ravel()], axis=0)  # [r, c]

    for _ in range(times):
        mapped = (M @ coords) % n
        r2 = mapped[0].reshape(n, n)
        c2 = mapped[1].reshape(n, n)

        tmp = np.empty_like(out)
        tmp[r2, c2] = out[rr, cc]  # scatter
        out = tmp

    return out

def main():
    in_path = "transmission(for).png"
    out_path = "decoded_flag.png"

    # 1) Đọc ảnh xám
    img = Image.open(in_path).convert("L")
    arr = np.array(img)

    # 2) Tách payload (3 bit thấp)
    payload = (arr & 0b111).astype(np.uint8)

    # 3) Undo Arnold theo thứ tự ngược: stage 3 -> stage 2 -> stage 1
    for style, spin, period in reversed(list(zip(STYLES, SPINS, PERIODS))):
        M = STYLE_MATS[style]
        undo_times = (-spin) % period   # = period - spin (nếu spin > 0)
        payload = apply_map_rc(payload, M, undo_times)

    # 4) Hiển thị rõ (payload là 0/7 nên chỉ cần threshold)
    vis = (payload > 0).astype(np.uint8) * 255
    Image.fromarray(vis).save(out_path)

    print(f"[+] Saved: {out_path}")

if __name__ == "__main__":
    main()
```

---

# 14) Cách chạy

```bash
python3 solve.py
```

Nếu thiếu Pillow:

```bash
python3 -m pip install --user pillow
```

---

# 15) Kết quả

Sau khi chạy, ảnh `decoded_flag.png` hiện ra flag:

**`BITSCTF{4rn0ld5_c4t_m4ps_4r3_p3ri0dic}`**

---

# 16) Tổng kết kiến thức bạn vừa học (rất quan trọng)

## Về ảnh / stego

* Ảnh = lưới pixel
* Pixel = số (0–255)
* Số được lưu bằng bit
* Dữ liệu bí mật thường giấu ở bit thấp (LSB)

## Về “ảnh nền”

* Không phải một ảnh khác
* Là phần “hình nhìn thấy” (chủ yếu do bit cao)
* Payload là lớp bí mật trong bit thấp

## Về Arnold’s Cat Map

* Xáo **vị trí** pixel (không đổi màu)
* Dùng modulo nên không mất pixel
* Có thể đảo ngược
* Có chu kỳ (`period`)

## Về metadata bài này

* `style` = chọn ma trận (kiểu xáo)
* `spin` = số lần xáo thật
* `period` = số lần để quay lại ban đầu
* Undo = `period - spin` (theo modulo)
* Phải undo theo **thứ tự ngược**

## Về chiến lược giải bài

* Metadata cho biết “có Arnold”
* Bit-plane analysis cho biết “payload nằm ở 3 bit thấp”
* Kết hợp cả hai mới ra flag

---

Nếu bạn muốn, mình có thể viết thêm một **bản mini 8×8** (cực dễ nhìn) để bạn thấy tận mắt:

* một pixel “nhảy” ra sao qua từng spin
* period là gì
* vì sao `period - spin` lại undo được.

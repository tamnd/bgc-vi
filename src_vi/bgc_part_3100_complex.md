<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Số phức

[i[Complex numbers]<]

Một đoạn nhập môn tí hon về [flw[số phức|Complex_number]] ăn cắp
thẳng từ Wikipedia:

> **Số phức** là số có thể được biểu diễn dưới dạng $a+bi$, trong đó
> $a$ và $b$ là số thực [tức là kiểu dấu chấm động trong C], và $i$
> đại diện cho đơn vị ảo, thoả mãn phương trình $i^2=−1$. Vì không
> có số thực nào thoả phương trình này, $i$ được gọi là số ảo. Với
> số phức $a+bi$, $a$ được gọi là **phần thực**, và $b$ được gọi là
> **phần ảo**.

Nhưng tới đây là hết rồi. Ta giả định nếu bạn đọc chương này, bạn
biết số phức là gì và bạn muốn làm gì với chúng.

Và tất cả ta cần lo là tiện ích của C để làm điều đó.

Hoá ra, hỗ trợ số phức trong compiler là tính năng _tuỳ chọn_. Không
phải compiler tuân chuẩn nào cũng làm được. Và những compiler làm
được, có thể làm ở các mức độ hoàn chỉnh khác nhau.

Bạn có thể check xem hệ của bạn có hỗ trợ số phức không với:

[i[`__STDC_NO_COMPLEX__` macro]<]

``` {.c}
#ifdef __STDC_NO_COMPLEX__
#error Complex numbers not supported!
#endif
```

[i[`__STDC_NO_COMPLEX__` macro]>]

[i[`__STDC_IEC_559_COMPLEX__` macro]<]

Thêm nữa, có một macro báo việc tuân theo chuẩn ISO 60559 (IEEE
754) cho toán dấu chấm động với số phức, cũng như sự hiện diện của
kiểu `_Imaginary`.

``` {.c}
#if __STDC_IEC_559_COMPLEX__ != 1
#error Need IEC 60559 complex support!
#endif
```

[i[`__STDC_IEC_559_COMPLEX__` macro]>]

Chi tiết thêm về chuyện đó ghi ở Annex G trong spec C11.

## Kiểu phức

Để dùng số phức, [i[`complex.h` header file]] `#include
<complex.h>`.

Với cái đó, bạn có ít nhất hai kiểu:

[i[`_Complex` type]]
[i[`complex` type]<]

``` {.c}
_Complex
complex
```

Cả hai đều có cùng nghĩa, nên cứ dùng `complex` cho đẹp.

[i[`complex` type]>]

Bạn cũng có vài kiểu cho số ảo nếu cài đặt của bạn tuân IEC 60559:

[i[`_Imaginary` type]]
[i[`imaginary` type]<]

``` {.c}
_Imaginary
imaginary
```

Cả hai này cũng có cùng nghĩa, nên cứ dùng `imaginary` cho đẹp.

[i[`imaginary` type]>]

Bạn cũng có giá trị cho số ảo $i$, chính nó:

[i[`I` macro]<]
[i[`_Complex_I` macro]<]
[i[`_Imaginary_I` macro]<]

``` {.c}
I
_Complex_I
_Imaginary_I
```

Macro `I` được set thành `_Imaginary_I` (nếu có), hoặc `_Complex_I`.
Nên cứ dùng `I` cho số ảo.

[i[`I` macro]>]
[i[`_Complex_I` macro]>]

[i[`__STDC_IEC_559_COMPLEX__` macro]<]

Lề một chút: tôi đã nói nếu compiler set `__STDC_IEC_559_COMPLEX__`
thành `1`, nó phải hỗ trợ kiểu `_Imaginary` để tuân chuẩn. Đó là
cách tôi đọc spec. Tuy nhiên, tôi không biết một compiler nào thực
sự hỗ trợ `_Imaginary` dù chúng có set
`__STDC_IEC_559_COMPLEX__`. Nên tôi sẽ viết một số code với kiểu đó
ở đây mà tôi không có cách nào test. Xin lỗi!

[i[`__STDC_IEC_559_COMPLEX__` macro]>]
[i[`_Imaginary_I` macro]>]

OK, giờ ta biết có kiểu `complex`, ta dùng nó sao?

## Gán số phức

[i[Complex numbers-->declaring]<]

Vì số phức có phần thực và phần ảo, nhưng cả hai đều dựa vào số dấu
chấm động để lưu giá trị, ta cũng cần báo C dùng độ chính xác nào
cho các phần đó của số phức.

[i[`complex float` type]<]
[i[`complex double` type]<]
[i[`complex long double` type]<]

Ta làm chuyện đó bằng cách đính kèm `float`, `double`, hay `long
double` vào `complex`, trước hay sau đều được.

[i[`complex long double` type]>]

Định nghĩa một số phức dùng `float` cho các thành phần của nó:

``` {.c}
float complex c;   // Spec prefers this way
complex float c;   // Same thing--order doesn't matter
```

Vậy khai báo thì ổn rồi, còn khởi tạo hay gán thì sao?

Hoá ra ta được dùng ký pháp khá tự nhiên. Ví dụ!

[i[`I` macro]<]

``` {.c}
double complex x = 5 + 2*I;
double complex y = 10 + 3*I;
```

[i[`I` macro]>]

Cho $5+2i$ và $10+3i$, tương ứng.

[i[`complex float` type]>]
[i[`complex double` type]>]
[i[Complex numbers-->declaring]>]

## Dựng, xé, và in

Ta đang tới đó...

Ta đã thấy một cách viết số phức:

``` {.c}
double complex x = 5 + 2*I;
```

Cũng không vấn đề gì khi dùng số dấu chấm động khác để dựng nó:

``` {.c}
double a = 5;
double b = 2;
double complex x = a + b*I;
```

[i[`CMPLX()` macro]<]

Cũng có một bộ macro để giúp dựng mấy cái này. Code trên có thể được
viết dùng macro `CMPLX()`, như vầy:

``` {.c}
double complex x = CMPLX(5, 2);
```

Theo như tôi tìm hiểu, mấy cái này _gần như_ tương đương:

``` {.c}
double complex x = 5 + 2*I;
double complex x = CMPLX(5, 2);
```

Nhưng macro `CMPLX()` sẽ xử lý zero âm ở phần ảo đúng mỗi lần, còn
cách kia có thể chuyển chúng thành zero dương. Tôi _nghĩ_^[Cái này
là cái khó research hơn, và tôi sẽ nhận thêm thông tin ai đó cho
tôi. `I` có thể được định nghĩa là `_Complex_I` hay `_Imaginary_I`,
nếu cái sau tồn tại. `_Imaginary_I` sẽ xử lý zero có dấu, nhưng
`_Complex_I` _có thể_ không. Cái này có hàm ý với branch cut và các
thứ toán-số-phức khác. Có lẽ. Bạn thấy tôi thực sự ra khỏi khu vực
chuyên môn chưa? Dù sao, macro `CMPLX()` hành xử như thể `I` được
định nghĩa là `_Imaginary_I`, với zero có dấu, kể cả khi
`_Imaginary_I` không tồn tại trên hệ.] Cái này có vẻ hàm ý rằng nếu
có khả năng phần ảo là zero, bạn nên dùng macro... nhưng ai đó nên
sửa tôi về điều này nếu tôi nhầm!

Macro `CMPLX()` chạy trên kiểu `double`. Có hai macro khác cho
`float` và `long double`: [i[`CMPLXF()` macro]] `CMPLXF()` và
[i[`CMPLXL()` macro]] `CMPLXL()`. (Hậu tố "f" và "l" xuất hiện hầu
như trong tất cả các hàm liên quan tới số phức.)

[i[`CMPLX()` macro]>]

Giờ thử ngược lại: nếu ta có số phức, làm sao tách nó ra phần thực
và phần ảo?

[i[`creal()` function]<]
[i[`cimag()` function]<]

Ta có một cặp hàm sẽ trích phần thực và phần ảo từ số: `creal()` và
`cimag()`:

``` {.c}
double complex x = 5 + 2*I;
double complex y = 10 + 3*I;

printf("x = %f + %fi\n", creal(x), cimag(x));
printf("y = %f + %fi\n", creal(y), cimag(y));
```

cho output:

``` {.default}
x = 5.000000 + 2.000000i
y = 10.000000 + 3.000000i
```

Lưu ý rằng chữ `i` tôi có trong chuỗi format của `printf()` là chữ
`i` theo nghĩa đen được in ra, nó không phải phần của format
specifier. Cả hai giá trị trả về từ `creal()` và `cimag()` đều là
`double`.

Và như thường, có các biến thể `float` và `long double` của các hàm
này: [i[`crealf()` function]] `crealf()`, [i[`cimagf()` function]]
`cimagf()`, [i[`creall()` function]] `creall()`, và [i[`cimagl()`
function]] `cimagl()`.

[i[`creal()` function]>]
[i[`cimag()` function]>]

## Số học và so sánh số phức

[i[Complex numbers-->arithmetic]<]

Số học có thể được thực hiện trên số phức, tuy cách nó chạy về mặt
toán học nằm ngoài phạm vi guide này.

``` {.c .numberLines}
#include <stdio.h>
#include <complex.h>

int main(void)
{
    double complex x = 1 + 2*I;
    double complex y = 3 + 4*I;
    double complex z;

    z = x + y;
    printf("x + y = %f + %fi\n", creal(z), cimag(z));

    z = x - y;
    printf("x - y = %f + %fi\n", creal(z), cimag(z));

    z = x * y;
    printf("x * y = %f + %fi\n", creal(z), cimag(z));

    z = x / y;
    printf("x / y = %f + %fi\n", creal(z), cimag(z));
}
```

cho kết quả:

``` {.default}
x + y = 4.000000 + 6.000000i
x - y = -2.000000 + -2.000000i
x * y = -5.000000 + 10.000000i
x / y = 0.440000 + 0.080000i
```

Bạn cũng có thể so sánh hai số phức về bằng nhau (hoặc không bằng):

``` {.c .numberLines}
#include <stdio.h>
#include <complex.h>

int main(void)
{
    double complex x = 1 + 2*I;
    double complex y = 3 + 4*I;

    printf("x == y = %d\n", x == y);  // 0
    printf("x != y = %d\n", x != y);  // 1
}
```

với output:

``` {.default}
x == y = 0
x != y = 1
```

Chúng bằng nhau nếu cả hai thành phần test bằng. Lưu ý rằng như với
mọi dấu chấm động, chúng có thể bằng nếu đủ gần do lỗi làm tròn^[Sự
đơn giản của câu này không làm tròn được lượng công sức khủng khiếp
đổ vào việc chỉ đơn thuần hiểu dấu chấm động thực sự hoạt động thế
nào.
https://randomascii.wordpress.com/2012/02/25/comparing-floating-point-numbers-2012-edition/].

[i[Complex numbers-->arithmetic]>]

## Toán số phức

Nhưng khoan! Còn nhiều hơn chỉ là số học số phức đơn giản!

Đây là bảng tổng hợp mọi hàm toán có sẵn cho bạn với số phức.

Tôi sẽ chỉ liệt kê phiên bản `double` của mỗi hàm, nhưng với tất cả
chúng đều có phiên bản `float` bạn lấy được bằng cách thêm `f` vào
tên hàm, và phiên bản `long double` bạn lấy được bằng cách thêm `l`.

Ví dụ, hàm `cabs()` dùng tính giá trị tuyệt đối của số phức cũng có
biến thể `cabsf()` và `cabsl()`. Tôi bỏ chúng cho ngắn.

### Hàm lượng giác

[i[`ccos()` function]]
[i[`csin()` function]]
[i[`ctan()` function]]
[i[`cacos()` function]]
[i[`casin()` function]]
[i[`catan()` function]]
[i[`ccosh()` function]]
[i[`csinh()` function]]
[i[`ctanh()` function]]
[i[`cacosh()` function]]
[i[`casinh()` function]]
[i[`catanh()` function]]

|Hàm|Mô tả|
|-|-|
|`ccos()`|Cosine|
|`csin()`|Sine|
|`ctan()`|Tangent|
|`cacos()`|Arc cosine|
|`casin()`|Arc sine|
|`catan()`|Chơi _Settlers of Catan_|
|`ccosh()`|Hyperbolic cosine|
|`csinh()`|Hyperbolic sine|
|`ctanh()`|Hyperbolic tangent|
|`cacosh()`|Arc hyperbolic cosine|
|`casinh()`|Arc hyperbolic sine|
|`catanh()`|Arc hyperbolic tangent|

### Hàm mũ và logarit

[i[`cexp()` function]]
[i[`clog()` function]]

|Hàm|Mô tả|
|-|-|
|`cexp()`|Mũ cơ số $e$|
|`clog()`|Logarit tự nhiên (cơ số $e$)|

### Hàm luỹ thừa và giá trị tuyệt đối

[i[`cabs()` function]]
[i[`cpow()` function]]
[i[`csqrt()` function]]

|Hàm|Mô tả|
|-|-|
|`cabs()`|Giá trị tuyệt đối|
|`cpow()`|Luỹ thừa|
|`csqrt()`|Căn bậc hai|

### Hàm thao tác

[i[`creal()` function]]
[i[`cimag()` function]]
[i[`CMPLX()` macro]]
[i[`carg()` function]]
[i[`conj()` function]]
[i[`cproj()` function]]

|Hàm|Mô tả|
|-|-|
|`creal()`|Trả phần thực|
|`cimag()`|Trả phần ảo|
|`CMPLX()`|Dựng số phức|
|`carg()`|Argument / góc pha|
|`conj()`|Liên hợp[^4a34]|
|`cproj()`|Phép chiếu lên mặt cầu Riemann|

[^4a34]: Đây là cái duy nhất không bắt đầu bằng chữ `c` thêm đằng
trước, lạ thay.

[i[Complex numbers]>]

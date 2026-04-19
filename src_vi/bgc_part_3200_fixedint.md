<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Kiểu số nguyên bề rộng cố định

[i[Fixed width integers]<]

C có đủ các kiểu số nguyên nhỏ, lớn hơn, và lớn nhất kiểu `int`,
`long` và thế nọ thế kia. Và bạn có thể xem trong [phần giới
hạn](#limits-macros) để thấy int lớn nhất là gì với `INT_MAX` và
tương tự.

Các kiểu đó to bao nhiêu? Tức là, chúng chiếm mấy byte? Ta có thể
dùng `sizeof` để ra câu trả lời.

Nhưng nếu tôi muốn đi ngược lại thì sao? Nếu tôi cần kiểu chính xác
32 bit (4 byte) hoặc ít nhất 16 bit hay đại loại thế?

Làm sao khai báo kiểu có kích thước nhất định?

Header [i[`stdint.h` header file]] `<stdint.h>` cho ta cách.

## Các kiểu theo số bit

Với cả số nguyên có dấu và không dấu, ta có thể chỉ định kiểu có số
bit nhất định, với vài cảnh báo, đương nhiên.

Và có ba nhóm chính các kiểu này (trong các ví dụ, `N` sẽ được thay
bằng số bit cụ thể):

* Số nguyên chính xác kích thước nào đó (`intN_t`)
* Số nguyên ít nhất kích thước nào đó (`int_leastN_t`)
* Số nguyên ít nhất kích thước nào đó và nhanh hết mức có thể
  (`int_fastN_t`)[^4582]

[^4582]: Một số kiến trúc có dữ liệu kích thước khác mà CPU và RAM
có thể thao tác với tốc độ nhanh hơn các kiểu khác. Trong các trường
hợp đó, nếu bạn cần số 8-bit nhanh nhất, có thể nó cho bạn kiểu 16-
hay 32-bit thay thế vì cái đó chỉ đơn giản là nhanh hơn. Nên với cái
này, bạn sẽ không biết kiểu to bao nhiêu, nhưng nó sẽ ít nhất to như
bạn nói.

`fast` nhanh hơn bao nhiêu? Chắc chắn có lẽ nhanh hơn một lượng nào
đó. Có thể. Spec không nói nhanh hơn bao nhiêu, chỉ nói chúng sẽ là
nhanh nhất trên kiến trúc này. Tuy nhiên, đa số compiler C khá tốt,
nên bạn chắc sẽ chỉ thấy cái này được dùng ở chỗ cần đảm bảo tốc độ
tối đa có thể (chứ không chỉ là hy vọng compiler xuất ra code
đủ-nhanh-phết, mà nó đang làm vậy).

Cuối cùng, các kiểu số không dấu này có thêm chữ `u` ở đầu để phân
biệt.

Ví dụ, các kiểu này có nghĩa tương ứng được liệt kê:

[i[`int_leastN_t` types]<]
[i[`uint_leastN_t` types]<]
[i[`int_fastN_t` types]<]
[i[`uint_fastN_t` types]<]
[i[`intN_t` types]<]
[i[`uintN_t` types]<]

``` {.c}
int32_t w;        // w is exactly 32 bits, signed
uint16_t x;       // x is exactly 16 bits, unsigned

int_least8_t y;   // y is at least 8 bits, signed

uint_fast64_t z;  // z is the fastest representation at least 64 bits, unsigned
```

Các kiểu sau được đảm bảo được định nghĩa:

``` {.c}
int_least8_t      uint_least8_t
int_least16_t     uint_least16_t
int_least32_t     uint_least32_t
int_least64_t     uint_least64_t

int_fast8_t       uint_fast8_t
int_fast16_t      uint_fast16_t
int_fast32_t      uint_fast32_t
int_fast64_t      uint_fast64_t
```

Có thể có các kiểu khác với bề rộng khác, nhưng chúng là tuỳ chọn.

Ê! Các kiểu cố định kiểu `int16_t` đâu? Hoá ra chúng hoàn toàn tuỳ
chọn... trừ khi có điều kiện nhất định được thoả^[Cụ thể, hệ có số
nguyên 8, 16, 32, hay 64 bit không padding dùng biểu diễn bù 2,
trong trường hợp đó biến thể `intN_t` cho số bit cụ thể đó _phải_
được định nghĩa.]. Và nếu bạn có hệ máy tính hiện đại trung bình
bình thường, các điều kiện đó khả năng cao được thoả. Và nếu thoả,
bạn sẽ có các kiểu này:

``` {.c}
int8_t      uint8_t
int16_t     uint16_t
int32_t     uint32_t
int64_t     uint64_t
```

Các biến thể khác với bề rộng khác có thể được định nghĩa, nhưng
chúng tuỳ chọn.

[i[`int_leastN_t` types]>]
[i[`uint_leastN_t` types]>]
[i[`int_fastN_t` types]>]
[i[`uint_fastN_t` types]>]
[i[`intN_t` types]>]
[i[`uintN_t` types]>]

## Kiểu số nguyên kích thước tối đa

Có kiểu bạn có thể dùng giữ số nguyên biểu diễn được lớn nhất có sẵn
trên hệ, cả có dấu và không dấu:

[i[`intmax_t` type]<]
[i[`uintmax_t` type]<]

``` {.c}
intmax_t
uintmax_t
```

[i[`intmax_t` type]>]
[i[`uintmax_t` type]>]

Dùng các kiểu này khi bạn muốn đi to hết mức.

Rõ ràng là giá trị từ bất kỳ kiểu số nguyên nào khác cùng dấu sẽ vừa
vào kiểu này, đương nhiên.

## Dùng hằng số kích thước cố định

Nếu bạn có một hằng mà bạn muốn nó vừa vào số bit nhất định, bạn có
thể dùng các macro này để tự động thêm hậu tố đúng vào số (ví dụ
`22L` hay `3490ULL`).

[i[`INTn_C()` macros]<]
[i[`UINTn_C()` macros]<]
[i[`INTMAX_C()` macro]<]
[i[`UINTMAX_C()` macro]<]

``` {.c}
INT8_C(x)     UINT8_C(x)
INT16_C(x)    UINT16_C(x)
INT32_C(x)    UINT32_C(x)
INT64_C(x)    UINT64_C(x)
INTMAX_C(x)   UINTMAX_C(x)
```

[i[`INTn_C()` macros]>]
[i[`UINTMAX_C()` macro]>]

Lại nữa, mấy cái này chỉ chạy với giá trị số nguyên hằng.

Ví dụ, ta có thể dùng một trong số đó để gán giá trị hằng như vầy:

``` {.c}
uint16_t x = UINT16_C(12);
intmax_t y = INTMAX_C(3490);
```

[i[`UINTn_C()` macros]>]
[i[`INTMAX_C()` macro]>]

## Giới hạn của số nguyên kích thước cố định

Ta cũng có một số giới hạn được định nghĩa để bạn lấy được giá trị
lớn nhất và nhỏ nhất cho các kiểu này:

[i[`INTn_MAX` macros]<]
[i[`INTn_MIN` macros]<]
[i[`UINTn_MAX` macros]<]
[i[`INT_LEASTn_MAX` macros]<]
[i[`INT_LEASTn_MIN` macros]<]
[i[`UINT_LEASTn_MAX` macros]<]
[i[`INT_FASTn_MAX` macros]<]
[i[`INT_FASTn_MIN` macros]<]
[i[`UINT_FASTn_MAX` macros]<]
[i[`INTMAX_MAX` macro]<]
[i[`INTMAX_MIN` macro]<]
[i[`UINTMAX_MAX` macro]<]

``` {.c}
INT8_MAX           INT8_MIN           UINT8_MAX
INT16_MAX          INT16_MIN          UINT16_MAX
INT32_MAX          INT32_MIN          UINT32_MAX
INT64_MAX          INT64_MIN          UINT64_MAX

INT_LEAST8_MAX     INT_LEAST8_MIN     UINT_LEAST8_MAX
INT_LEAST16_MAX    INT_LEAST16_MIN    UINT_LEAST16_MAX
INT_LEAST32_MAX    INT_LEAST32_MIN    UINT_LEAST32_MAX
INT_LEAST64_MAX    INT_LEAST64_MIN    UINT_LEAST64_MAX

INT_FAST8_MAX      INT_FAST8_MIN      UINT_FAST8_MAX
INT_FAST16_MAX     INT_FAST16_MIN     UINT_FAST16_MAX
INT_FAST32_MAX     INT_FAST32_MIN     UINT_FAST32_MAX
INT_FAST64_MAX     INT_FAST64_MIN     UINT_FAST64_MAX

INTMAX_MAX         INTMAX_MIN         UINTMAX_MAX
```

[i[`INTn_MAX` macros]>]
[i[`UINTn_MAX` macros]>]
[i[`INT_LEASTn_MAX` macros]>]
[i[`UINT_LEASTn_MAX` macros]>]
[i[`INT_FASTn_MAX` macros]>]
[i[`UINT_FASTn_MAX` macros]>]
[i[`INTMAX_MAX` macro]>]
[i[`UINTMAX_MAX` macro]>]

Lưu ý `MIN` cho mọi kiểu không dấu là `0`, nên như vậy, không có
macro cho nó.

[i[`INTn_MIN` macros]>]
[i[`INT_LEASTn_MIN` macros]>]
[i[`INT_FASTn_MIN` macros]>]
[i[`INTMAX_MIN` macro]>]

## Format specifier

Để in các kiểu này, bạn cần truyền đúng format specifier cho
[i[`printf()` function]] `printf()`. (Và vấn đề tương tự khi lấy
input với [i[`scanf()` function]] `scanf()`.)

Nhưng làm sao bạn biết được kiểu to bao nhiêu dưới mui? May thay,
lần nữa, C cung cấp vài macro để giúp chuyện này.

Tất cả chuyện này có thể thấy trong `<inttypes.h>`.

Giờ, ta có một đống macro. Kiểu một vụ nổ phức tạp của macro. Nên
tôi sẽ thôi liệt ra từng cái và chỉ đặt chữ thường `n` ở chỗ mà bạn
nên đặt `8`, `16`, `32`, hay `64` tuỳ nhu cầu.

Nhìn qua các macro cho in số nguyên có dấu:

[i[`PRIdn` macros]<]
[i[`PRIin` macros]<]
[i[`PRIdLEASTn` macros]<]
[i[`PRIiLEASTn` macros]<]
[i[`PRIdFASTn` macros]<]
[i[`PRIiFASTn` macros]<]
[i[`PRIdMAX` macro]<]
[i[`PRIiMAX` macro]<]

``` {.c}
PRIdn    PRIdLEASTn    PRIdFASTn    PRIdMAX
PRIin    PRIiLEASTn    PRIiFASTn    PRIiMAX
```

Nhìn pattern ở đó. Bạn có thể thấy có biến thể cho kiểu fixed,
least, fast, và max.

Và bạn cũng có chữ thường `d` và chữ thường `i`. Các chữ đó tương
ứng với format specifier `%d` và `%i` của `printf()`.

Nên nếu tôi có thứ kiểu:

``` {.c}
int_least16_t x = 3490;
```

Tôi có thể in cái đó với format specifier tương đương với `%d` bằng
`PRIdLEAST16`.

Nhưng sao? Ta dùng macro đó sao?

Trước hết, macro đó chỉ định một chuỗi chứa chữ cái hay các chữ cái
`printf()` cần dùng để in kiểu đó. Ví dụ, nó có thể là `"d"` hay
`"ld"`.

Nên tất cả ta cần làm là nhúng nó vào chuỗi format của lời gọi
`printf()`.

Để làm chuyện này, ta có thể tận dụng một chuyện về C bạn có thể đã
quên: chuỗi literal kề nhau được nối tự động thành một chuỗi. Ví dụ:

``` {.c}
printf("Hello, " "world!\n");   // Prints "Hello, world!"
```

Và vì các macro này là chuỗi literal, ta có thể dùng chúng như vầy:

``` {.c .numberLines}
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void)
{
    int_least16_t x = 3490;

    printf("The value is %" PRIdLEAST16 "!\n", x);
}
```

[i[`PRIdn` macros]>]
[i[`PRIin` macros]>]
[i[`PRIdLEASTn` macros]>]
[i[`PRIiLEASTn` macros]>]
[i[`PRIdFASTn` macros]>]
[i[`PRIiFASTn` macros]>]
[i[`PRIdMAX` macro]>]
[i[`PRIiMAX` macro]>]

Ta cũng có một đống macro để in kiểu không dấu:

[i[`PRIon` macros]<]
[i[`PRIun` macros]<]
[i[`PRIxn` macros]<]
[i[`PRIXn` macros]<]
[i[`PRIoLEASTn` macros]<]
[i[`PRIuLEASTn` macros]<]
[i[`PRIxLEASTn` macros]<]
[i[`PRIXLEASTn` macros]<]
[i[`PRIoFASTn` macros]<]
[i[`PRIuFASTn` macros]<]
[i[`PRIxFASTn` macros]<]
[i[`PRIXFASTn` macros]<]
[i[`PRIoMAX` macros]<]
[i[`PRIuMAX` macros]<]
[i[`PRIxMAX` macros]<]
[i[`PRIXMAX` macros]<]

``` {.c}
PRIon    PRIoLEASTn    PRIoFASTn    PRIoMAX
PRIun    PRIuLEASTn    PRIuFASTn    PRIuMAX
PRIxn    PRIxLEASTn    PRIxFASTn    PRIxMAX
PRIXn    PRIXLEASTn    PRIXFASTn    PRIXMAX
```

Trong trường hợp này, `o`, `u`, `x`, và `X` tương ứng với các format
specifier đã documented trong `printf()`.

Và, như trước, chữ thường `n` nên được thay bằng `8`, `16`, `32`,
hay `64`.

[i[`PRIon` macros]>]
[i[`PRIun` macros]>]
[i[`PRIxn` macros]>]
[i[`PRIXn` macros]>]
[i[`PRIoLEASTn` macros]>]
[i[`PRIuLEASTn` macros]>]
[i[`PRIxLEASTn` macros]>]
[i[`PRIXLEASTn` macros]>]
[i[`PRIoFASTn` macros]>]
[i[`PRIuFASTn` macros]>]
[i[`PRIxFASTn` macros]>]
[i[`PRIXFASTn` macros]>]
[i[`PRIoMAX` macros]>]
[i[`PRIuMAX` macros]>]
[i[`PRIxMAX` macros]>]
[i[`PRIXMAX` macros]>]

Nhưng ngay khi bạn tưởng mình đã chán macro, hoá ra ta có một bộ
hoàn chỉnh đối ứng cho [i[`scanf()` function]] `scanf()`!

[i[`SCNdn` macros]<]
[i[`SCNin` macros]<]
[i[`SCNon` macros]<]
[i[`SCNun` macros]<]
[i[`SCNxn` macros]<]
[i[`SCNdLEASTn` macros]<]
[i[`SCNiLEASTn` macros]<]
[i[`SCNoLEASTn` macros]<]
[i[`SCNuLEASTn` macros]<]
[i[`SCNxLEASTn` macros]<]
[i[`SCNdFASTn` macros]<]
[i[`SCNiFASTn` macros]<]
[i[`SCNoFASTn` macros]<]
[i[`SCNuFASTn` macros]<]
[i[`SCNxFASTn` macros]<]
[i[`SCNdMAX` macros]<]
[i[`SCNiMAX` macros]<]
[i[`SCNoMAX` macros]<]
[i[`SCNuMAX` macros]<]
[i[`SCNxMAX` macros]<]

``` {.c}
SCNdn    SCNdLEASTn    SCNdFASTn    SCNdMAX
SCNin    SCNiLEASTn    SCNiFASTn    SCNiMAX
SCNon    SCNoLEASTn    SCNoFASTn    SCNoMAX
SCNun    SCNuLEASTn    SCNuFASTn    SCNuMAX
SCNxn    SCNxLEASTn    SCNxFASTn    SCNxMAX
```

[i[`SCNdn` macros]>]
[i[`SCNin` macros]>]
[i[`SCNon` macros]>]
[i[`SCNun` macros]>]
[i[`SCNxn` macros]>]
[i[`SCNdLEASTn` macros]>]
[i[`SCNiLEASTn` macros]>]
[i[`SCNoLEASTn` macros]>]
[i[`SCNuLEASTn` macros]>]
[i[`SCNxLEASTn` macros]>]
[i[`SCNdFASTn` macros]>]
[i[`SCNiFASTn` macros]>]
[i[`SCNoFASTn` macros]>]
[i[`SCNuFASTn` macros]>]
[i[`SCNxFASTn` macros]>]
[i[`SCNdMAX` macros]>]
[i[`SCNiMAX` macros]>]
[i[`SCNoMAX` macros]>]
[i[`SCNuMAX` macros]>]
[i[`SCNxMAX` macros]>]

Nhớ: khi bạn muốn in một kiểu số nguyên kích thước cố định bằng
`printf()` hay `scanf()`, lấy format specifier tương ứng đúng từ
`<inttypes.h>`.

[i[Fixed width integers]>]

<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Function Specifier, Alignment Specifier/Operator

Theo kinh nghiệm của tôi, mấy thứ này không được dùng nhiều lắm,
nhưng cứ trình bày cho đủ.

## Function Specifier

[i[Function specifiers]<]

Khi bạn khai báo một hàm, bạn có thể cho compiler vài gợi ý về cách
hàm đó có thể hay sẽ được dùng. Điều này cho phép hoặc khuyến khích
compiler thực hiện một số tối ưu hoá.

### `inline` để tăng tốc, có lẽ

[i[`inline` function specifier]<]

Bạn có thể khai báo hàm là inline như vầy:

``` {.c}
static inline int add(int x, int y) {
    return x + y;
}
```

Ý nghĩa là khuyến khích compiler làm lời gọi hàm này nhanh nhất có
thể. Và trong lịch sử, một cách để làm điều đó là _inlining_, tức là
thân hàm sẽ được nhúng nguyên vẹn tại nơi gọi. Cái này tránh tất cả
overhead set up lời gọi hàm và tháo dỡ nó, đổi lại kích thước code
lớn hơn vì hàm được copy khắp nơi thay vì tái sử dụng.

Những điều nhanh-gọn cần nhớ:

1. Bạn có lẽ không cần dùng `inline` để tăng tốc. Compiler hiện đại
   biết cái gì tốt nhất.

2. Nếu bạn dùng nó để tăng tốc, dùng với phạm vi file, tức là
   `static inline`. Cái này tránh quy tắc lộn xộn của external
   linkage và hàm inline.

Đừng đọc phần này nữa.

Kẻ thèm bị trừng phạt hả?

Thử bỏ `static` đi nào.

``` {.c .numberLines}
#include <stdio.h>

inline int add(int x, int y)
{
    return x + y;
}

int main(void)
{
    printf("%d\n", add(1, 2));
}
```

`gcc` báo lỗi linker trên `add()`^[Trừ khi bạn compile có bật
optimization (có lẽ)! Nhưng tôi nghĩ khi nó làm vậy, nó không tuân
spec.]. Spec yêu cầu nếu bạn có một hàm inline không-`extern` thì
bạn cũng phải cung cấp một phiên bản có external linkage.

Nên bạn sẽ phải có phiên bản `extern` ở đâu đó khác để cái này chạy.
Nếu compiler có cả hàm `inline` trong file hiện tại và phiên bản
external của cùng hàm ở nơi khác, nó được chọn gọi cái nào. Nên tôi
khuyên mạnh là chúng giống nhau.

Một cách khác bạn có thể làm là khai báo hàm là `extern inline`.
Cái này sẽ thử inline trong cùng file (để tăng tốc), nhưng cũng tạo
phiên bản có external linkage.

[i[`inline` function specifier]>]

### `noreturn` và `_Noreturn` {#noreturn}

[i[`noreturn` function specifier]<]
[i[`_Noreturn` function specifier]<]

Cái này báo cho compiler rằng một hàm cụ thể sẽ không bao giờ
return về chỗ gọi, tức là chương trình sẽ thoát bằng cơ chế nào đó
trước khi hàm return.

Nó cho phép compiler có thể thực hiện một số tối ưu quanh lời gọi
hàm.

Nó cũng cho phép bạn báo cho các dev khác rằng có logic chương
trình phụ thuộc vào một hàm _không_ return.

Có lẽ bạn sẽ không bao giờ cần dùng cái này, nhưng bạn sẽ thấy nó
trên một số lời gọi thư viện như
[fl[`exit()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-exit]]
và
[fl[`abort()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-abort]].

Từ khoá có sẵn là `_Noreturn`, nhưng nếu nó không làm hỏng code sẵn
có của bạn, mọi người đều khuyên include `<stdnoreturn.h>` và dùng
`noreturn` dễ đọc hơn.

Là hành vi không xác định nếu một hàm được chỉ định là `noreturn`
thực sự return. Cái đó không trung thực về mặt tính toán, thấy đó.

Đây là ví dụ dùng `noreturn` đúng:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <stdnoreturn.h>

noreturn void foo(void) // This function should never return!
{
    printf("Happy days\n");

    exit(1);            // And it doesn't return--it exits here!
}

int main(void)
{
    foo();
}
```

Nếu compiler phát hiện một hàm `noreturn` có thể return, nó có thể
cảnh báo bạn, hữu ích.

Thay thế hàm `foo()` bằng cái này:

``` {.c}
noreturn void foo(void)
{
    printf("Breakin' the law\n");
}
```

cho tôi một cảnh báo:

``` {.default}
foo.c:7:1: warning: function declared 'noreturn' should not return
```

[i[`noreturn` function specifier]>]
[i[`_Noreturn` function specifier]>]
[i[Function specifiers]>]

## Alignment Specifier và Operator

[i[Alignment]<]

[flw[_Alignment_|Data_structure_alignment]] là về các bội số của
địa chỉ mà các đối tượng có thể được lưu. Bạn có thể lưu cái này ở
địa chỉ bất kỳ? Hay phải là địa chỉ bắt đầu chia hết cho 2? Hay 8?
Hay 16?

Nếu bạn đang code thứ gì đó thấp cấp như bộ cấp phát bộ nhớ giao
tiếp với OS, bạn có thể cần nghĩ đến điều này. Phần lớn dev đi hết
sự nghiệp mà không dùng chức năng này trong C.

### `alignas` và `_Alignas`

[i[`alignas` alignment specifier]<]
[i[`_Alignas` alignment specifier]<]

Cái này không phải hàm. Đây là một _alignment specifier_ bạn có thể
dùng với một khai báo biến.

Specifier có sẵn là `_Alignas`, nhưng header `<stdalign.h>` định
nghĩa nó là `alignas` cho đẹp hơn.

Nếu bạn cần `char` của mình được căn chỉnh như `int`, bạn có thể ép
như vầy khi khai báo:

``` {.c}
char alignas(int) c;
```

Bạn cũng có thể truyền giá trị hằng hay biểu thức vào làm alignment.
Cái này phải là thứ được hệ thống hỗ trợ, nhưng spec ngừng trước
chuyện quy định bạn có thể đưa giá trị nào vào. Các luỹ thừa nhỏ
của 2 (1, 2, 4, 8, và 16) nhìn chung là đặt cược an toàn.

``` {.c}
char alignas(8) c;   // align on 8-byte boundaries
```

Nếu bạn muốn căn chỉnh ở alignment lớn nhất hệ thống bạn dùng,
include `<stddef.h>` và dùng kiểu `max_align_t`, như sau:

``` {.c}
char alignas(max_align_t) c;
```

Bạn có thể _over-align_ bằng cách chỉ định alignment lớn hơn của
`max_align_t`, nhưng chuyện đó có được phép hay không phụ thuộc hệ
thống.

[i[`alignas` alignment specifier]>]
[i[`_Alignas` alignment specifier]>]

### `alignof` và `_Alignof`

[i[`alignof` operator]<]
[i[`_Alignof` operator]<]

Toán tử này sẽ trả về bội số địa chỉ mà một kiểu cụ thể dùng cho
alignment trên hệ thống này. Ví dụ, có thể `char` được căn chỉnh
mỗi 1 địa chỉ, và `int` được căn chỉnh mỗi 4 địa chỉ.

Toán tử có sẵn là `_Alignof`, nhưng header `<stdalign.h>` định
nghĩa nó là `alignof` nếu bạn muốn trông chất hơn.

Đây là chương trình sẽ in ra alignment của nhiều kiểu khác nhau.
Lại nữa, chúng sẽ thay đổi từ hệ thống này sang hệ thống khác. Chú
ý kiểu `max_align_t` sẽ cho bạn alignment lớn nhất hệ thống dùng.

``` {.c .numberLines}
#include <stdalign.h>
#include <stdio.h>     // for printf()
#include <stddef.h>    // for max_align_t

struct t {
    int a;
    char b;
    float c;
};

int main(void)
{
    printf("char       : %zu\n", alignof(char));
    printf("short      : %zu\n", alignof(short));
    printf("int        : %zu\n", alignof(int));
    printf("long       : %zu\n", alignof(long));
    printf("long long  : %zu\n", alignof(long long));
    printf("double     : %zu\n", alignof(double));
    printf("long double: %zu\n", alignof(long double));
    printf("struct t   : %zu\n", alignof(struct t));
    printf("max_align_t: %zu\n", alignof(max_align_t));
}
```

Output trên hệ của tôi:

``` {.default}
char       : 1
short      : 2
int        : 4
long       : 8
long long  : 8
double     : 8
long double: 16
struct t   : 16
max_align_t: 16
```

[i[`alignof` operator]>]
[i[`_Alignof` operator]>]

## Hàm `memalignment()`

[i[`memalignment()` function]<]

Mới trong C23!

(Lưu ý: không compiler nào của tôi hỗ trợ hàm này chưa, nên code
chủ yếu chưa được test.)

`alignof` hay nếu bạn biết kiểu dữ liệu. Nhưng nếu bạn _ngu dốt
đáng thương_ về kiểu, và chỉ có con trỏ tới dữ liệu?

Sao điều đó xảy ra được?

À, với người bạn tốt `void*` của ta, tất nhiên. Ta không thể truyền
cái đó cho `alignof`, nhưng nếu ta cần biết alignment của thứ nó
trỏ tới?

Ta có thể muốn biết điều này nếu ta sắp dùng bộ nhớ cho thứ gì đó
có nhu cầu alignment đáng kể. Ví dụ, kiểu atomic và floating thường
hành xử xấu nếu không căn chỉnh đúng.

Với hàm này ta có thể kiểm tra alignment của một số dữ liệu miễn là
có con trỏ tới dữ liệu đó, ngay cả khi là `void*`.

Hãy làm một test nhanh xem một void pointer có được căn chỉnh tốt
để dùng như kiểu atomic hay không, và nếu có, lấy một biến dùng
như kiểu đó:

``` {.c}
void foo(void *p)
{
    if (memalignment(p) >= alignof(atomic int)) {
        atomic int *i = p;
        do_things(i);
    } else
        puts("This pointer is no good as an atomic int\n");

...
```

Tôi ngờ bạn sẽ hiếm khi (đến mức không bao giờ, có lẽ) cần dùng hàm
này trừ khi bạn đang làm thứ gì đó thấp cấp.

[i[`memalignment()` function]>]

Và xong! Alignment!

[i[Alignment]>]

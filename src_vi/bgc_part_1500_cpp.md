<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# C Preprocessor

[i[Preprocessor]<]

Trước khi chương trình được biên dịch, nó thật ra chạy qua một pha
gọi là _preprocessing_. Gần như là có một ngôn ngữ _nằm trên_ ngôn
ngữ C chạy trước. Và nó xuất ra code C, rồi code đó mới được
biên dịch.

Ta đã thấy chuyện này phần nào với `#include`! Đó là C Preprocessor!
Chỗ nào nó thấy directive đó, nó nhét file được gọi tên ngay vào,
y như bạn đã gõ vào đó. Và _rồi_ compiler mới build cả mớ.

Nhưng hóa ra nó mạnh hơn nhiều so với chỉ biết include. Bạn có thể
định nghĩa _macro_ để được thay thế... và thậm chí cả macro nhận
tham số!

## `#include`

[i[`#include` directive]<]

Bắt đầu với cái ta đã thấy nhiều lần. Đây dĩ nhiên là một cách để
include các nguồn khác vào nguồn của bạn. Rất hay dùng với file
header.

Dù spec cho phép `#include` hành xử đủ kiểu, ta sẽ chọn cách thực
dụng hơn và nói về cách nó hoạt động trên mọi hệ tôi từng gặp.

Ta có thể chia file header thành hai loại: hệ thống và nội bộ. Những
thứ dựng sẵn như `stdio.h`, `stdlib.h`, `math.h` và v.v., bạn có
thể include bằng ngoặc nhọn:

``` {.c}
#include <stdio.h>
#include <stdlib.h>
```

Ngoặc nhọn nói với C: "Này, đừng tìm file header này ở thư mục hiện
tại, tìm trong thư mục include chung của hệ thống kìa."

[i[`#include` directive-->local files]<]

Cái đó, dĩ nhiên, ngầm bảo rằng phải có cách để include file nội bộ
từ thư mục hiện tại. Và có: dùng nháy kép:

``` {.c}
#include "myheader.h"
```

Hoặc rất có thể bạn tìm được ở thư mục tương đối bằng dấu gạch chéo
xuôi và dấu chấm, kiểu thế này:

``` {.c}
#include "mydir/myheader.h"
#include "../someheader.py"
```

Đừng dùng dấu gạch chéo ngược (`\`) làm dấu ngăn đường dẫn trong
`#include`! Đó là undefined behavior! Chỉ dùng gạch chéo xuôi (`/`),
ngay cả trên Windows.

Tóm lại, dùng ngoặc nhọn (`<` và `>`) cho include hệ thống, và dùng
nháy kép (`"`) cho include cá nhân của bạn.

[i[`#include` directive-->local files]>]
[i[`#include` directive]>]

## Macro đơn giản

[i[Preprocessor-->macros]<]

_Macro_ là một định danh được _mở rộng_ thành một mẩu code khác
trước khi compiler thấy được. Hãy coi nó như một chỗ trống, khi
preprocessor thấy một trong các định danh đó, nó thay bằng giá trị
mà bạn đã định nghĩa.

[i[`#define` directive]<]

Ta làm chuyện này bằng `#define` (thường đọc là "pound define", hay
"hash define", và hiếm khi, nếu có, đọc là "octothorpe define"). Ví
dụ:

``` {.c .numberLines}
#include <stdio.h>

#define HELLO "Hello, world"
#define PI 3.14159

int main(void)
{
    printf("%s, %f\n", HELLO, PI);
}
```

Ở dòng 3 và 4 ta định nghĩa vài macro. Ở đâu mà chúng xuất hiện
trong code (dòng 8), chúng sẽ được thay bằng giá trị đã định nghĩa.

Từ góc nhìn của compiler C, y hệt như ta đã viết thế này:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    printf("%s, %f\n", "Hello, world", 3.14159);
}
```

Thấy cách `HELLO` được thay bằng `"Hello, world"` và `PI` được thay
bằng `3.14159` chứ? Từ góc nhìn compiler, giống như những giá trị
đó đã có mặt ngay trong code.

Lưu ý rằng macro không có kiểu cụ thể, _tự thân nó_. Thật ra chỉ là
chúng được thay nguyên xi bằng bất cứ thứ gì được `#define` thành.
Nếu code C kết quả không hợp lệ, compiler sẽ phun bậy ra.

Bạn cũng có thể định nghĩa macro không có giá trị:

``` {.c}
#define EXTRA_HAPPY
```

trong trường hợp đó, macro tồn tại và được định nghĩa, nhưng định
nghĩa là không có gì. Nên chỗ nào nó xuất hiện trong văn bản cũng
sẽ được thay bằng không có gì. Ta sẽ thấy cách dùng của cái này
sau.

Theo quy ước, tên macro viết `ALL_CAPS` dù kỹ thuật thì không bắt
buộc.

[i[`#define` directive-->versus `const`]<]

Nhìn chung, cái này cho bạn cách định nghĩa giá trị hằng gần như
toàn cục và dùng được _bất cứ_ chỗ nào. Kể cả chỗ mà biến `const`
không dùng được, ví dụ trong `case` của `switch` và độ dài mảng cố
định.

Dù vậy, cuộc tranh cãi vẫn nổ ra trên mạng về việc dùng biến `const`
có kiểu có tốt hơn macro `#define` trong trường hợp chung không.

Nó cũng có thể được dùng để thay thế hoặc sửa các từ khóa, một khái
niệm hoàn toàn xa lạ với `const`, dù chuyện này nên dùng tiết kiệm.

[i[`#define` directive-->versus `const`]>]
[i[`#define` directive]>]
[i[Preprocessor-->macros]>]

## Biên dịch có điều kiện

[i[Conditional compilation]<]

Có thể bắt preprocessor quyết định có đưa một số block code cho
compiler hay không, hoặc xóa chúng hẳn đi trước khi compile.

Ta làm chuyện đó bằng cách bọc code trong các block điều kiện, giống
như câu lệnh `if`-`else`.

### Nếu đã định nghĩa, `#ifdef` và `#endif`

Đầu tiên, thử compile code cụ thể tùy thuộc vào macro có được định
nghĩa hay không.

[i[`#ifdef` directive]<]
[i[`#endif` directive]<]

``` {.c .numberLines}
#include <stdio.h>

#define EXTRA_HAPPY

int main(void)
{

#ifdef EXTRA_HAPPY
    printf("I'm extra happy!\n");
#endif

    printf("OK!\n");
}
```

Trong ví dụ đó, ta định nghĩa `EXTRA_HAPPY` (thành không gì cả, nhưng
nó _được_ định nghĩa), rồi ở dòng 8 ta kiểm tra xem nó có được định
nghĩa hay không bằng directive `#ifdef`. Nếu có, code tiếp sau đó sẽ
được đưa vào cho đến `#endif`.

[i[`#ifdef` directive]>]
[i[`#endif` directive]>]

Vì nó được định nghĩa, code sẽ được đưa vào để compile và output sẽ
là:

``` {.default}
I'm extra happy!
OK!
```

Nếu ta comment cái `#define` đi, kiểu:

``` {.c}
//#define EXTRA_HAPPY
```

thì nó sẽ không được định nghĩa, và code sẽ không được đưa vào
compile. Và output sẽ chỉ là:

``` {.default}
OK!
```

Quan trọng là nhớ rằng những quyết định này xảy ra lúc compile! Code
thực sự được compile hoặc bị bỏ đi tùy vào điều kiện. Chuyện này
khác với lệnh `if` tiêu chuẩn được đánh giá lúc chương trình chạy.

### Nếu chưa định nghĩa, `#ifndef`

Cũng có nghĩa phủ định của "nếu đã định nghĩa": "nếu chưa định
nghĩa", hay `#ifndef`. Ta có thể sửa ví dụ trước để in ra thứ khác
dựa trên việc một thứ có được định nghĩa hay không:

[i[`#ifndef` directive]<]
[i[`#endif` directive]<]

``` {.c .numberLines startFrom="8"}
#ifdef EXTRA_HAPPY
    printf("I'm extra happy!\n");
#endif

#ifndef EXTRA_HAPPY
    printf("I'm just regular\n");
#endif
```

Ta sẽ thấy cách gọn hơn để làm ở phần sau.

Nối lại với file header, ta đã thấy cách khiến file header chỉ được
include đúng một lần bằng cách bọc chúng trong directive
preprocessor thế này:

``` {.c}
#ifndef MYHEADER_H  // First line of myheader.h
#define MYHEADER_H

int x = 12;

#endif  // Last line of myheader.h
```

[i[`#ifndef` directive]>]
[i[`#endif` directive]>]

Cái này cho thấy cách một macro tồn tại qua các file và qua nhiều
`#include`. Nếu chưa được định nghĩa, ta định nghĩa nó rồi compile
cả file header.

Nhưng lần sau khi nó được include, ta thấy `MYHEADER_H` _đã_ được
định nghĩa, nên ta không gửi file header cho compiler nữa, nó bị
xóa hẳn đi.


### `#else`

[i[`#else` directive]<]

Nhưng chưa phải tất cả! Còn có `#else` mà ta có thể quẳng vào mớ đó.

[i[`#ifdef` directive]<]
[i[`#endif` directive]<]

Sửa ví dụ trước:

``` {.c .numberLines startFrom="8"}
#ifdef EXTRA_HAPPY
    printf("I'm extra happy!\n");
#else
    printf("I'm just regular\n");
#endif
```

[i[`#ifdef` directive]>]
[i[`#else` directive]>]
[i[`#endif` directive]>]

Giờ nếu `EXTRA_HAPPY` chưa được định nghĩa, nó sẽ trúng nhánh `#else`
và in:

``` {.default}
I'm just regular
```

### Else-If: `#elifdef`, `#elifndef`

[i[`#elifdef` directive]<]
[i[`#elifndef` directive]<]

Tính năng này mới có ở C23!

Thế nếu bạn muốn cái gì phức tạp hơn? Có thể bạn cần cấu trúc if-else
nối tầng để code được build đúng?

May là ta có mấy directive này để xài. Ta có thể dùng `#elifdef` cho
"else if defined":

``` {.c}
#ifdef MODE_1
    printf("This is mode 1\n");
#elifdef MODE_2
    printf("This is mode 2\n");
#elifdef MODE_3
    printf("This is mode 3\n");
#else
    printf("This is some other mode\n");
#endif
```

[i[`#elifdef` directive]>]

Ngược lại, bạn có thể dùng `#elifndef` cho "else if not defined".

[i[`#elifndef` directive]>]

### Điều kiện tổng quát: `#if`, `#elif`

[i[`#if` directive]<]
[i[`#elif` directive]<]

Cái này hoạt động rất giống `#ifdef` và `#ifndef` ở chỗ bạn cũng có
thể có `#else` và cả mớ được khép lại bằng `#endif`.

Khác biệt duy nhất là biểu thức hằng sau `#if` phải tính ra true
(khác không) thì code trong `#if` mới được compile. Nên thay vì việc
một thứ có được định nghĩa hay chưa, ta muốn một biểu thức tính ra
true.

``` {.c .numberLines}
#include <stdio.h>

#define HAPPY_FACTOR 1

int main(void)
{

#if HAPPY_FACTOR == 0
    printf("I'm not happy!\n");
#elif HAPPY_FACTOR == 1
    printf("I'm just regular\n");
#else
    printf("I'm extra happy!\n");
#endif

    printf("OK!\n");
}
```

[i[`#elif` directive]>]

Lại một lần nữa, với các nhánh `#if` không khớp, compiler sẽ không
nhìn thấy những dòng đó. Với code trên, sau khi preprocessor làm
xong, tất cả compiler thấy chỉ là:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{

    printf("I'm just regular\n");

    printf("OK!\n");
}
```

Một cách dùng hơi hacker là để comment nhanh một mớ dòng^[Bạn không
thể lúc nào cũng bọc code trong comment `/*` `*/` vì chúng không
lồng được.].

[i[`#if 0` directive]<]

Nếu bạn đặt `#if 0` ("if false") ở đầu block cần comment và `#endif`
ở cuối, bạn được hiệu quả này:

``` {.c}
#if 0
    printf("All this code"); /* is effectively */
    printf("commented out"); // by the #if 0
#endif
```

[i[`#if 0` directive]>]

Thế nếu bạn trên compiler cũ hơn C23 và không có hỗ trợ directive
`#elifdef` hay `#elifndef` thì sao? Làm sao đạt cùng hiệu quả với
`#if`? Ví dụ, nếu tôi muốn cái này:

``` {.c}
#ifdef FOO
    x = 2;
#elifdef BAR  // POTENTIAL ERROR: Not supported before C23
    x = 3;
#endif
```

Làm sao làm được?

Hóa ra có một toán tử preprocessor tên `defined` mà ta có thể dùng
với lệnh `#if`.

Các thứ sau là tương đương:

[i[`#if defined` directive]<]
``` {.c}
#ifdef FOO
#if defined FOO
#if defined(FOO)   // Parentheses optional
```

Các thứ sau cũng vậy:

``` {.c}
#ifndef FOO
#if !defined FOO
#if !defined(FOO)   // Parentheses optional
```

Chú ý cách ta dùng toán tử NOT logic tiêu chuẩn (`!`) cho "chưa định
nghĩa".

Vậy giờ ta quay lại vùng đất `#if` và có thể dùng `#elif` mà không
phải sợ gì!

Đoạn code hỏng này:

``` {.c}
#ifdef FOO
    x = 2;
#elifdef BAR  // POTENTIAL ERROR: Not supported before C23
    x = 3;
#endif
```

có thể được thay bằng:

``` {.c}
#if defined FOO
    x = 2;
#elif defined BAR
    x = 3;
#endif
```

[i[`#if defined` directive]>]
[i[`#if` directive]>]
[i[Conditional compilation]>]

### Vứt macro đi: `#undef`

[i[`#undef` directive]<]

Nếu bạn đã định nghĩa gì đó nhưng không cần nữa, bạn có thể bỏ định
nghĩa bằng `#undef`.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
#define GOATS

#ifdef GOATS
    printf("Goats detected!\n");  // prints
#endif

#undef GOATS  // Make GOATS no longer defined

#ifdef GOATS
    printf("Goats detected, again!\n"); // doesn't print
#endif
}
```

[i[`#undef` directive]>]

## Macro dựng sẵn

[i[Preprocessor-->predefined macros]<]

Chuẩn định nghĩa một đống macro dựng sẵn mà bạn có thể kiểm tra và
dùng cho biên dịch có điều kiện. Xem qua ở đây.


### Macro bắt buộc

Tất cả những cái này đều được định nghĩa:

[i[`__DATE__` macro]<]
[i[`__TIME__` macro]<]
[i[`__FILE__` macro]<]
[i[`__LINE__` macro]<]
[i[`__func__` identifier]<]
[i[`__STDC_VERSION__` macro]<]

[i[`__STDC__` macro]]
[i[`__STDC_HOSTED__` macro]]

|Macro|Mô tả|
|-----------|----------------------------------------------------|
|`__DATE__`|Ngày compile, kiểu lúc bạn đang compile file này, ở định dạng `Mmm dd yyyy`|
|`__TIME__`|Giờ compile ở định dạng `hh:mm:ss`|
|`__FILE__`|Một chuỗi chứa tên file này|
|`__LINE__`|Số dòng của file mà macro này xuất hiện ở đó|
|`__func__`|Tên hàm mà cái này xuất hiện trong, dưới dạng chuỗi^[Đây không hẳn là macro, về kỹ thuật thì nó là một định danh. Nhưng nó là định danh được định nghĩa trước duy nhất và cảm giác rất giống macro, nên tôi để nó ở đây. Kiểu nổi loạn tí.]|
|`__STDC__`|Được định nghĩa bằng `1` nếu đây là compiler C chuẩn|
|`__STDC_HOSTED__`|Sẽ là `1` nếu compiler là _hosted implementation_^[Hosted implementation về cơ bản nghĩa là bạn đang chạy chuẩn C đầy đủ, có lẽ trên một hệ điều hành nào đó. Mà chắc là đúng thế. Nếu bạn đang chạy trên phần cứng trần trong kiểu hệ embedded, bạn chắc đang trên _standalone implementation_.], ngược lại `0`|
|`__STDC_VERSION__`|Phiên bản của C, một hằng `long int` ở dạng `yyyymmL`, ví dụ `201710L`|

Ghép chúng lại.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    printf("This function: %s\n", __func__);
    printf("This file: %s\n", __FILE__);
    printf("This line: %d\n", __LINE__);
    printf("Compiled on: %s %s\n", __DATE__, __TIME__);
    printf("C Version: %ld\n", __STDC_VERSION__);
}
```

[i[`__DATE__` macro]>]
[i[`__TIME__` macro]>]

Output trên hệ của tôi là:

``` {.default}
This function: main
This file: foo.c
This line: 7
Compiled on: Nov 23 2020 17:16:27
C Version: 201710
```

`__FILE__`, `__func__` và `__LINE__` đặc biệt hữu ích để báo tình
trạng lỗi trong thông điệp cho dev. Macro `assert()` trong
`<assert.h>` dùng chúng để chỉ ra chỗ nào trong code assertion thất
bại.

[i[`__FILE__` macro]>]
[i[`__LINE__` macro]>]
[i[`__func__` identifier]>]

#### `__STDC_VERSION__`

[i[Language versions]<]

Nếu bạn tò mò, đây là các số phiên bản cho những bản phát hành lớn
khác nhau của Spec Ngôn ngữ C:

|Release|Phiên bản ISO/IEC|`__STDC_VERSION__`|
|-|-|-|
|C89|ISO/IEC 9899:1990|không định nghĩa|
|**C89**|ISO/IEC 9899:1990/Amd.1:1995|`199409L`|
|**C99**|ISO/IEC 9899:1999|`199901L`|
|**C11**|ISO/IEC 9899:2011/Amd.1:2012|`201112L`|

Chú ý macro này ban đầu không tồn tại trong C89.

Cũng chú ý rằng kế hoạch là số phiên bản sẽ tăng nghiêm ngặt, nên
bạn luôn có thể kiểm tra, chẳng hạn "ít nhất C99" bằng:

``` {.c}
#if __STDC_VERSION__ >= 1999901L
```

[i[Language versions]>]
[i[`__STDC_VERSION__` macro]>]

### Macro tùy chọn

Implementation của bạn có thể cũng định nghĩa những cái này. Hoặc
không.

[i[`__STDC_ISO_10646__` macro]]
[i[`__STDC_MB_MIGHT_NEQ_WC__` macro]]
[i[`__STDC_UTF_16__` macro]]
[i[`__STDC_UTF_32__` macro]]
[i[`__STDC_ANALYZABLE__` macro]]
[i[`__STDC_IEC_559__` macro]]
[i[`__STDC_IEC_559_COMPLEX__` macro]]
[i[`__STDC_LIB_EXT1__` macro]]
[i[`__STDC_NO_ATOMICS__` macro]]
[i[`__STDC_NO_COMPLEX__` macro]]
[i[`__STDC_NO_THREADS__` macro]]
[i[`__STDC_NO_VLA__` macro]]

|Macro|Mô tả|
|----------------|--------------------------------------------------|
|`__STDC_ISO_10646__`|Nếu được định nghĩa, `wchar_t` chứa giá trị Unicode, ngược lại thì chứa gì khác|
|`__STDC_MB_MIGHT_NEQ_WC__`|Giá trị `1` báo rằng các giá trị trong ký tự đa byte có thể không ánh xạ bằng với giá trị trong ký tự wide|
|`__STDC_UTF_16__`|Giá trị `1` báo rằng hệ thống dùng mã hóa UTF-16 trong kiểu `char16_t`|
|`__STDC_UTF_32__`|Giá trị `1` báo rằng hệ thống dùng mã hóa UTF-32 trong kiểu `char32_t`|
|`__STDC_ANALYZABLE__`|Giá trị `1` báo code có thể phân tích được^[Được, tôi biết đó là câu trả lời né tránh. Về cơ bản có một phần mở rộng tùy chọn mà compiler có thể cài, trong đó chúng đồng ý giới hạn vài kiểu undefined behavior để code C dễ làm static code analysis hơn. Ít khả năng bạn cần dùng cái này.]|
|`__STDC_IEC_559__`|`1` nếu hỗ trợ floating point IEEE-754 (còn gọi là IEC 60559)|
|`__STDC_IEC_559_COMPLEX__`|`1` nếu hỗ trợ floating point phức IEC 60559|
|`__STDC_LIB_EXT1__`|`1` nếu implementation này hỗ trợ nhiều hàm thư viện chuẩn "an toàn" thay thế (chúng có hậu tố `_s` trong tên)|
|`__STDC_NO_ATOMICS__`|`1` nếu implementation này **không** hỗ trợ `_Atomic` hay `<stdatomic.h>`|
|`__STDC_NO_COMPLEX__`|`1` nếu implementation này **không** hỗ trợ kiểu complex hay `<complex.h>`|
|`__STDC_NO_THREADS__`|`1` nếu implementation này **không** hỗ trợ `<threads.h>`|
|`__STDC_NO_VLA__`|`1` nếu implementation này **không** hỗ trợ mảng có độ dài thay đổi|

[i[Preprocessor-->predefined macros]>]

## Macro có tham số

[i[Preprocessor-->macros with arguments]<]

Macro mạnh hơn là chỉ thay thế đơn giản. Bạn có thể cài chúng nhận
tham số để thay vào.

Câu hỏi thường nảy ra là khi nào dùng macro có tham số thay cho hàm.
Trả lời ngắn: dùng hàm. Nhưng bạn sẽ thấy vô số macro ngoài đời và
trong thư viện chuẩn. Người ta hay dùng chúng cho mấy thứ ngắn, tính
toán, và cho các tính năng có thể thay đổi tùy nền tảng. Bạn có thể
định nghĩa các từ khóa khác nhau cho nền tảng này hay nền tảng khác.

### Macro có một tham số

Bắt đầu với cái đơn giản bình phương một số:

[i[`#define` directive]<]

``` {.c .numberLines}
#include <stdio.h>

#define SQR(x) x * x  // Not quite right, but bear with me

int main(void)
{
    printf("%d\n", SQR(12));  // 144
}
```

Cái đó nói "ở đâu thấy `SQR` với giá trị nào đó, thay bằng giá trị
đó nhân với chính nó".

Nên dòng 7 sẽ đổi thành:

``` {.c .numberLines startFrom="7"}
    printf("%d\n", 12 * 12);  // 144
```

C thoải mái chuyển thành 144.

Nhưng ta đã phạm lỗi sơ đẳng trong macro đó, lỗi mà ta cần tránh.

Cùng xem. Nếu ta muốn tính `SQR(3 + 4)`? Ừ, $3+4=7$, nên ta chắc
muốn tính $7^2=49$. Đấy, `49`, câu trả lời cuối cùng.

Cho vào code và ta được... 19?

``` {.c .numberLines startFrom="7"}
    printf("%d\n", SQR(3 + 4));  // 19!!??
```

Có chuyện gì?

Nếu ta theo dõi mở rộng macro, ta có 

``` {.c .numberLines startFrom="7"}
    printf("%d\n", 3 + 4 * 3 + 4);  // 19!
```

Ối! Vì nhân có độ ưu tiên cao hơn, ta làm $4\times3=12$ trước, rồi
được $3+12+4=19$. Không phải thứ ta muốn.

Nên ta phải sửa cho nó đúng.

**Cái này phổ biến tới mức cứ tự động làm mỗi khi bạn tạo macro tính
toán có tham số!**

Sửa thì dễ: thêm vài cặp ngoặc đơn!

``` {.c .numberLines startFrom="3"}
#define SQR(x) (x) * (x)   // Better... but still not quite good enough!
```

Và giờ macro của ta mở rộng thành:

``` {.c .numberLines startFrom="7"}
    printf("%d\n", (3 + 4) * (3 + 4));  // 49! Woo hoo!
```

Nhưng thật ra ta vẫn còn cùng vấn đề, có thể thò ra nếu quanh đó có
toán tử ưu tiên cao hơn nhân (`*`).

Nên cách an toàn, đúng bài để ghép macro là bọc cả cục trong thêm
cặp ngoặc đơn, thế này:

``` {.c .numberLines startFrom="3"}
#define SQR(x) ((x) * (x))   // Good!
```

Cứ biến nó thành thói quen khi tạo macro tính toán là không sai
được.

### Macro có nhiều hơn một tham số

Bạn có thể xếp chồng mấy thứ này lên bao nhiêu tùy ý:

``` {.c}
#define TRIANGLE_AREA(w, h) (0.5 * (w) * (h))
```

Làm vài macro giải $x$ dùng công thức nghiệm bậc hai. Phòng khi bạn
không còn nhớ nằm lòng, với phương trình dạng:

$ax^2+bx+c=0$

bạn có thể giải $x$ bằng công thức nghiệm bậc hai:

$x=\displaystyle\frac{-b\pm\sqrt{b^2-4ac}}{2a}$

Cái đó điên rồ. Để ý cả cái dấu cộng-trừ ($\pm$) trong đó, báo rằng
thật ra có hai nghiệm.

Nên ta làm macro cho cả hai:

``` {.c}
#define QUADP(a, b, c) ((-(b) + sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
#define QUADM(a, b, c) ((-(b) - sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
```

Vậy là có được tí tính toán. Nhưng ta định nghĩa thêm một cái nữa
để dùng làm tham số cho `printf()` in cả hai đáp án.

``` {.c}
//          macro              replacement
//      |-----------| |----------------------------|
#define QUAD(a, b, c) QUADP(a, b, c), QUADM(a, b, c)
```

Đó chỉ là vài giá trị ngăn cách bởi dấu phẩy, và ta có thể dùng cái
đó như tham số "ghép" của `printf()` thế này:

``` {.c}
printf("x = %f or x = %f\n", QUAD(2, 10, 5));
```

Ghép lại thành code:

``` {.c .numberLines}
#include <stdio.h>
#include <math.h>  // For sqrt()

#define QUADP(a, b, c) ((-(b) + sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
#define QUADM(a, b, c) ((-(b) - sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
#define QUAD(a, b, c) QUADP(a, b, c), QUADM(a, b, c)

int main(void)
{
    printf("2*x^2 + 10*x + 5 = 0\n");
    printf("x = %f or x = %f\n", QUAD(2, 10, 5));
}
```

Và cái này cho output:

``` {.default}
2*x^2 + 10*x + 5 = 0
x = -0.563508 or x = -4.436492
```

Thế giá trị nào vào cũng cho xấp xỉ không (hơi lệch vì các số không
chính xác):

$2\times-0.563508^2+10\times-0.563508+5\approx0.000003$

### Macro với tham số biến

[i[Preprocessor-->macros with variable arguments]<]

Cũng có cách truyền số lượng tham số biến đổi vào macro, dùng dấu
ba chấm (`...`) sau các tham số có tên đã biết. Khi macro mở rộng,
mọi tham số thừa sẽ ở trong danh sách ngăn bởi dấu phẩy trong macro
`__VA_ARGS__`, và có thể được thay từ đó:

``` {.c .numberLines}
#include <stdio.h>

// Combine the first two arguments to a single number,
// then have a commalist of the rest of them:

#define X(a, b, ...) (10*(a) + 20*(b)), __VA_ARGS__

int main(void)
{
    printf("%d %f %s %d\n", X(5, 4, 3.14, "Hi!", 12));
}
```

Việc thay thế diễn ra ở dòng 10 sẽ là:

``` {.c .numberLines startFrom="10"}
    printf("%d %f %s %d\n", (10*(5) + 20*(4)), 3.14, "Hi!", 12);
```

cho output:

``` {.default}
130 3.140000 Hi! 12
```

Bạn cũng có thể "stringify" `__VA_ARGS__` bằng cách đặt `#` ở trước:

``` {.c}
#define X(...) #__VA_ARGS__

printf("%s\n", X(1,2,3));  // Prints "1, 2, 3"
```

[i[`#define` directive]>]
[i[Preprocessor-->macros with variable arguments]>]
[i[Preprocessor-->macros with arguments]>]

### Stringification

[i[`#` stringification]<]

Như vừa nhắc ở trên, bạn có thể biến tham số thành chuỗi bằng cách
đặt `#` phía trước nó trong phần thay thế.

Ví dụ, ta có thể in bất cứ thứ gì dưới dạng chuỗi với macro này và
`printf()`:

``` {.c}
#define STR(x) #x

printf("%s\n", STR(3.14159));
```

Trong trường hợp đó, phép thay thế dẫn đến:

``` {.c}
printf("%s\n", "3.14159");
```

Xem ta có thể dùng cái này hiệu quả hơn không bằng cách truyền bất
kỳ tên biến `int` nào vào macro, rồi cho nó in ra tên và giá trị.

``` {.c .numberLines}
#include <stdio.h>

#define PRINT_INT_VAL(x) printf("%s = %d\n", #x, x)

int main(void)
{
    int a = 5;

    PRINT_INT_VAL(a);  // prints "a = 5"
}
```

Ở dòng 9, ta có phép thay thế macro sau:

``` {.c .numberLines startFrom="9"}
    printf("%s = %d\n", "a", a);
```

[i[`#` stringification]>]

### Nối chuỗi

[i[`##` concatenation]<]

Ta cũng có thể nối hai tham số với nhau bằng `##`. Vui quá đi!

``` {.c}
#define CAT(a, b) a ## b

printf("%f\n", CAT(3.14, 1592));   // 3.141592
```

[i[`##` concatenation]>]

## Macro nhiều dòng

[i[Preprocessor-->multiline macros]<]

Có thể kéo dài macro qua nhiều dòng nếu bạn escape ký tự xuống dòng
bằng gạch chéo ngược (`\`).

Hãy viết một macro nhiều dòng in các số từ `0` đến tích của hai tham
số truyền vào.

[i[`do`-`while` statement-->in multiline macros]<]

``` {.c .numberLines}
#include <stdio.h>

#define PRINT_NUMS_TO_PRODUCT(a, b) do { \
    int product = (a) * (b); \
    for (int i = 0; i < product; i++) { \
        printf("%d\n", i); \
    } \
} while(0)

int main(void)
{
    PRINT_NUMS_TO_PRODUCT(2, 4);  // Outputs numbers from 0 to 7
}
```

Vài điều chú ý:

* Escape ở cuối mỗi dòng trừ dòng cuối để báo macro còn tiếp.
* Cả mớ được bọc trong vòng `do`-`while(0)` với ngoặc xoắn xoắn
  xuýt.

Điểm thứ hai có thể hơi lạ, nhưng tất cả là để nuốt dấu `;` mà coder
quăng vào sau macro.

Lúc đầu tôi tưởng chỉ cần ngoặc xoắn là đủ, nhưng có một tình huống
nó hỏng nếu coder đặt dấu chấm phẩy sau macro. Đây là tình huống
đó:

``` {.c .numberLines}
#include <stdio.h>

#define FOO(x) { (x)++; }

int main(void)
{
    int i = 0;

    if (i == 0)
        FOO(i);
    else
        printf(":-(\n");

    printf("%d\n", i);
}
```

Trông có vẻ đơn giản, nhưng nó không build được vì lỗi cú pháp:

``` {.default}
foo.c:11:5: error: ‘else’ without a previous ‘if’  
```

Bạn thấy chưa?

Xem đoạn mở rộng:

``` {.c}

    if (i == 0) {
        (i)++;
    };             // <-- Trouble with a capital-T!

    else
        printf(":-(\n");
```

Dấu `;` kết thúc câu lệnh `if`, nên `else` cứ lơ lửng ngoài đó một
cách bất hợp pháp^[_Breakin' the law... breakin' the law..._].

Nên hãy bọc macro nhiều dòng đó trong `do`-`while(0)`.

[i[`do`-`while` statement-->in multiline macros]>]
[i[Preprocessor-->multiline macros]>]

## Ví dụ: Macro Assert {#my-assert}

Thêm assert vào code là cách hay để bắt các tình trạng mà bạn nghĩ
không nên xảy ra. C có sẵn chức năng `assert()`. Nó kiểm tra một
điều kiện, và nếu là false, chương trình nổ tung báo cho bạn biết
file và số dòng mà assertion thất bại.

Nhưng cái này còn thiếu.

1. Trước hết, bạn không thể đặc tả thêm thông điệp đi kèm assert.
2. Thứ hai, không có công tắc bật/tắt dễ dàng cho toàn bộ assert.

Ta có thể giải quyết cái đầu bằng macro.

Về cơ bản, khi tôi có code này:

``` {.c}
ASSERT(x < 20, "x must be under 20");
```

Tôi muốn cái gì đó như thế này xảy ra (giả sử `ASSERT()` ở dòng
220 của `foo.c`):

``` {.c}
if (!(x < 20)) {
    fprintf(stderr, "foo.c:220: assertion x < 20 failed: ");
    fprintf(stderr, "x must be under 20\n");
    exit(1);
}
```

Ta có thể lấy tên file từ macro `__FILE__`, và số dòng từ `__LINE__`.
Thông điệp đã là chuỗi, còn `x < 20` thì không, nên ta phải
stringify nó bằng `#`. Ta có thể làm macro nhiều dòng bằng cách
escape gạch chéo ngược ở cuối dòng.

``` {.c}
#define ASSERT(c, m) \
do { \
    if (!(c)) { \
        fprintf(stderr, __FILE__ ":%d: assertion %s failed: %s\n", \
                        __LINE__, #c, m); \
        exit(1); \
    } \
} while(0)
```

(Nó trông hơi lạ với `__FILE__` ở đầu như vậy, nhưng nhớ rằng nó là
string literal, và các string literal nằm cạnh nhau sẽ tự động nối
lại. `__LINE__` thì ngược lại, nó chỉ là `int`.)

Và nó chạy! Nếu tôi chạy cái này:

``` {.c}
int x = 30;

ASSERT(x < 20, "x must be under 20");
```

Tôi được output này:

```
foo.c:23: assertion x < 20 failed: x must be under 20
```

Ngon lành!

Cái còn lại là cách bật/tắt, và ta có thể làm với biên dịch có điều
kiện.

Đây là ví dụ hoàn chỉnh:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

#define ASSERT_ENABLED 1

#if ASSERT_ENABLED
#define ASSERT(c, m) \
do { \
    if (!(c)) { \
        fprintf(stderr, __FILE__ ":%d: assertion %s failed: %s\n", \
                        __LINE__, #c, m); \
        exit(1); \
    } \
} while(0)
#else
#define ASSERT(c, m)  // Empty macro if not enabled
#endif

int main(void)
{
    int x = 30;

    ASSERT(x < 20, "x must be under 20");
}
```

Cái này có output:

``` {.default}
foo.c:23: assertion x < 20 failed: x must be under 20
```

## Directive `#error`

[i[`#error` directive]<]

Directive này bắt compiler báo lỗi ngay khi nó thấy.

Thường thì dùng trong một điều kiện để ngăn biên dịch trừ khi vài
điều kiện tiên quyết được thỏa mãn:

``` {.c}
#ifndef __STDC_IEC_559__
    #error I really need IEEE-754 floating point to compile. Sorry!
#endif
```

[i[`#error` directive]>]
[i[`#warning` directive]<]

Vài compiler có directive `#warning` không chuẩn bổ sung sẽ xuất cảnh
báo nhưng không dừng biên dịch, nhưng cái này không có trong spec
C11.

[i[`#warning` directive]>]

## Directive `#embed`

[i[`#embed` directive]<]

<!-- Godbolt demo: https://godbolt.org/z/Kb3ejE7q5 -->

Mới ở C23!

Và hiện chưa chạy với bất kỳ compiler nào của tôi, nên hãy đọc phần
này với một hạt muối!

Ý là bạn có thể include các byte của một file dưới dạng hằng số
nguyên y như là bạn đã gõ chúng vào.

Ví dụ, nếu bạn có file nhị phân tên `foo.bin` chứa bốn byte với giá
trị thập phân 11, 22, 33, và 44, và bạn làm cái này:

``` {.c}
int a[] = {
#embed "foo.bin"
};
```

Thì y như bạn đã gõ:

``` {.c}
int a[] = {11,22,33,44};
```

Đây là cách rất mạnh để khởi tạo mảng với dữ liệu nhị phân mà không
cần chuyển tất cả sang code trước, preprocessor làm giùm bạn!

Trường hợp dùng điển hình hơn có thể là một file chứa hình ảnh nhỏ
để hiển thị mà bạn không muốn nạp lúc runtime.

Đây là ví dụ khác:

``` {.c}
int a[] = {
#embed <foo.bin>
};
```

Nếu bạn dùng ngoặc nhọn, preprocessor tìm trong một loạt vị trí do
implementation định nghĩa để tìm file, giống như `#include` làm. Nếu
bạn dùng nháy kép mà tài nguyên không tìm thấy, compiler sẽ thử như
là bạn đã dùng ngoặc nhọn, như một nỗ lực tuyệt vọng cuối cùng để
tìm file.

`#embed` hoạt động như `#include` ở chỗ nó dán giá trị vào trước khi
compiler thấy. Tức là bạn có thể dùng nó ở đủ loại chỗ:

```
return
#embed "somevalue.dat"
;
```

hoặc

```
int x =
#embed "xvalue.dat"
;
```

Giờ, đây có luôn là byte không? Nghĩa là giá trị sẽ từ 0 đến 255,
bao gồm cả? Câu trả lời mặc định chắc chắn là "có", trừ khi là
"không".

Về kỹ thuật, các phần tử sẽ rộng `CHAR_BIT` bit. Và rất có thể là 8
trên hệ của bạn, nên bạn sẽ có khoảng 0-255 trong các giá trị.
(Chúng sẽ luôn không âm.)

Cũng có thể một implementation cho phép ghi đè cái này theo cách nào
đó, ví dụ trên dòng lệnh hay với tham số.

Kích thước file tính bằng bit phải là bội của kích thước phần tử.
Tức là, nếu mỗi phần tử là 8 bit, kích thước file (tính bằng bit)
phải là bội của 8. Trong sử dụng hằng ngày, đây là cách nói vòng vo
rằng mỗi file phải là số nguyên lần byte... dĩ nhiên là vậy rồi.
Thành thật mà nói, tôi cũng chẳng biết sao mình lại bận tâm viết
đoạn này. Đọc spec nếu bạn thật sự tò mò.

### Tham số cho `#embed`

Có đủ loại tham số bạn có thể chỉ định cho directive `#embed`. Đây
là ví dụ với tham số `limit()` còn chưa giới thiệu:

``` {.c}
int a[] = {
#embed "/dev/random" limit(5)
};
```

Nhưng nếu bạn đã định nghĩa `limit` ở chỗ khác thì sao? May là bạn
có thể đặt `__` quanh từ khóa và nó sẽ hoạt động y vậy:

``` {.c}
int a[] = {
#embed "/dev/random" __limit__(5)
};
```

Giờ... cái `limit` này là gì?

### Tham số `limit()`

Bạn có thể đặt giới hạn số phần tử để embed bằng tham số này.

Đây là giá trị tối đa, không phải tuyệt đối. Nếu file được embed
ngắn hơn giới hạn chỉ định, chỉ bấy nhiêu byte sẽ được nhập vào.

Ví dụ `/dev/random` ở trên là ví dụ cho động lực của chuyện này,
trong Unix, đó là một _character device file_ sẽ trả về dòng số
ngẫu nhiên vô hạn.

Embed vô hạn byte thì ác với RAM, nên tham số `limit` cho bạn cách
dừng sau một số nhất định.

Cuối cùng, bạn được phép dùng macro `#define` trong `limit`, phòng
khi bạn tò mò.

### Tham số `if_empty`

[i[`if_empty()` embed parameter]<]

Tham số này định nghĩa kết quả embed phải là gì nếu file tồn tại
nhưng không chứa dữ liệu. Giả sử file `foo.dat` chứa một byte duy
nhất với giá trị 123. Nếu ta làm:

``` {.c}
int x = 
#embed "foo.dat" if_empty(999)
;
```

ta sẽ có:

``` {.c}
int x = 123;   // When foo.dat contains a 123 byte
```

Nhưng nếu `foo.dat` dài 0 byte (tức là không chứa dữ liệu và rỗng)?
Nếu vậy, nó sẽ mở rộng thành:

``` {.c}
int x = 999;   // When foo.dat is empty
```

Đáng chú ý là nếu `limit` đặt thành `0`, thì `if_empty` sẽ luôn được
thay vào. Tức là, giới hạn bằng không có nghĩa là file rỗng.

Cái này sẽ luôn phát ra `x = 999` bất kể trong `foo.dat` có gì:

``` {.c}
int x = 
#embed "foo.dat" limit(0) if_empty(999)
;
```

[i[`if_empty()` embed parameter]>]

### Tham số `prefix()` và `suffix()`

[i[`prefix()` embed parameter]<]
[i[`suffix()` embed parameter]<]

Đây là cách để thêm dữ liệu vào trước/sau embed.

Chú ý rằng các tham số này chỉ ảnh hưởng dữ liệu không rỗng! Nếu
file rỗng, cả `prefix` lẫn `suffix` đều không có tác dụng.

Ví dụ ta embed ba số ngẫu nhiên, nhưng thêm tiền tố `11,` và hậu tố
`,99`:

``` {.c}
int x[] = {
#embed "/dev/urandom" limit(3) prefix(11,) suffix(,99)
};
```

Ví dụ kết quả:

``` {.c}
int x[] = {11,135,116,220,99};
```

Không bắt buộc phải dùng cả `prefix` và `suffix`. Bạn có thể dùng cả
hai, một cái, cái kia, hay không cái nào.

Ta có thể tận dụng đặc tính chỉ áp dụng với file không rỗng này để
có hiệu quả hay ho, như trong ví dụ sau ăn cắp trắng trợn từ spec.

Giả sử có file `foo.dat` có dữ liệu trong đó. Và ta muốn dùng nó để
khởi tạo một mảng, rồi muốn hậu tố của mảng là một phần tử không.

Không vấn đề, đúng không?

``` {.c}
int x[] = {
#embed "foo.dat" suffix(,0)
};
```

Nếu `foo.dat` chứa 11, 22, và 33, ta sẽ có:

``` {.c}
int x[] = {11,22,33,0};
```

Nhưng khoan! Nếu `foo.dat` rỗng thì sao? Ta có:

``` {.c}
int x[] = {};
```

và cái đó không tốt.

Nhưng ta có thể sửa thế này:

``` {.c}
int x[] = {
#embed "foo.dat" suffix(,)
    0
};
```

Vì tham số `suffix` bị bỏ đi nếu file rỗng, cái này sẽ biến thành:

``` {.c}
int x[] = {0};
```

thế là ổn.

[i[`prefix()` embed parameter]>]
[i[`suffix()` embed parameter]>]

### Định danh `__has_embed()`

[i[`__has_embed()` identifier]<]

Đây là cách hay để kiểm tra xem một file cụ thể có sẵn sàng để embed
không, và cũng xem nó có rỗng hay không.

Bạn dùng nó với directive `#if`.

Đây là một đoạn code lấy 5 số ngẫu nhiên từ character device sinh số
ngẫu nhiên. Nếu cái đó không tồn tại, nó thử lấy từ file
`myrandoms.dat`. Nếu cái đó không tồn tại, nó dùng vài giá trị
cứng:

``` {.c}
    int random_nums[] = {
#if __has_embed("/dev/urandom")
    #embed "/dev/urandom" limit(5)
#elif __has_embed("myrandoms.dat")
    #embed "myrandoms.dat" limit(5)
#else
    140,178,92,167,120
#endif
    };
```

Về kỹ thuật, định danh `__has_embed()` phân giải ra một trong ba giá
trị:

|Kết quả `__has_embed()`|Mô tả|
|-|-|
|`__STDC_EMBED_NOT_FOUND__`|Nếu file không tìm thấy.|
|`__STDC_EMBED_FOUND__`|Nếu file tìm thấy và không rỗng.|
|`__STDC_EMBED_EMPTY__`|Nếu file tìm thấy và rỗng.|

Tôi có lý do chính đáng để tin rằng `__STDC_EMBED_NOT_FOUND__` là
`0` và mấy cái còn lại khác không (vì điều đó được ngầm chỉ trong
đề xuất và hợp logic), nhưng tôi đang vất vả tìm chỗ đó trong bản
dự thảo spec này.

[i[`__has_embed()` identifier]>]

TODO

### Tham số khác

Một implementation của compiler có thể định nghĩa các tham số embed
khác tùy ý, tìm các tham số không chuẩn này trong tài liệu của
compiler.

Ví dụ:

``` {.c}
#embed "foo.bin" limit(12) frotz(lamp)
```

Chúng thường có tiền tố để giúp namespace:

``` {.c}
#embed "foo.bin" limit(12) fmc::frotz(lamp)
```

Có lẽ hợp lý là thử phát hiện xem những cái này có sẵn không trước
khi dùng, và may là ta có thể dùng `__has_embed` để giúp.

Thường, `__has_embed()` sẽ chỉ báo file có ở đó hay không. Nhưng,
và đây là phần vui, nó cũng sẽ trả false nếu thêm bất kỳ tham số
nào không được hỗ trợ!

Vậy nếu ta đưa nó một file mà ta _biết_ tồn tại cùng với tham số mà
ta muốn test sự tồn tại, nó sẽ hiệu quả báo tham số đó có được hỗ
trợ không.

Nhưng file nào _luôn_ tồn tại? Hóa ra ta có thể dùng macro
`__FILE__`, mở rộng thành tên file nguồn tham chiếu đến nó! File đó
_phải_ tồn tại, không thì có chuyện cực kỳ nghiêm trọng trong mảng
con gà đẻ trứng.

Test tham số `frotz` xem có dùng được không:

``` {.c}
#if __has_embed(__FILE__ fmc::frotz(lamp))
    puts("fmc::frotz(lamp) is supported!");
#else
    puts("fmc::frotz(lamp) is NOT supported!");
#endif
```

### Embed giá trị nhiều byte

Thế còn việc nhét vài `int` vào thay vì byte đơn thì sao? Thế còn giá
trị nhiều byte trong file embed?

Đây không phải cái được chuẩn C23 hỗ trợ, nhưng có thể có các
extension implementation định nghĩa cho nó trong tương lai.

[i[`#embed` directive]>]

## Directive `#pragma` {#pragma}

[i[`#pragma` directive]<]

Đây là một directive kỳ cục, viết tắt của "pragmatic". Bạn có thể
dùng để làm... ờ, bất cứ gì mà compiler của bạn hỗ trợ bạn làm với
nó.

Về cơ bản, lần duy nhất bạn thêm cái này vào code là khi tài liệu
nào đó bảo bạn làm vậy.

### Pragma không chuẩn

[i[`#pragma` directive-->nonstandard pragmas]<]

Đây là một ví dụ không chuẩn dùng `#pragma` để bắt compiler chạy
vòng lặp `for` song song trên nhiều thread (nếu compiler hỗ trợ
extension [fl[OpenMP|https://www.openmp.org/]]):

``` {.c}
#pragma omp parallel for
for (int i = 0; i < 10; i++) { ... }
```

Có đủ loại directive `#pragma` được ghi chép ở khắp bốn góc của địa
cầu.

Mọi `#pragma` không nhận diện được đều bị compiler bỏ qua.

[i[`#pragma` directive-->nonstandard pragmas]>]

### Pragma chuẩn

Cũng có vài cái chuẩn, và chúng bắt đầu bằng `STDC`, theo cùng dạng:

``` {.c}
#pragma STDC pragma_name on-off
```

Phần `on-off` có thể là `ON`, `OFF`, hoặc `DEFAULT`.

Và `pragma_name` có thể là một trong các cái sau:

[i[`FP_CONTRACT` pragma]<]
[i[`CX_LIMITED_RANGE` pragma]<]

[i[`FENV_ACCESS` pragma]]

|Tên Pragma|Mô tả|
|-------------|--------------------------------------------------|
|`FP_CONTRACT`|Cho phép biểu thức floating point được rút gọn thành một phép toán duy nhất để tránh lỗi làm tròn có thể xảy ra từ nhiều phép toán.|
|`FENV_ACCESS`|Đặt `ON` nếu bạn định truy cập các cờ trạng thái floating point. Nếu `OFF`, compiler có thể thực hiện tối ưu gây ra giá trị trong cờ không nhất quán hoặc không hợp lệ.|
|`CX_LIMITED_RANGE`|Đặt `ON` để compiler bỏ qua kiểm tra tràn khi làm số học phức. Mặc định là `OFF`.|

Ví dụ:

``` {.c}
#pragma STDC FP_CONTRACT OFF
#pragma STDC CX_LIMITED_RANGE ON
```

[i[`FP_CONTRACT` pragma]>]

Về `CX_LIMITED_RANGE`, spec chỉ ra:

> Mục đích của pragma là cho implementation dùng các công thức:
>
> $(x+iy)\times(u+iv) = (xu-yv)+i(yu+xv)$
>
> $(x+iy)/(u+iv) = [(xu+yv)+i(yu-xv)]/(u^2+v^2)$
>
> $|x+iy|=\sqrt{x^2+y^2}$
>
> khi lập trình viên có thể xác định rằng chúng an toàn.

[i[`CX_LIMITED_RANGE` pragma]>]

### Toán tử `_Pragma`

[i[`_Pragma` operator]<]

Đây là cách khác để khai báo pragma mà bạn có thể dùng trong một
macro.

Hai thứ sau là tương đương:

``` {.c}
#pragma "Unnecessary" quotes
_Pragma("\"Unnecessary\" quotes")
```

[i[`_Pragma` operator-->in a macro]<]

Cái này có thể dùng trong macro, khi cần:

``` {.c}
#define PRAGMA(x) _Pragma(#x)
```

[i[`_Pragma` operator-->in a macro]>]
[i[`_Pragma` operator]>]
[i[`#pragma` directive]>]

## Directive `#line`

[i[`#line` directive]<]
[i[`__LINE__` macro]<]

Cái này cho phép bạn ghi đè giá trị cho `__LINE__` và `__FILE__`.
Nếu bạn muốn.

Tôi chưa bao giờ muốn làm chuyện này, nhưng trong K&R2, họ viết:

> Để hữu ích cho các preprocessor khác sinh ra chương trình C [...]

Vậy có khi có chỗ cho nó.

Để ghi đè số dòng thành, chẳng hạn, 300:

``` {.c}
#line 300
```

và `__LINE__` sẽ tiếp tục đếm lên từ đó.

[i[`__LINE__` macro]>]

Để ghi đè số dòng và tên file:

``` {.c}
#line 300 "newfilename"
```

[i[`#line` directive]>]

## Directive Null

[i[`#` null directive]<]

Một ký tự `#` trên một dòng đứng một mình sẽ bị preprocessor bỏ qua.
Thành thật mà nói, tôi không biết ca dùng cho chuyện này là gì.

Tôi đã thấy ví dụ kiểu này:

``` {.c}
#ifdef FOO
    #
#else
    printf("Something");
#endif
```

đó chỉ là về mặt thẩm mỹ; dòng với `#` đơn độc có thể bị xóa đi mà
không có tác động gì xấu.

Hay có lẽ vì tính nhất quán thẩm mỹ, thế này:

``` {.c}
#
#ifdef FOO
    x = 2;
#endif
#
#if BAR == 17
    x = 12;
#endif
#
```

Nhưng, về mặt thẩm mỹ, cái đó chỉ là xấu.

Một bài đăng khác nhắc đến chuyện xóa comment, rằng trong GCC, một
comment sau `#` sẽ không được compiler nhìn thấy. Cái đó tôi không
nghi ngờ, nhưng spec dường như không nói đây là hành vi chuẩn.

Mấy tìm kiếm của tôi cho nguyên do không mang về nhiều quả. Nên tôi
đành nói đây là một thứ C esoterica kiểu cổ điển.

[i[`#` null directive]>]
[i[Preprocessor]>]

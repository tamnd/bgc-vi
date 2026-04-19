<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Types III: Chuyển đổi

[i[Type conversions]<]

Ở chương này, ta muốn nói hết về chuyện chuyển đổi từ kiểu này sang
kiểu khác. C có nhiều cách để làm điều này, và một số có thể hơi khác
bạn quen ở ngôn ngữ khác.

Trước khi nói cách ép chuyển đổi xảy ra, hãy bàn về cách chúng hoạt
động khi chúng _đã_ xảy ra.

## Chuyển đổi chuỗi

[i[Type conversions-->strings]<]

Khác nhiều ngôn ngữ, C không làm chuyển đổi chuỗi-sang-số (và ngược
lại) theo kiểu gọn gàng như với chuyển đổi số.

Với mấy thứ này, ta phải gọi hàm để làm việc bẩn.

### Giá trị số sang chuỗi

Khi muốn chuyển số sang chuỗi, ta có thể dùng `sprintf()` (phát âm là
_SPRINT-f_) hoặc `snprintf()` (_s-n-print-f_)^[Chúng giống nhau, chỉ
khác `snprintf()` cho phép bạn chỉ định số byte xuất tối đa, tránh
tràn cuối chuỗi. Nên an toàn hơn.]

Mấy cái này về cơ bản hoạt động như `printf()`, chỉ khác chúng xuất
ra chuỗi, và bạn có thể in chuỗi đó sau, hay gì tuỳ ý.

Ví dụ, biến một phần giá trị π thành chuỗi:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    char s[10];
    float f = 3.14159;

    // Convert "f" to string, storing in "s", writing at most 10 characters
    // including the NUL terminator

    snprintf(s, 10, "%f", f);

    printf("String value: %s\n", s);  // String value: 3.141590
}
```

Bạn có thể dùng `%d` hay `%u` như bạn quen cho số nguyên.

### Chuỗi sang giá trị số

Có hai họ hàm làm việc này trong C. Ta sẽ gọi chúng là họ `atoi` (phát
âm _a-to-i_) và họ `strtol` (_stir-to-long_).

Để chuyển đổi cơ bản từ chuỗi sang số, thử các hàm `atoi` từ
`<stdlib.h>`. Chúng có đặc tính xử lý lỗi tệ (kể cả undefined behavior
nếu bạn truyền chuỗi xấu), nên dùng cẩn thận.

|Hàm|Mô tả|
|:-|:-|
|`atoi`|Chuỗi sang `int`|
|`atof`|Chuỗi sang `float`|
|`atol`|Chuỗi sang `long int`|
|`atoll`|Chuỗi sang `long long int`|

Dù spec không thừa nhận, chữ `a` đầu tên hàm là viết tắt của
[flw[ASCII|ASCII]], nên thực ra `atoi()` là "ASCII-to-integer", nhưng
nói thế giờ hơi quy ASCII về làm trung tâm.

Ví dụ chuyển chuỗi sang `float`:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *pi = "3.14159";
    float f;

    f = atof(pi);

    printf("%f\n", f);
}
```

Nhưng, như đã nói, ta có undefined behavior từ những chuyện lạ lùng
như:

``` {.c}
int x = atoi("what");  // "What" ain't no number I ever heard of
```

(Khi chạy cái đó, tôi nhận về `0`, nhưng bạn thật sự không nên trông
cậy vào đó bằng bất cứ cách nào. Có thể bạn nhận về thứ hoàn toàn
khác.)

Để có đặc tính xử lý lỗi tốt hơn, xem đống hàm `strtol`, cũng trong
`<stdlib.h>`. Không chỉ thế, chúng còn chuyển sang nhiều kiểu và
nhiều cơ số hơn!

|Hàm|Mô tả|
|:-|:-|
|`strtol`|Chuỗi sang `long int`|
|`strtoll`|Chuỗi sang `long long int`|
|`strtoul`|Chuỗi sang `unsigned long int`|
|`strtoull`|Chuỗi sang `unsigned long long int`|
|`strtof`|Chuỗi sang `float`|
|`strtod`|Chuỗi sang `double`|
|`strtold`|Chuỗi sang `long double`|

Các hàm này đều đi theo mẫu dùng tương tự, và là trải nghiệm đầu tiên
của nhiều người với con-trỏ-tới-con-trỏ! Nhưng đừng lo, dễ hơn trông
thấy nhiều.

Ví dụ chuyển chuỗi sang `unsigned long`, bỏ qua thông tin lỗi (tức
thông tin về ký tự sai trong chuỗi đầu vào):

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = "3490";

    // Convert string s, a number in base 10, to an unsigned long int.
    // NULL means we don't care to learn about any error information.

    unsigned long int x = strtoul(s, NULL, 10);

    printf("%lu\n", x);  // 3490
}
```

Chú ý vài thứ. Dù ta không hạ cố lấy thông tin gì về ký tự lỗi trong
chuỗi, `strtoul()` không cho ta undefined behavior; nó chỉ trả về
`0`.

Ta cũng chỉ định đây là số thập phân (base 10).

Thế nghĩa là ta có thể chuyển số ở cơ số khác? Chắc rồi! Làm nhị phân!

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = "101010";  // What's the meaning of this number?

    // Convert string s, a number in base 2, to an unsigned long int.

    unsigned long int x = strtoul(s, NULL, 2);

    printf("%lu\n", x);  // 42
}
```

Được rồi, vui thú đó, nhưng cái `NULL` trong đó là gì? Để làm gì?

Nó giúp ta biết có lỗi xảy ra khi xử lý chuỗi hay không. Là một con
trỏ tới con trỏ tới `char`, nghe đáng sợ, nhưng không còn đáng sợ khi
bạn ghép được trong đầu.

Làm ví dụ với số xấu cố tình, xem `strtol()` báo cho ta vị trí của chữ
số hợp lệ đầu tiên thế nào.

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = "34x90";  // "x" is not a valid digit in base 10!
    char *badchar;

    // Convert string s, a number in base 10, to an unsigned long int.

    unsigned long int x = strtoul(s, &badchar, 10);

    // It tries to convert as much as possible, so gets this far:

    printf("%lu\n", x);  // 34

    // But we can see the offending bad character because badchar
    // points to it!

    printf("Invalid character: %c\n", *badchar);  // "x"
}
```

Thế là `strtoul()` chỉnh cái `badchar` trỏ tới để báo cho ta chỗ có
chuyện không hay^[Ta phải truyền con trỏ tới `badchar` cho `strtoul()`
chứ không nó không chỉnh được theo cách ta thấy, tương tự lý do bạn
phải truyền con trỏ tới `int` cho hàm nếu muốn hàm đó có thể đổi giá
trị của `int` đó.].

Nhưng nếu không có gì trục trặc thì sao? Trường hợp đó, `badchar` sẽ
trỏ tới ký tự kết chuỗi `NUL` ở cuối chuỗi. Nên ta có thể kiểm tra:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = "3490";  // "x" is not a valid digit in base 10!
    char *badchar;

    // Convert string s, a number in base 10, to an unsigned long int.

    unsigned long int x = strtoul(s, &badchar, 10);

    // Check if things went well

    if (*badchar == '\0') {
        printf("Success! %lu\n", x);
    } else  {
        printf("Partial conversion: %lu\n", x);
        printf("Invalid character: %c\n", *badchar);
    }
}
```

Vậy là xong. Hàm kiểu `atoi()` tốt khi tình huống gấp có kiểm soát,
nhưng hàm kiểu `strtol()` cho bạn kiểm soát tốt hơn hẳn về xử lý lỗi
và cơ số đầu vào.

[i[Type conversions-->strings]>]

## Chuyển đổi `char`

[i[Type conversions-->`char`]<]

Nếu bạn có một ký tự chứa chữ số, như `'5'`... có giống giá trị `5`
không?

Thử xem.

``` {.c}
printf("%d %d\n", 5, '5');
```

Trên hệ thống UTF-8 của tôi, cái này in:

``` {.default}
5 53
```

Vậy... không. Còn 53? Là gì? Đó là code point trong UTF-8 (và ASCII)
cho ký tự `'5'`^[Mỗi ký tự có một giá trị gắn với nó ứng với sơ đồ
mã hoá ký tự cụ thể nào đó.]

Vậy làm sao chuyển ký tự `'5'` (có giá trị 53 rõ ràng) thành giá trị
`5`?

Bằng một trick lanh lợi, đây này!

Chuẩn C đảm bảo các ký tự này có code point liên tiếp và theo thứ tự
này:

``` {.default}
0  1  2  3  4  5  6  7  8  9
```

Nghĩ một giây, ta có thể tận dụng thế nào? Spoiler ở dưới...

Xem các ký tự và code point của chúng trong UTF-8:

``` {.default}
0  1  2  3  4  5  6  7  8  9
48 49 50 51 52 53 54 55 56 57
```

Bạn thấy `'5'` là `53`, y như ta đã có. Và `'0'` là `48`.

Nên ta có thể trừ `'0'` khỏi bất kỳ ký tự chữ số nào để lấy giá trị
số của nó:

``` {.c}
char c = '6';

int x = c;  // x has value 54, the code point for '6'

int y = c - '0'; // y has value 6, just like we want
```

Và ta cũng có thể chuyển chiều kia, chỉ việc cộng giá trị vào.

``` {.c}
int x = 6;

char c = x + '0';  // c has value 54

printf("%d\n", c);  // prints 54
printf("%c\n", c);  // prints 6 with %c
```

Bạn có thể nghĩ đây là cách lạ lùng để chuyển đổi, và theo chuẩn ngày
nay, đúng là thế. Nhưng hồi xưa khi máy tính được làm bằng gỗ theo
đúng nghĩa đen, đây là cách chuyển đổi. Và không hư, nên C chưa sửa.

[i[Type conversions-->`char`]>]

## Chuyển đổi số

[i[Type conversions-->numeric]<]

### Boolean

[i[Type conversions-->Boolean]]

Nếu bạn chuyển một số 0 sang `bool`, kết quả là `0`. Ngược lại là `1`.

### Chuyển giữa số nguyên

[i[Type conversions-->integer]<]

Nếu kiểu số nguyên được chuyển sang unsigned mà không vừa, kết quả
unsigned sẽ wrap-around kiểu công-tơ-mét cho tới khi vừa kiểu
unsigned^[Trong thực tế, cái nhiều khả năng đang xảy ra trên cài đặt
của bạn là các bit bậc cao bị bỏ khỏi kết quả, nên số 16-bit `0x1234`
khi chuyển sang số 8-bit sẽ thành `0x0034`, hay chỉ `0x34`.].

Nếu kiểu số nguyên được chuyển sang số signed mà không vừa, kết quả
là implementation-defined! Chuyện gì đó có ghi sẽ xảy ra, nhưng bạn
phải tra tài liệu^[Lần nữa, trong thực tế, cái có khả năng xảy ra
trên hệ thống của bạn là mẫu bit của số gốc sẽ bị cắt rồi dùng luôn
để biểu diễn số signed, two's complement. Ví dụ, hệ của tôi lấy một
`unsigned char` `192` và chuyển thành `signed char` `-64`. Trong
two's complement, mẫu bit cho cả hai số này là nhị phân `11000000`.]

### Chuyển số nguyên và số dấu phẩy động

[i[Type conversions-->floating point]<]

Nếu kiểu dấu phẩy động được chuyển sang kiểu nguyên, phần lẻ bị vứt
đi không thương tiếc^[Thật ra không, cứ vứt như thường.].

Nhưng, và đây là bẫy, nếu số quá lớn không vừa kiểu nguyên, bạn có
undefined behavior. Nên đừng làm thế.

Đi từ số nguyên hay dấu phẩy động sang dấu phẩy động, C cố gắng hết
sức để tìm số dấu phẩy động gần nhất với số nguyên.

Lần nữa, nếu giá trị gốc không biểu diễn được, đó là undefined
behavior.

[i[Type conversions-->floating point]>]
[i[Type conversions-->integer]>]

## Chuyển đổi ngầm

[i[Type conversions-->implicit]<]

Đây là các chuyển đổi compiler tự làm cho bạn khi bạn trộn các kiểu.

### Integer Promotions {#integer-promotions}

Ở nhiều chỗ, nếu một `int` có thể dùng để biểu diễn giá trị từ `char`
hay `short` (signed hay unsigned), giá trị đó được _promote_ (nâng)
lên `int`. Nếu không vừa `int`, nó được nâng lên `unsigned int`.

Đó là cách ta làm được chuyện như:

``` {.c}
char x = 10, y = 20;
int i = x + y;
```

Trường hợp đó, `x` và `y` được C nâng lên `int` trước khi phép toán
diễn ra.

Integer promotion xảy ra trong The Usual Arithmetic Conversions, với
hàm variadic^[Hàm có số đối số thay đổi.], toán tử `+` và `-` đơn,
hoặc khi truyền giá trị cho hàm không có prototype^[Hiếm làm vì
compiler sẽ than phiền và có prototype là _Cách làm đúng_. Tôi nghĩ
cái này vẫn chạy vì lý do lịch sử, trước khi prototype ra đời.].

### The Usual Arithmetic Conversions {#usual-arithmetic-conversions}

Đây là các chuyển đổi tự động C làm quanh các phép toán số bạn yêu
cầu. (Tên gọi thật sự là thế, nhân tiện, theo C11 §6.3.1.8.) Chú ý ở
mục này, ta chỉ nói kiểu số, chuỗi sẽ bàn sau.

Các chuyển đổi này trả lời câu hỏi chuyện gì xảy ra khi bạn trộn
kiểu, như:

``` {.c}
int x = 3 + 1.2;   // Mixing int and double
                   // 4.2 is converted to int
                   // 4 is stored in x

float y = 12 * 2;  // Mixing float and int
                   // 24 is converted to float
                   // 24.0 is stored in y
```

Chúng thành `int`? Chúng thành `float`? Hoạt động thế nào?

Đây là các bước, diễn giải lại cho dễ nuốt.

1. Nếu có một thứ trong biểu thức là kiểu dấu phẩy động, chuyển các
   thứ khác sang kiểu dấu phẩy động đó.

2. Ngược lại, nếu cả hai đều là kiểu nguyên, thực hiện integer
   promotion trên mỗi cái, rồi làm kiểu của các toán hạng đủ lớn để
   chứa giá trị lớn chung. Đôi khi việc này liên quan đến chuyện đổi
   signed sang unsigned.

Nếu muốn biết chi tiết vụn, xem C11 §6.3.1.8. Nhưng chắc bạn không
muốn đâu.

Nhớ đại khái là kiểu int thành kiểu float nếu có kiểu dấu phẩy động
ở đâu trong đó, và compiler cố gắng đảm bảo các kiểu int trộn không
bị tràn.

Cuối cùng, nếu bạn chuyển từ kiểu dấu phẩy động này sang kiểu dấu
phẩy động khác, compiler sẽ cố chuyển đổi chính xác. Nếu không được,
nó sẽ làm xấp xỉ tốt nhất có thể. Nếu số quá lớn không vừa kiểu bạn
đang chuyển qua, _bùm_: undefined behavior!

### `void*`

Kiểu `void*` thú vị vì nó có thể chuyển từ hay sang bất kỳ kiểu con
trỏ nào.

``` {.c}
int x = 10;

void *p = &x;  // &x is type int*, but we store it in a void*

int *q = p;    // p is void*, but we store it in an int*
```

[i[Type conversions-->implicit]>]

## Chuyển đổi tường minh

[i[Type conversions-->explicit]<]

Đây là các chuyển đổi từ kiểu sang kiểu mà bạn phải yêu cầu, compiler
sẽ không tự làm.

Bạn có thể chuyển từ kiểu này sang kiểu khác bằng cách gán với `=`.

Bạn cũng có thể chuyển tường minh bằng _cast_.

[i[Type conversions-->numeric]>]

### Casting

[i[Type conversions-->casting]<]

Bạn có thể đổi tường minh kiểu của biểu thức bằng cách đặt một kiểu
mới trong ngoặc trước nó. Vài dev C cau mày với cách này trừ khi thật
sự cần, nhưng bạn có khả năng gặp một ít code C có cast bên trong.

Làm ví dụ muốn chuyển `int` sang `long` để lưu trong `long`.

Chú ý: ví dụ này bịa đặt và cast ở đây hoàn toàn không cần vì biểu
thức `x + 12` sẽ tự chuyển sang `long int` để hợp với kiểu rộng hơn
của `y`.

``` {.c}
int x = 10;
long int y = (long int)x + 12;
```

Trong ví dụ đó, mặc dù `x` là kiểu `int` trước đó, biểu thức
`(long int)x` có kiểu `long int`. Ta nói, "Ta cast `x` sang `long
int`."

Thường gặp hơn, bạn có thể thấy cast được dùng để chuyển `void*`
thành kiểu con trỏ cụ thể để có thể dereference.

Callback từ hàm có sẵn `qsort()` có thể thể hiện hành vi này vì nó
có `void*` được truyền vào:

``` {.c}
int compar(const void *elem1, const void *elem2)
{
    if (*((const int*)elem2) > *((const int*)elem1)) return 1;
    if (*((const int*)elem2) < *((const int*)elem1)) return -1;
    return 0;
}
```

Nhưng bạn cũng có thể viết rõ ràng bằng phép gán:

``` {.c}
int compar(const void *elem1, const void *elem2)
{
    const int *e1 = elem1;
    const int *e2 = elem2;

    return *e2 - *e1;
}
```

Một chỗ bạn thấy cast phổ biến hơn là để tránh warning khi in giá trị
con trỏ với `%p` hiếm dùng, cái này khó tính với bất cứ thứ gì không
phải `void*`:

``` {.c}
int x = 3490;
int *p = &x;

printf("%p\n", p);
```

sinh ra warning này:

``` {.default}
warning: format ‘%p’ expects argument of type ‘void *’, but argument
         2 has type ‘int *’
```

Bạn có thể fix bằng cast:

``` {.c}
printf("%p\n", (void *)p);
```

Chỗ khác là với đổi con trỏ tường minh, nếu không muốn dùng `void*`
ở giữa, nhưng cái này cũng khá hiếm:

``` {.c}
long x = 3490;
long *p = &x;
unsigned char *c = (unsigned char *)p;
```

Chỗ thứ ba thường yêu cầu là với các hàm chuyển đổi ký tự trong
[fl[`<ctype.h>`|https://beej.us/guide/bgclr/html/split/ctype.html]]
ở đó bạn nên cast các giá trị signedness đáng ngờ sang `unsigned
char` để tránh undefined behavior.

Một lần nữa, cast hiếm khi _cần_ trong thực tế. Nếu bạn thấy mình
đang cast, có khả năng có cách khác làm cùng chuyện đó, hoặc có thể
bạn đang cast không cần thiết.

Hoặc có thể là cần. Cá nhân tôi, tôi cố tránh, nhưng không ngại dùng
nếu phải.

[i[Type conversions-->explicit]>]
[i[Type conversions-->casting]>]
[i[Type conversions]>]

<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Types II: Còn nhiều kiểu nữa!

Ta đã quen với các kiểu `char`, `int`, và `float`, nhưng giờ là lúc
nâng những thứ đó lên tầm cao mới và xem còn gì nữa ở khoản kiểu!

## Số nguyên có dấu và không dấu

[i[Types-->signed and unsigned]<]
Tới giờ ta dùng `int` như kiểu _có dấu_ (signed), tức giá trị có thể
âm hoặc dương. Nhưng C còn có các kiểu nguyên _không dấu_ (unsigned)
cụ thể, chỉ chứa được số dương.

Các kiểu này dùng từ khoá [i[`unsigned` type]<]`unsigned` đi trước.

``` {.c}
int a;           // signed
signed int a;    // signed
signed a;        // signed, "shorthand" for "int" or "signed int", rare
unsigned int b;  // unsigned
unsigned c;      // unsigned, shorthand for "unsigned int"
```

Vì sao? Vì sao bạn quyết định chỉ muốn chứa số dương?

Đáp: với biến unsigned, bạn có thể chứa số lớn hơn so với biến signed.

Nhưng vì sao thế?

Bạn có thể nghĩ về số nguyên được biểu diễn bởi một số _bit_^["Bit"
là viết tắt của _binary digit_ (chữ số nhị phân). Nhị phân chỉ là một
cách biểu diễn số khác. Thay vì các chữ số 0-9 như ta quen, là các
chữ số 0-1.]. Trên máy tôi, một `int` được biểu diễn bởi 64 bit.

Và mỗi hoán vị các bit là `1` hay `0` biểu diễn một số. Ta có thể
quyết định chia các số này thế nào.

Với số signed, ta dùng (xấp xỉ) một nửa số hoán vị để biểu diễn số âm,
nửa kia biểu diễn số dương.

Với unsigned, ta dùng _tất cả_ các hoán vị để biểu diễn số dương.

Trên máy tôi với `int` 64-bit dùng [flw[two's
complement|Two%27s_complement]] để biểu diễn số, tôi có các giới hạn
sau về miền số nguyên:

|Kiểu|Min|Max|
|:-|-:|-:|
|`int`|`-9,223,372,036,854,775,808`|`9,223,372,036,854,775,807`|
|`unsigned int`|`0`|`18,446,744,073,709,551,615`|

Chú ý số `unsigned int` dương lớn nhất xấp xỉ gấp đôi số `int` dương
lớn nhất. Nên bạn có một chút linh hoạt.
[i[`unsigned` type]>]
[i[Types-->signed and unsigned]>]

## Kiểu ký tự

[i[Types-->character]<]
[i[`char` type]<]
Nhớ `char` chứ? Kiểu dùng để chứa một ký tự?

``` {.c}
char c = 'B';

printf("%c\n", c);  // "B"
```

Tôi có tin sốc cho bạn: nó thật ra là một số nguyên.

``` {.c}
char c = 'B';

// Change this from %c to %d:
printf("%d\n", c);  // 66 (!!)
```

Ở tầng sâu, `char` chỉ là một `int` nhỏ, cụ thể là một số nguyên dùng
đúng một byte chỗ, hạn chế miền giá trị xuống còn...

Ở đây spec C hơi khó chịu. Nó đảm bảo rằng `char` là một byte, tức
`sizeof(char) == 1`. Nhưng rồi ở C11 §3.6¶3 nó đi thẳng ra nói:

> A byte is composed of a contiguous sequence of bits, _the number of
> which is implementation-defined._

Khoan, cái gì? Có lẽ một số bạn quen với ý niệm một byte là 8 bit
chứ? Ý tôi là đúng vậy mà, đúng không? Và câu trả lời là, "Gần như
chắc chắn."^[Thuật ngữ ngành cho một dãy chính xác, không tranh cãi,
8 bit là _octet_.] Nhưng C là ngôn ngữ đời cũ, và máy móc thời đó có,
hãy nói là, ý kiến _thoáng_ hơn về chuyện một byte có bao nhiêu bit.
Và qua năm tháng, C vẫn giữ sự linh hoạt đó.

Nhưng giả định byte trong C là 8 bit, như thực tế với gần như mọi máy
trên thế giới bạn từng thấy, miền của một `char` là...

[i[`unsigned char` type]<]
[i[`signed char` type]<]
Khoan, trước khi tôi nói được, hoá ra `char` có thể signed hoặc
unsigned tuỳ compiler. Trừ khi bạn chỉ định rõ.

Nhiều trường hợp, có `char` là ổn vì bạn không quan tâm dấu của dữ
liệu. Nhưng nếu cần signed hoặc unsigned `char`, bạn _phải_ chỉ định
rõ:

``` {.c}
char a;           // Could be signed or unsigned
signed char b;    // Definitely signed
unsigned char c;  // Definitely unsigned
```

Được rồi, giờ cuối cùng, ta có thể tính miền các số nếu giả định
`char` là 8 bit và hệ thống bạn dùng biểu diễn two's complement gần
như phổ biến cho signed và unsigned^[Nói chung, nếu bạn có số two's
complement $n$ bit, miền signed là $-2^{n-1}$ tới $2^{n-1}-1$. Còn
miền unsigned là $0$ tới $2^n-1$.].

Với các ràng buộc đó, cuối cùng ta có miền:

|Kiểu `char`|Min|Max|
|:-|-:|-:|
|`signed char`|`-128`|`127`|
|`unsigned char`|`0`|`255`|

Và miền cho `char` là implementation-defined.
[i[`unsigned char` type]>]
[i[`signed char` type]>]

Cho tôi xác nhận lại. `char` thực ra là một số, vậy ta làm toán với
nó được không?

Được! Chỉ cần nhớ giữ mọi thứ trong miền của `char`!

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    char a = 10, b = 20;

    printf("%d\n", a + b);  // 30!
}
```

[i[`'` single quote]<]
Thế còn các hằng ký tự trong dấu nháy đơn như `'B'`? Làm sao nó có
giá trị số?

Spec ở đây cũng mơ hồ, vì C không được thiết kế để chạy trên một kiểu
hệ thống nền duy nhất.

Nhưng cứ giả định tạm rằng bộ ký tự của bạn dựa trên [flw[ASCII|ASCII]]
ít nhất cho 128 ký tự đầu. Trường hợp đó, hằng ký tự sẽ được chuyển
thành một `char` có giá trị bằng giá trị ASCII của ký tự đó.

Nghe nhiều nhỉ. Xem ví dụ:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    char a = 10;
    char b = 'B';  // ASCII value 66

    printf("%d\n", a + b);  // 76!
}
```

Chuyện này tuỳ vào môi trường thực thi và [flw[bộ ký tự được
dùng|List_of_information_system_character_sets]]. Một trong những bộ
ký tự phổ biến nhất hiện nay là [flw[Unicode|Unicode]] (vốn là tập
cha của ASCII), nên với các 0-9, A-Z, a-z và dấu câu cơ bản, bạn gần
như chắc chắn lấy ra được giá trị ASCII.
[i[`'` single quote]>]
[i[`char` type]>]
[i[Types-->character]>]

## Thêm kiểu nguyên: `short`, `long`, `long long`

Tới giờ nói chung ta mới dùng hai kiểu số nguyên:

* `char`
* `int`

và gần đây học thêm các biến thể unsigned của các kiểu nguyên. Và ta
đã biết `char` bí mật là một `int` nhỏ trá hình. Nên ta biết `int`
có thể tới với nhiều kích cỡ bit.

Nhưng còn vài kiểu nguyên nữa ta nên xem, và miền tối thiểu chúng có
thể chứa. (Cài đặt của bạn có thể có miền rộng hơn so với spec yêu
cầu, nhưng các miền ở đây là cái bạn có thể **chắc chắn** có portable
khắp nơi.)

File header `<limits.h>` định nghĩa các macro chứa miền cho các kiểu
khác nhau; dựa vào đó cho chắc, và _đừng bao giờ hard-code hay giả
định các giá trị này_.

[i[`short` type]<]
[i[`long` type]<]
[i[`long long` type]<]
Các kiểu thêm này là `short int`, `long int`, và `long long int`. Khi
dùng các kiểu này, dev C thường bỏ phần `int` đi (ví dụ `long long`),
và compiler vẫn vui vẻ.

``` {.c}
// These two lines are equivalent:
long long int x;
long long x;

// And so are these:
short int x;
short x;
```

Xem các kiểu dữ liệu nguyên và kích cỡ theo thứ tự tăng, nhóm theo
tính signed. Lần nữa, các giới hạn min/max này là cái spec đảm bảo
portable; hệ thống của bạn có thể có miền rộng hơn.

|Kiểu|Min Bytes|Min|Max|
|:-|-:|-:|-:|
|`char`|1|-128 or 0|127 or 255^[Tuỳ vào `char` mặc định là `signed char` hay `unsigned char`]|
|`signed char`|1|-128|127|
|`short`|2|-32768|32767|
|`int`|2|-32768|32767|
|`long`|4|-2147483648|2147483647|
|`long long`|8|-9223372036854775808|9223372036854775807|
|`unsigned char`|1|0|255|
|`unsigned short`|2|0|65535|
|`unsigned int`|2|0|65535|
|`unsigned long`|4|0|4294967295|
|`unsigned long long`|8|0|18446744073709551615|

Không có kiểu `long long long`. Bạn không thể cứ thêm `long` vào thế.
Đừng ngốc.

> Fan two's complement có thể đã nhận ra gì đó hài về các số đó. Ví
> dụ, vì sao `signed char` dừng ở -127 chứ không phải -128? Nhớ: đây
> chỉ là tối thiểu spec yêu cầu. Vài cách biểu diễn số (như [flw[sign
> and
> magnitude|Signed_number_representations#Signed_magnitude_representation]])
> chặn ở ±127.

Chạy lại bảng đó trên hệ thống 64-bit two's complement của tôi xem
ra gì:

|Kiểu|My Bytes|Min|Max|
|:-|-:|-:|-:|
|`char`|1|-128|127^[`char` của tôi là signed.]|
|`signed char`|1|-128|127|
|`short`|2|-32768|32767|
|`int`|4|-2147483648|2147483647|
|`long`|8|-9223372036854775808|9223372036854775807|
|`long long`|8|-9223372036854775808|9223372036854775807|
|`unsigned char`|1|0|255|
|`unsigned short`|2|0|65535|
|`unsigned int`|4|0|4294967295|
|`unsigned long`|8|0|18446744073709551615|
|`unsigned long long`|8|0|18446744073709551615|

Hợp lý hơn chút, nhưng ta có thể thấy hệ thống của tôi có giới hạn
lớn hơn so với tối thiểu trong spec.

[Vậy các macro trong `<limits.h>` là gì?]{#limits-macros}

|Kiểu|Min Macro|Max Macro|
|:-|:-|:-|
|`char`|`CHAR_MIN`|`CHAR_MAX`|
|`signed char`|`SCHAR_MIN`|`SCHAR_MAX`|
|`short`|`SHRT_MIN`|`SHRT_MAX`|
|`int`|`INT_MIN`|`INT_MAX`|
|`long`|`LONG_MIN`|`LONG_MAX`|
|`long long`|`LLONG_MIN`|`LLONG_MAX`|
|`unsigned char`|`0`|`UCHAR_MAX`|
|`unsigned short`|`0`|`USHRT_MAX`|
|`unsigned int`|`0`|`UINT_MAX`|
|`unsigned long`|`0`|`ULONG_MAX`|
|`unsigned long long`|`0`|`ULLONG_MAX`|

Chú ý có cách kín đáo ở đó để xác định hệ thống dùng `char` signed
hay unsigned. Nếu `CHAR_MAX == UCHAR_MAX`, nó phải là unsigned.

Cũng chú ý không có macro min cho các biến thể `unsigned`, chúng chỉ
là `0`.
[i[`short` type]>]
[i[`long` type]>]
[i[`long long` type]>]

## Thêm float: `double` và `long double`

Xem spec nói gì về số dấu phẩy động ở §5.2.4.2.2¶1-2:

>The following parameters are used to define the model for each
>floating-point type:
>
> |Parameter|Definition|
> |:-|:-|
> |$s$|sign ($\pm1$)|
> |$b$|base or radix of exponent representation (an integer $> 1$)|
> |$e$|exponent (an integer between a minimum $e_{min}$ and a maximum $e_{max}$)|
> |$p$|precision (the number of base-$b$ digits in the significand)|
> |$f_k$|nonnegative integers less than $b$ (the significand digits)|
>
> A _floating-point number_ ($x$) is defined by the following model:
>
>> $x=sb^e\sum\limits_{k=1}^p f_kb^{-k},$\ \ \ \ $e_{min}\le e\le e_{max}$

Hy vọng mọi chuyện giờ đã sáng tỏ với bạn.

Thôi được. Lùi lại một bước xem cái gì thực tế.

Chú ý: ta nhắc đến một đống macro trong mục này. Chúng nằm ở header
`<float.h>`.

Số dấu phẩy động được mã hoá theo một chuỗi bit cụ thể ([flw[định
dạng IEEE-754|IEEE_754]] vô cùng phổ biến) trong các byte.

Đào sâu thêm, số đó về cơ bản được biểu diễn là _significand_ (phần
số, bản thân các chữ số có nghĩa, đôi khi còn gọi là _mantissa_) và
_exponent_ (số mũ), tức luỹ thừa mà ta dùng để nâng các chữ số lên.
Nhớ rằng số mũ âm làm số nhỏ đi.

Tưởng tượng ta dùng $10$ làm số để nâng theo số mũ. Ta có thể biểu
diễn các số sau bằng cách dùng significand là $12345$, và số mũ là
$-3$, $4$, và $0$ để mã hoá các giá trị dấu phẩy động sau:

$12345\times10^{-3}=12.345$

$12345\times10^4=123450000$

$12345\times10^0=12345$

Với tất cả các số đó, significand giữ nguyên. Khác biệt duy nhất là
số mũ.

Trên máy bạn, base của số mũ có lẽ là $2$, không phải $10$, vì máy
tính thích nhị phân. Bạn có thể kiểm bằng cách in macro `FLT_RADIX`.

Vậy ta có một số biểu diễn bằng một số byte, mã hoá theo cách nào đó.
Vì số mẫu bit có giới hạn, số dấu phẩy động biểu diễn được cũng có
giới hạn.

Cụ thể hơn, chỉ một số chữ số thập phân có nghĩa nhất định được biểu
diễn chính xác.

Làm sao để có nhiều hơn? Bạn dùng kiểu dữ liệu lớn hơn!

Và ta có vài cái. Ta biết `float` rồi, nhưng để có chính xác hơn thì
có `double`. Và cho chính xác hơn nữa, có `long double` (không liên
quan tới `long int` ngoài cái tên).

Spec không ghi mỗi kiểu nên chiếm bao nhiêu byte lưu trữ, nhưng trên
máy tôi, ta có thể thấy kích cỡ tăng tương đối:

[i[`double` type]<]
[i[`long double` type]<]

|Kiểu|`sizeof`|
|:-|-:|
|`float`|4|
|`double`|8|
|`long double`|16|

Nên mỗi kiểu (trên máy tôi) dùng các bit thêm đó cho độ chính xác cao
hơn.

Nhưng _chính xác tới đâu_? Bao nhiêu số thập phân có thể biểu diễn
được bằng các giá trị này?

Thì C cung cấp cho ta một đống macro trong `<float.h>` để giúp hình
dung chuyện đó.

Hơi lắt léo nếu bạn dùng hệ base-2 (nhị phân) để lưu số (thực ra là
gần như mọi người trên hành tinh này, có lẽ kể cả bạn), nhưng chịu
khó theo khi ta tính ra.
[i[`double` type]>]
[i[`long double` type]>]

### Bao nhiêu chữ số thập phân?

[i[Significant digits]<]
Câu hỏi triệu đô là, "Tôi có thể lưu bao nhiêu chữ số thập phân có
nghĩa trong một kiểu dấu phẩy động nhất định để lấy ra đúng số thập
phân đó khi in?"

Số chữ số thập phân bạn có thể lưu trong kiểu dấu phẩy động và chắc
chắn lấy lại đúng số đó khi in được cho bởi các macro sau:

[i[`FLT_DIG` macro]<]
[i[`DBL_DIG` macro]<]
[i[`LDBL_DIG` macro]<]

|Kiểu|Chữ số thập phân lưu được|Min|Máy tôi|
|:-|-:|-:|
|`float`|`FLT_DIG`|6|6|
|`double`|`DBL_DIG`|10|15|
|`long double`|`LDBL_DIG`|10|18|

[i[`DBL_DIG` macro]>]
[i[`LDBL_DIG` macro]>]

Trên máy tôi, `FLT_DIG` là 6, nên tôi có thể chắc rằng nếu in ra
`float` 6 chữ số, tôi sẽ lấy lại đúng thứ đó. (Có thể nhiều chữ số
hơn, một số số sẽ trở về đúng với nhiều chữ số hơn. Nhưng 6 thì chắc
chắn trở về.)

Ví dụ, in ra các `float` theo mẫu tăng dần chữ số, có vẻ ta tới 8 chữ
số trước khi có gì đó sai, nhưng sau đó ta quay về 7 chữ số đúng.

``` {.default}
0.12345
0.123456
0.1234567
0.12345678
0.123456791  <-- Things start going wrong
0.1234567910
```

Làm demo nữa. Trong code này ta có hai `float` đều chứa số có
`FLT_DIG` chữ số thập phân có nghĩa^[Chương trình này chạy như bình
luận cho biết trên hệ thống có `FLT_DIG` là `6`, dùng số dấu phẩy
động IEEE-754 base-2. Ngoài ra, bạn có thể có output khác.]. Rồi ta
cộng chúng lại, đáng lẽ được 12 chữ số thập phân có nghĩa. Nhưng thế
nhiều hơn ta có thể lưu trong một `float` và khôi phục về chuỗi đúng
được, nên ta thấy khi in ra, mọi thứ bắt đầu sai sau chữ số có nghĩa
thứ 7.


``` {.c .numberLines}
#include <stdio.h>
#include <float.h>

int main(void)
{
    // Both these numbers have 6 significant digits, so they can be
    // stored accurately in a float:

    float f = 3.14159f;
    float g = 0.00000265358f;

    printf("%.5f\n", f);   // 3.14159       -- correct!
    printf("%.11f\n", g);  // 0.00000265358 -- correct!

    // Now add them up
    f += g;                // 3.14159265358 is what f _should_ be

    printf("%.11f\n", f);  // 3.14159274101 -- wrong!
}
```

(Code trên có `f` sau các hằng số, cái này cho biết hằng đó là kiểu
`float`, khác với mặc định là `double`. Sẽ nói thêm sau.)

Nhớ rằng `FLT_DIG` là số chữ số an toàn bạn có thể lưu trong một
`float` và lấy lại đúng.

Đôi khi bạn có thể lấy ra thêm một hai chữ số. Nhưng đôi khi chỉ có
`FLT_DIG` chữ số trở về. Điều chắc chắn: nếu bạn lưu bất kỳ số chữ số
nào lên tới và bao gồm `FLT_DIG` trong một `float`, bạn chắc chắn lấy
lại chúng đúng.

Vậy là hết chuyện. `FLT_DIG`. Hết.

[i[`FLT_DIG` macro]>]

...Hay chưa hết?

### Chuyển sang thập phân và trở lại

Nhưng lưu số base 10 trong số dấu phẩy động và lấy ra chỉ mới là một
nửa câu chuyện.

Hoá ra số dấu phẩy động có thể mã hoá các số cần nhiều chữ số thập
phân hơn để in ra đầy đủ. Chỉ là số thập phân lớn của bạn có thể
không ánh xạ tới một trong các số đó.

Tức là, khi nhìn các số dấu phẩy động đi từ cái này sang cái kế, có
khoảng hở. Nếu bạn thử mã hoá một số thập phân trong khoảng hở đó, nó
sẽ dùng số dấu phẩy động gần nhất. Đó là vì sao bạn chỉ có thể mã
hoá `FLT_DIG` cho một `float`.

Nhưng còn các số dấu phẩy động _không_ nằm trong khoảng hở thì sao?
Cần bao nhiêu chữ số để in chúng ra chính xác?

Một cách đặt câu hỏi khác là với một số dấu phẩy động bất kỳ, tôi cần
giữ bao nhiêu chữ số thập phân nếu muốn chuyển số thập phân đó ngược
lại thành cùng số dấu phẩy động? Tức là tôi phải in bao nhiêu chữ số
base 10 để khôi phục **tất cả** các chữ số base 2 trong số gốc?

Đôi khi có thể chỉ vài cái. Nhưng cho chắc, bạn sẽ muốn chuyển sang
thập phân với một số vị trí thập phân an toàn nhất định. Số đó được
mã hoá trong các macro sau:

[i[`FLT_DECMIAL_DIG` macro]<]
[i[`DBL_DECMIAL_DIG` macro]<]
[i[`LDBL_DECMIAL_DIG` macro]<]
[i[`DECMIAL_DIG` macro]<]

|Macro|Mô tả|
|-----------|---------------------------------------------------|
|`FLT_DECIMAL_DIG`|Số chữ số thập phân mã hoá trong `float`.|
|`DBL_DECIMAL_DIG`|Số chữ số thập phân mã hoá trong `double`.|
|`LDBL_DECIMAL_DIG`|Số chữ số thập phân mã hoá trong `long double`.|
|`DECIMAL_DIG`|Giống như cách mã hoá rộng nhất, `LDBL_DECIMAL_DIG`.|

[i[`LDBL_DECMIAL_DIG` macro]>]
[i[`DECMIAL_DIG` macro]>]

Xem ví dụ với `DBL_DIG` là 15 (nên đó là hết ta có thể có trong một
hằng), nhưng `DBL_DECIMAL_DIG` là 17 (nên ta phải chuyển sang 17 số
thập phân để giữ hết mọi bit của `double` gốc).

Gán số có 15 chữ số có nghĩa `0.123456789012345` cho `x`, và gán số có
1 chữ số có nghĩa `0.0000000000000006` cho `y`.

``` {.default}
x is exact: 0.12345678901234500    Printed to 17 decimal places
y is exact: 0.00000000000000060
```

Nhưng cộng chúng lại. Đáng lẽ ra `0.1234567890123456`, nhưng thế là
nhiều hơn `DBL_DIG`, nên chuyện lạ có thể xảy ra... xem:

``` {.default}
x + y not quite right: 0.12345678901234559    Should end in 4560!
```

Đó là cái ta có vì in nhiều hơn `DBL_DIG`, đúng không? Nhưng xem
này... số đó, ở trên, được biểu diễn chính xác đúng y như vậy!

Nếu ta gán `0.12345678901234559` (17 chữ số) cho `z` rồi in, ta có:

``` {.default}
z is exact: 0.12345678901234559   17 digits correct! More than DBL_DIG!
```

Nếu ta cắt `z` về 15 chữ số, nó sẽ không còn là cùng một số. Đó là vì
sao để giữ toàn bộ bit của một `double`, ta cần `DBL_DECIMAL_DIG`, chứ
không chỉ `DBL_DIG` nhỏ hơn.

[i[`DBL_DECMIAL_DIG` macro]>]

Tất cả đã nói, rõ ràng khi nghịch số thập phân nói chung, không an
toàn khi in nhiều hơn [i[`FLT_DIG`
macro]]`FLT_DIG`, [i[`DBL_DIG` macro]]`DBL_DIG`, hay [i[`LDBL_DIG`
macro]]`LDBL_DIG` chữ số để hợp lý tương ứng với các số base 10 gốc
và bất kỳ phép toán tiếp theo.

Nhưng khi chuyển từ `float` sang biểu diễn thập phân rồi _trở lại_
`float`, chắc chắn dùng `FLT_DECIMAL_DIG` để bảo toàn chính xác mọi
bit. [i[`FLT_DECMIAL_DIG` macro]>]

[i[Significant digits]>]

## Kiểu hằng số

Khi bạn viết một số hằng, như `1234`, nó có kiểu. Nhưng kiểu gì? Xem
cách C quyết định kiểu hằng là gì, và cách ép nó chọn kiểu cụ thể.

### Hệ cơ số 16 và cơ số 8

Ngoài hệ thập phân cũ kỹ như bà ngoại hay nướng, C còn hỗ trợ hằng ở
các hệ cơ số khác.

[i[`0x` hexadecimal]<]

Nếu bạn mở đầu một số bằng `0x`, nó được đọc như số hex:

``` {.c}
int a = 0x1A2B;   // Hexadecimal
int b = 0x1a2b;   // Case doesn't matter for hex digits

printf("%x", a);  // Print a hex number, "1a2b"
```

[i[`0x` hexadecimal]>]


[i[`0` octal]<]
Nếu bạn mở đầu một số bằng `0`, nó được đọc như số bát phân:

``` {.c}
int a = 012;

printf("%o\n", a);  // Print an octal number, "12"
```

Cái này đặc biệt rắc rối với lập trình viên mới, khi họ cố độn số
thập phân bên trái bằng `0` để canh cho đẹp, vô tình đổi luôn cơ số
của số:

``` {.c}
int x = 11111;  // Decimal 11111
int y = 00111;  // Decimal 73 (Octal 111)
int z = 01111;  // Decimal 585 (Octal 1111)
```

[i[`0` octal]>]

#### Ghi chú về nhị phân

[i[`0b` binary]<]

Một mở rộng không chính thức^[Tôi khá ngạc nhiên là C chưa có cái
này trong spec. Trong tài liệu C99 Rationale, họ viết, "A proposal
to add binary constants was rejected due to lack of precedent and
insufficient utility." Nghe hơi ngốc khi so với một số tính năng
khác họ tống vào! Tôi cược một trong các bản phát hành tới sẽ có.]
trong nhiều compiler C cho phép bạn biểu diễn số nhị phân với tiền
tố `0b`:

``` {.c}
int x = 0b101010;    // Binary 101010

printf("%d\n", x);   // Prints 42 decimal
```

Không có format specifier của `printf()` nào để in số nhị phân. Bạn
phải làm từng ký tự một bằng toán tử bitwise.

[i[`0b` binary]>]

### Hằng số nguyên

[i[Integer constants]<]

Bạn có thể ép một hằng nguyên thành kiểu cụ thể bằng cách gắn hậu tố
chỉ kiểu đó.

Ta sẽ làm vài gán để demo, nhưng dev hầu hết bỏ hậu tố trừ khi cần
chính xác. Compiler khá giỏi trong việc đảm bảo các kiểu tương thích.

[i[`U` unsigned constant]<]
[i[`L` long constant]<]
[i[`LL` long long constant]<]
[i[`UL` unsigned long constant]<]
[i[`ULL` unsigned long long constant]<]

``` {.c}
int           x = 1234;
long int      x = 1234L;
long long int x = 1234LL

unsigned int           x = 1234U;
unsigned long int      x = 1234UL;
unsigned long long int x = 1234ULL;
```

Hậu tố có thể viết hoa hay thường. Và `U` cùng `L` hay `LL` có thể
xuất hiện cái nào trước cũng được.

|Kiểu|Hậu tố|
|:-|:-|
|`int`|Không|
|`long int`|`L`|
|`long long int`|`LL`|
|`unsigned int`|`U`|
|`unsigned long int`|`UL`|
|`unsigned long long int`|`ULL`|

Tôi có nhắc trong bảng rằng "không hậu tố" nghĩa là `int`... nhưng
thực tế phức tạp hơn thế.

Vậy chuyện gì xảy ra khi bạn có số không hậu tố như:

``` {.c}
int x = 1234;
```

Nó kiểu gì?

Cái C thường làm là chọn kiểu nhỏ nhất từ `int` trở lên có thể chứa
giá trị đó.

Nhưng cụ thể, điều đó cũng tuỳ vào cơ số của số (thập phân, hex, hay
bát phân).

Spec có một bảng ngon chỉ ra kiểu nào được dùng cho giá trị không
hậu tố nào. Thực ra, tôi sẽ chỉ chép y nguyên vào đây.

C11 §6.4.4.1¶5 ghi, "The type of an integer constant is the first of
the first of the corresponding list in which its value can be
represented."

Rồi tiếp theo là bảng này:

+----------------+------------------------+-------------------------+
|Hậu tố          |Hằng thập phân          |Hằng bát phân hoặc\      |
|                |                        |hexadecimal              |
+:===============+:=======================+:========================+
|không có        |`int`\                  |`int`\                   |
|                |`long int`              |`unsigned int`\          |
|                |                        |`long int`\              |
|                |                        |`unsigned long int`\     |
|                |                        |`long long int`\         |
|                |                        |`unsigned long long int`\|
|                |                        |                         |
+----------------+------------------------+-------------------------+
|`u` hay `U`     |`unsigned int`\         |`unsigned int`\          |
|                |`unsigned long int`\    |`unsigned long int`\     |
|                |`unsigned long long int`|`unsigned long long int`\|
|                |                        |                         |
+----------------+------------------------+-------------------------+
|`l` hay `L`     |`long int`\             |`long int`\              |
|                |`long long int`         |`unsigned long int`\     |
|                |                        |`long long int`\         |
|                |                        |`unsigned long long int`\|
|                |                        |                         |
+----------------+------------------------+-------------------------+
|Cả `u` hay `U`\ |`unsigned long int`\    |`unsigned long int`\     |
|và `l` hay `L`  |`unsigned long long int`|`unsigned long long int`\|
|                |                        |                         |
+----------------+------------------------+-------------------------+
|`ll` hay `LL`   |`long long int`         |`long long int`\         |
|                |                        |`unsigned long long int`\|
|                |                        |                         |
+----------------+------------------------+-------------------------+
|Cả `u` hay `U`\ |`unsigned long long int`|`unsigned long long int` |
|và `ll` hay `LL`|                        |                         |
+----------------+------------------------+-------------------------+

[i[`L` long constant]>]
[i[`LL` long long constant]>]
[i[`UL` unsigned long constant]>]
[i[`ULL` unsigned long long constant]>]

Ý nghĩa là, chẳng hạn, nếu bạn chỉ định một số như `123456789U`, C
sẽ thử xem nó có vừa `unsigned int` không. Nếu không vừa, nó sẽ thử
`unsigned long int`. Rồi `unsigned long long int`. Nó sẽ dùng kiểu
nhỏ nhất có thể chứa số đó.

[i[`U` unsigned constant]>]

[i[Integer constants]>]

### Hằng dấu phẩy động

[i[Floating point constants]<]

Bạn nghĩ hằng dấu phẩy động như `1.23` sẽ có kiểu mặc định là `float`
chứ?

Bất ngờ! Hoá ra số dấu phẩy động không hậu tố là kiểu `double`! Chúc
mừng sinh nhật muộn nhé!

[i[`F` float constant]<]
[i[`L` long double constant]<]

Bạn có thể ép nó thành kiểu `float` bằng cách gắn `f` (hoặc `F`,
không phân biệt hoa thường). Bạn có thể ép nó thành `long double`
bằng cách gắn `l` (hoặc `L`).

|Kiểu|Hậu tố|
|:-|:-|
|`float`|`F`|
|`double`|Không|
|`long double`|`L`|

Ví dụ:

``` {.c}
float x       = 3.14f;
double x      = 3.14;
long double x = 3.14L;
```

[i[`F` float constant]>]
[i[`L` long double constant]>]

Suốt thời gian qua ta vẫn làm thế này, đúng không?

``` {.c}
float x = 3.14;
```

Bên trái không phải `float` và bên phải `double` sao? Đúng vậy!
Nhưng C khá giỏi với chuyển đổi số tự động, nên hằng dấu phẩy động
không hậu tố còn phổ biến hơn có hậu tố. Sẽ nói thêm sau.

[i[Floating point constants]>]

#### Ký hiệu khoa học

[i[Scientific notation]<]

Nhớ trước đó ta nói về chuyện số dấu phẩy động có thể biểu diễn bởi
significand, base, và exponent chứ?

Có một cách viết phổ biến cho số kiểu đó, thể hiện ở đây kèm cái
tương đương dễ nhận hơn là thứ bạn có khi thực sự chạy tính toán:

$1.2345\times10^3 = 1234.5$

Viết số dạng $s\times b^e$ gọi là [flw[_scientific
notation_|Scientific_notation]] (ký hiệu khoa học). Trong C, chúng
được viết bằng "E notation", nên các cái này tương đương:

|Scientific Notation|E notation|
|:-|:-|
|$1.2345\times10^{-3}=0.0012345$|`1.2345e-3`|
|$1.2345\times10^8=123450000$|`1.2345e+8`|

Bạn có thể in số theo ký hiệu này với `%e`:

``` {.c}
printf("%e\n", 123456.0);  // Prints 1.234560e+05
```

Vài sự thật vui về ký hiệu khoa học:

* Bạn không bắt buộc phải viết với đúng một chữ số trước dấu thập
  phân. Có thể bao nhiêu chữ số cũng được đằng trước.

  ``` {.c}
  double x = 123.456e+3;  // 123456
  ```

  Tuy nhiên khi in ra, nó sẽ đổi số mũ để chỉ có một chữ số trước
  dấu thập phân.

* Dấu cộng có thể bỏ ở số mũ, vì đó là mặc định, nhưng theo tôi thấy
  ít thấy trong thực tế.

  ``` {.c}
  1.2345e10 == 1.2345e+10
  ```
* Bạn có thể áp hậu tố `F` hay `L` cho hằng E-notation:

  ``` {.c}
  1.2345e10F
  1.2345e10L
  ```

[i[Scientific notation]>]

#### Hằng dấu phẩy động hexadecimal

[i[Hex floating point constants]<]

Nhưng khoan, còn nhiều dấu phẩy động để xử!

Hoá ra cũng có hằng dấu phẩy động hexadecimal!

Chúng hoạt động tương tự số dấu phẩy động thập phân, nhưng bắt đầu
bằng `0x` y như số nguyên.

Điều lưu ý là bạn _phải_ chỉ định số mũ, và số mũ này tạo ra luỹ thừa
của 2. Tức là: $2^x$.

Rồi bạn dùng `p` thay cho `e` khi viết số:

Nên `0xa.1p3` là $10.0625\times2^3 == 80.5$.

Khi dùng hằng hex dấu phẩy động,
Ta có thể in hex scientific notation với `%a`:

``` {.c}
double x = 0xa.1p3;

printf("%a\n", x);  // 0x1.42p+6
printf("%f\n", x);  // 80.500000
```

[i[Hex floating point constants]>]


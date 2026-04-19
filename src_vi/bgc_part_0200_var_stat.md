<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Biến và câu lệnh

> _"It takes all kinds to make a world, does it not, Padre?"_ \
> _"So it does, my son, so it does."_
>
> ---Pirate Captain Thomas Bartholomew Red to the Padre, Pirates

Một chương trình C có thể chứa đủ thứ trên đời.

Ừ đấy.

Và vì nhiều lý do, sẽ dễ cho tất cả chúng ta nếu phân loại một vài thứ
hay gặp trong chương trình, để ai nấy đều rõ chúng ta đang nói về cái
gì.

## Biến

[i[Variables]<]Người ta hay nói "biến giữ giá trị". Nhưng một cách nghĩ
khác là: biến là một cái tên dễ đọc đối với con người, dùng để tham
chiếu tới một mẩu dữ liệu nào đó trong bộ nhớ.

Chúng ta sẽ dừng lại một nhịp để he hé nhìn xuống cái hang thỏ mang tên
pointer (con trỏ). Đừng lo lắng gì cả.

Bạn có thể hình dung bộ nhớ như một mảng khổng lồ gồm các byte^[Một
"byte" thường là một số nhị phân 8 bit. Cứ coi như một số nguyên chỉ có
thể chứa giá trị từ 0 đến 255. Về mặt kỹ thuật, C cho phép byte có số
bit bất kỳ, và nếu muốn chỉ rõ ràng một số 8 bit, bạn nên dùng từ
_octet_. Nhưng lập trình viên mặc nhiên hiểu "byte" là 8 bit trừ khi
bạn nói rõ khác đi.]. Dữ liệu được lưu trong "mảng" này^[Tôi đang đơn
giản hoá cực độ cách bộ nhớ hiện đại hoạt động. Nhưng mô hình tưởng
tượng này vẫn dùng được, nên xin thứ lỗi.]. Nếu một số lớn hơn một
byte, nó được lưu trong nhiều byte. Vì bộ nhớ giống như một mảng, mỗi
byte có thể được tham chiếu qua chỉ số của nó. Chỉ số này vào bộ nhớ
còn được gọi là _address_ (địa chỉ), _location_ (vị trí), hay
_pointer_ (con trỏ).

Khi bạn có một biến trong C, giá trị của biến đó nằm trong bộ nhớ ở
_đâu đó_, tại một địa chỉ nào đó. Dĩ nhiên. Chứ nó còn ở chỗ nào được
nữa? Nhưng nhắc tới một giá trị bằng địa chỉ số thì thật khổ sở, nên
ta đặt cho nó cái tên, và cái tên đó chính là biến.

Lý do tôi nói hết đống này có hai:

1. Nó sẽ giúp bạn dễ hiểu biến con trỏ sau này, chúng là biến chứa địa
   chỉ của các biến khác!
2. Và nó cũng giúp bạn dễ hiểu con trỏ sau này.

Tóm lại, biến là một cái tên cho mẩu dữ liệu được lưu trong bộ nhớ ở
một địa chỉ nào đó.

### Tên biến

[i[Variables]<]Bạn có thể dùng các ký tự trong khoảng 0-9, A-Z, a-z, và
dấu gạch dưới cho tên biến, với các luật sau:

* Không được bắt đầu tên biến bằng chữ số 0-9.
* Không được bắt đầu tên biến bằng hai dấu gạch dưới.
* Không được bắt đầu tên biến bằng một dấu gạch dưới rồi tới chữ hoa
  A-Z.

Với Unicode thì cứ thử xem. Trong spec §D.2 có vài luật nói về dải
codepoint Unicode nào được phép ở phần nào của định danh, nhưng viết
hết ra thì dài quá, và có lẽ đời bạn cũng chẳng cần nghĩ tới.[i[Variables]>]

### Kiểu biến

[i[Types]]Tuỳ vào ngôn ngữ bạn đã biết, có thể bạn đã quen với khái
niệm type (kiểu), có thể chưa. Nhưng C hơi khó tính ở chỗ này, nên ta
nên ôn lại một chút.

Vài kiểu ví dụ, thuộc loại cơ bản nhất:

[i[`int` type]][i[`float` type]][i[`char` type]][i[`char *` type]]

|Kiểu|Ví dụ|Kiểu C|
|:---|------:|:-----|
|Số nguyên|`3490`|`int`|
|Số dấu phẩy động|`3.14159`|`float`^[Tôi đang nói dối một chút ở đây. Về kỹ thuật `3.14159` là kiểu `double`, nhưng ta chưa tới chỗ đó và tôi muốn bạn gắn `float` với "Floating Point", và C sẽ vui vẻ ép kiểu đó thành `float`. Tóm lại, đừng bận tâm tới khi gặp lại sau.]|
|Ký tự (đơn)|`'c'`|`char`|
|Chuỗi|`"Hello, world!"`|`char *`^[Đọc là "pointer to a char" hoặc "char pointer" (con trỏ tới char). "Char" là viết tắt của character (ký tự). Dù tôi không tìm thấy nghiên cứu nào, theo quan sát thì đa số đọc là "char", một thiểu số đọc "car", và lác đác đọc "care". Ta sẽ nói kỹ hơn về con trỏ sau.]|

C cố gắng tự động chuyển đổi giữa hầu hết các kiểu số khi bạn yêu cầu.
Ngoài chuyện đó, mọi phép chuyển đổi đều phải làm bằng tay, đặc biệt
là giữa chuỗi và số.

Gần như mọi kiểu trong C đều là biến thể của những kiểu trên.

Trước khi dùng một biến, bạn phải _khai báo_ (declare) biến đó và cho
C biết biến chứa kiểu gì. Một khi đã khai báo, kiểu của biến không thể
đổi sau này lúc chạy chương trình. Đặt là gì thì nó là thế cho đến khi
rơi ra khỏi scope (phạm vi) và bị vũ trụ hấp thụ lại.

Hãy lấy code "Hello, world" trước đó và thêm vài biến vào:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i;    // Holds signed integers, e.g. -3, -2, 0, 1, 10
    float f;  // Holds signed floating point numbers, e.g. -3.1416

    printf("Hello, World!\n");  // Ah, blessed familiarity
}
```

Đó! Ta đã khai báo vài biến. Chưa dùng đến, và cả hai đều chưa được
khởi tạo. Một biến giữ số nguyên, biến kia giữ số dấu phẩy động (về cơ
bản là số thực, nếu bạn có nền toán).

[i[Variables-->uninitialized]]Biến chưa được khởi tạo có giá trị không
xác định^[Nói chung ta bảo chúng có giá trị "ngẫu nhiên", nhưng thật ra
không phải số ngẫu nhiên thật, thậm chí cũng chẳng phải ngẫu nhiên giả
lập.]. Chúng phải được khởi tạo, nếu không bạn phải giả định chúng
chứa một số nhảm nào đó.

> Đây là một trong những chỗ C có thể "cắn" bạn. Theo kinh nghiệm của
> tôi, phần lớn trường hợp cái giá trị không xác định đó là số
> không... nhưng nó có thể khác nhau giữa các lần chạy! Đừng bao giờ
> giả định giá trị sẽ là 0, kể cả khi bạn thấy đúng là 0. _Luôn luôn_
> khởi tạo biến một cách rõ ràng trước khi dùng^[Điều này không hoàn
> toàn đúng 100%. Khi tới phần static storage duration (thời gian lưu
> trữ tĩnh), bạn sẽ thấy một số biến được tự động khởi tạo về 0. Nhưng
> cách an toàn là luôn tự khởi tạo.].

Khoan, bạn muốn lưu số vào mấy biến đó à? Điên rồ!

Thì cứ làm đi:
[i[`=` assignment operator]]

``` {.c .numberLines}
int main(void)
{
    int i;

    i = 2; // Assign the value 2 into the variable i

    printf("Hello, World!\n");
}
```

Đỉnh. Ta vừa lưu một giá trị. Giờ in nó ra nào.

[i[`printf()` function]<]Ta sẽ in bằng cách truyền _hai_ đối số tuyệt
vời cho hàm `printf()`. Đối số thứ nhất là một chuỗi mô tả cần in gì
và in như thế nào (gọi là _format string_), và đối số thứ hai là giá
trị cần in, cụ thể là thứ đang nằm trong biến `i`.

`printf()` quét chuỗi format tìm các chuỗi đặc biệt bắt đầu bằng dấu
phần trăm (`%`) để biết phải in gì. Ví dụ, khi gặp `%d`, nó nhìn vào
tham số kế tiếp và in ra dưới dạng số nguyên. Gặp `%f` thì in dưới
dạng float. Gặp `%s` thì in chuỗi.

Nhờ vậy, ta có thể in ra giá trị của nhiều kiểu khác nhau như này:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i = 2;
    float f = 3.14;
    char *s = "Hello, world!";  // char * ("char pointer") is the string type

    printf("%s  i = %d and f = %f!\n", s, i, f);
}
```

Và output sẽ là:

``` {.default}
Hello, world!  i = 2 and f = 3.14!
```

Kiểu này, `printf()` có thể giống với các loại chuỗi format hay chuỗi
tham số hoá trong những ngôn ngữ khác mà bạn đã quen.
[i[`printf()` function]>]

***LƯU Ý:*** Trong các mục tiếp theo, tôi mặc định bạn đã khai báo các
biến từ trước. Nếu ví dụ dùng số nguyên `i` và `j`, cứ giả định đâu đó
phía trên ví dụ tôi đã có:

``` {.c}
int i, j;
```

### Kiểu Boolean

[i[Boolean types]<]C có kiểu Boolean, true hay false?

`1`!

Về lịch sử, C không có kiểu Boolean, và vài người có thể tranh cãi là
bây giờ vẫn chưa có.

Trong C, `0` nghĩa là "false", và khác không nghĩa là "true".

Nên `1` là true. `-37` cũng true. Và `0` là false.

Bạn có thể khai báo Boolean như `int`:

``` {.c}
int x = 1;

if (x) {
    printf("x is true!\n");
}
```

Trong C23, bạn có `bool`, `true`, và `false` thực sự. Trước đó, nếu bạn
có phiên bản C đủ mới, có thể `#include <stdbool.h>`[i[`stdbool.h` header file]]
để có thứ tương tự.

``` {.c .numberLines}
#include <stdio.h>
#include <stdbool.h>  // not needed in C23

int main(void) {
    bool x = true;

    if (x) {
        printf("x is true!\n");
    }
}
```

Về lý thuyết bạn nên gán biến `bool` là `true`, `false`, hoặc kết quả
của biểu thức trả ra true/false, nhưng thật ra bạn có thể ép đủ thứ
thành `bool`. Có vài luật cụ thể, nhưng đại khái thứ nào giống-số-không
thường thành `false`, còn thứ khác-không thì thành true.

Nhưng cẩn thận nếu bạn trộn lẫn, vì giá trị số của `true` là `1`, gần
như chắc chắn[^B1CA], và nếu bạn trông cậy vào một giá trị dương khác
mang nghĩa true, bạn có thể bị lệch. Ví dụ:

``` {.c .numberLines}
printf("%d\n", true == 12);  // Prints "0", false!
```

[^B1CA]: Về kỹ thuật chỉ một bit của `char` được dùng để biểu diễn
    `bool`, nên nó chỉ có thể là 0 hoặc 1. Chỉ là các bit còn lại
    (padding) của `char` thì không được quy định rõ. Với `false`, chắc
    chắn phải toàn 0. Nhưng với `true`, tôi không chắc là nó phải toàn
    0 hay không.

[i[Boolean types]>]

## Toán tử và biểu thức {#operators}

Các toán tử trong C chắc đã quen thuộc với bạn từ ngôn ngữ khác. Ta
lướt nhanh qua một số ở đây.

(Có cả đống chi tiết hơn thế này, nhưng trong mục này ta sẽ làm đủ để
bắt đầu thôi.)


### Số học

[i[Arithmetic Operators]()]Hy vọng mấy cái này quen thuộc:
[i[`+` addition operator]] [i[`-` subtraction operator]]
[i[`*` multiplication operator]] [i[`/` division operator]]
[i[`%` modulus operator]]

``` {.c}
i = i + 3;  // Addition (+) and assignment (=) operators, add 3 to i
i = i - 8;  // Subtraction, subtract 8 from i
i = i * 9;  // Multiplication
i = i / 2;  // Division
i = i % 5;  // Modulo (division remainder)
```

Có các biến thể viết tắt cho tất cả mấy dòng trên. Mỗi dòng có thể viết
ngắn gọn hơn như sau:
[i[`+=` assignment operator]] [i[`-=` assignment operator]]
[i[`*=` assignment operator]] [i[`/=` assignment operator]]
[i[`%=` assignment operator]]

``` {.c}
i += 3;  // Same as "i = i + 3", add 3 to i
i -= 8;  // Same as "i = i - 8"
i *= 9;  // Same as "i = i * 9"
i /= 2;  // Same as "i = i / 2"
i %= 5;  // Same as "i = i % 5"
```

Không có toán tử luỹ thừa. Bạn sẽ phải dùng một trong các biến thể của
hàm `pow()`[i[pow()]T] trong `math.h`.

Giờ thì nhảy vào mấy thứ lạ hơn mà có thể ngôn ngữ khác của bạn không
có![i[Arithmetic Operators]>]

### Toán tử ba ngôi

[i[`?:` ternary operator]<]C cũng có _toán tử ba ngôi_ (ternary
operator). Đây là một biểu thức mà giá trị của nó phụ thuộc vào kết
quả của một điều kiện được nhúng trong biểu thức.

``` {.c}
// If x > 10, add 17 to y. Otherwise add 37 to y.

y += x > 10? 17: 37;
```

Rối thật! Đọc nhiều sẽ quen. Để giúp một chút, tôi viết lại biểu thức
trên bằng câu lệnh `if`:

``` {.c}
// This expression:

y += x > 10? 17: 37;

// is equivalent to this non-expression:

if (x > 10)
    y += 17;
else
    y += 37;
```

So sánh hai đoạn cho tới khi bạn nhận ra từng thành phần của toán tử
ba ngôi.

Hoặc một ví dụ khác, in ra xem số trong `x` là chẵn hay lẻ:

``` {.c}
printf("The number %d is %s.\n", x, x % 2 == 0? "even": "odd");
```

Format specifier `%s` trong `printf()`[i[printf()]T] nghĩa là in một
chuỗi. Nếu biểu thức `x % 2` cho ra `0`, giá trị của toàn bộ biểu thức
ba ngôi là chuỗi `"even"`. Ngược lại là chuỗi `"odd"`. Khá ngầu!

Cần lưu ý rằng toán tử ba ngôi không phải flow control (điều khiển
luồng) như câu lệnh `if`. Nó chỉ là một biểu thức cho ra một giá trị.
[i[`?:` ternary operator]>]

### Tăng giảm tiền tố và hậu tố

[i[`++` increment operator]<] [i[`--` decrement operator]<]
Giờ nghịch tiếp một thứ mà có lẽ bạn chưa thấy.

Đây là cặp toán tử huyền thoại post-increment và post-decrement:

``` {.c}
i++;        // Add one to i (post-increment)
i--;        // Subtract one from i (post-decrement)
```

Thường thì chúng được dùng như phiên bản ngắn của:

``` {.c}
i += 1;        // Add one to i
i -= 1;        // Subtract one from i
```

nhưng tinh ý hơn thì chúng khác một chút, mấy anh bạn ranh mãnh này.

Xem luôn biến thể pre-increment và pre-decrement:

``` {.c}
++i;        // Add one to i (pre-increment)
--i;        // Subtract one from i (pre-decrement)
```

Với pre-increment và pre-decrement, giá trị của biến được tăng hoặc
giảm _trước_ khi biểu thức được tính. Sau đó biểu thức được tính với
giá trị mới.

Với post-increment và post-decrement, giá trị của biểu thức được tính
trước bằng giá trị hiện tại, _rồi sau đó_ giá trị mới được tăng hay
giảm sau khi giá trị của biểu thức đã được xác định.

Bạn có thể nhúng chúng vào biểu thức như sau:

``` {.c}
i = 10;
j = 5 + i++;  // Compute 5 + i, _then_ increment i

printf("%d, %d\n", i, j);  // Prints 11, 15
```

So sánh với toán tử pre-increment:

``` {.c}
i = 10;
j = 5 + ++i;  // Increment i, _then_ compute 5 + i

printf("%d, %d\n", i, j);  // Prints 11, 16
```

Kỹ thuật này được dùng rất thường xuyên khi truy cập và thao tác mảng
và con trỏ. Nó cho bạn cách dùng giá trị trong một biến, đồng thời
tăng hoặc giảm giá trị đó trước hoặc sau khi dùng.

Nhưng chỗ bạn hay thấy nhất là trong vòng lặp `for`:

``` {.c}
for (i = 0; i < 10; i++)
    printf("i is %d\n", i);
```

Để sau nói tiếp.
[i[`++` increment operator]>] [i[`--` decrement operator]>]

### Toán tử dấu phẩy

[i[`,` comma operator]<]
Đây là một cách ít dùng để ngăn các biểu thức sẽ được chạy từ trái
sang phải:

``` {.c}
x = 10, y = 20;  // First assign 10 to x, then 20 to y
```

Nghe hơi vô nghĩa, vì bạn có thể thay dấu phẩy bằng dấu chấm phẩy
đúng không?

``` {.c}
x = 10; y = 20;  // First assign 10 to x, then 20 to y
```

Nhưng hai cái hơi khác nhau đấy. Cái sau là hai biểu thức riêng biệt,
còn cái trước là một biểu thức duy nhất!

Với toán tử dấu phẩy, giá trị của biểu thức dấu phẩy là giá trị của
biểu thức ngoài cùng bên phải:

``` {.c}
x = (1, 2, 3);

printf("x is %d\n", x);  // Prints 3, because 3 is rightmost in the comma list
```

Nhưng ngay cả thế cũng khá gượng gạo. Một chỗ phổ biến hay dùng toán
tử dấu phẩy là trong vòng lặp `for` để làm nhiều việc trong từng phần
của câu lệnh:

``` {.c}
for (i = 0, j = 10; i < 100; i++, j++)
    printf("%d, %d\n", i, j);
```

Ta sẽ quay lại phần này sau.
[i[`,` comma operator]>]

### Toán tử điều kiện

[i[Conditional Operators]<]
Với giá trị Boolean, ta có cả loạt toán tử chuẩn:
[i[Comparison operators]] [i[`==` equality operator]]
[i[`!=` inequality operator]] [i[`<` less than operator]]
[i[`>` greater than operator]] [i[`<=` less or equal operator]]
[i[`>=` greater or equal operator]]

``` {.c}
a == b;  // True if a is equivalent to b
a != b;  // True if a is not equivalent to b
a < b;   // True if a is less than b
a > b;   // True if a is greater than b
a <= b;  // True if a is less than or equal to b
a >= b;  // True if a is greater than or equal to b
```

Đừng lẫn phép gán (`=`) với phép so sánh (`==`)! Hai dấu bằng là so
sánh, một dấu bằng là gán.

Ta có thể dùng biểu thức so sánh với câu lệnh `if`:

``` {.c}
if (a <= 10)
    printf("Success!\n");
```
[i[Conditional Operators]>]

### Toán tử Boolean

[i[Boolean Operators]<]
Ta có thể nối hoặc biến đổi các biểu thức điều kiện bằng toán tử
Boolean cho _and_, _or_, và _not_.
[i[`&&` boolean AND]]
[i[`!` boolean NOT]]
[i[`||` boolean OR]]

|Toán tử|Nghĩa Boolean|
|:------:|:-------------:|
|`&&`|and|
|`||`|or|
|`!`|not|

Ví dụ Boolean "and":

``` {.c}
// Do something if x less than 10 and y greater than 20:

if (x < 10 && y > 20)
    printf("Doing something!\n");
```

Ví dụ Boolean "not":

``` {.c}
if (!(x < 12))
    printf("x is not less than 12\n");
```

`!` có độ ưu tiên cao hơn các toán tử Boolean khác, nên trong trường
hợp này ta phải dùng dấu ngoặc.

Dĩ nhiên, thế thì cũng chỉ tương đương:

``` {.c}
if (x >= 12)
    printf("x is not less than 12\n");
```

nhưng tôi cần ví dụ mà!
[i[Boolean Operators]>]

### Toán tử `sizeof` {#sizeof-operator}

[i[`sizeof` operator]<]
Toán tử này cho bạn biết kích thước (tính bằng byte) mà một biến hoặc
một kiểu dữ liệu cụ thể chiếm trong bộ nhớ.

Chính xác hơn, nó cho biết kích thước (tính bằng byte) mà _kiểu của
một biểu thức cụ thể_ (có thể chỉ là một biến đơn) chiếm trong bộ nhớ.

Con số này có thể khác nhau trên các hệ thống khác nhau, trừ `char`[i[`char`
type]] và các biến thể của nó (luôn là 1 byte).

Và có thể hiện tại trông nó chưa hữu ích lắm, nhưng ta sẽ nhắc tới
đây đó, nên đáng nói qua.

Vì nó tính số byte cần để lưu một kiểu, bạn có thể nghĩ nó sẽ trả về
một `int`. Hoặc... vì kích thước không thể âm, có lẽ trả về `unsigned`?

[i[`size_t` type]<]
Hoá ra C có một kiểu đặc biệt cho giá trị trả về từ `sizeof`. Đó là
`size_t`, đọc là "_size tee_"^[Chữ `_t` là viết tắt của `type`.]. Tất
cả những gì ta biết là nó là kiểu số nguyên unsigned có thể chứa kích
thước tính bằng byte của bất cứ thứ gì bạn đưa vào `sizeof`.

`size_t` xuất hiện ở rất nhiều nơi khi ta truyền hoặc trả về đếm số
lượng. Cứ coi nó như một giá trị đại diện cho một phép đếm.
[i[`size_t` type]>]

Bạn có thể lấy `sizeof` của một biến hoặc biểu thức:

``` {.c}
int a = 999;

// %zu is the format specifier for type size_t
// If your compiler balks at the "z" part, leave it off

printf("%zu\n", sizeof a);      // Prints 4 on my system
printf("%zu\n", sizeof(2 + 7)); // Prints 4 on my system
printf("%zu\n", sizeof 3.14);   // Prints 8 on my system

// If you need to print out negative size_t values, use %zd
```

Nhớ nhé: đó là kích thước tính bằng byte của _kiểu_ của biểu thức, chứ
không phải kích thước của chính biểu thức. Đó là lý do kích thước của
`2+7` bằng với kích thước của `a`, cả hai đều kiểu `int`. Ta sẽ gặp
lại con số `4` ở khối code kế tiếp...

...Ở đó bạn sẽ thấy có thể lấy `sizeof` của một kiểu (lưu ý cần dấu
ngoặc quanh tên kiểu, khác với biểu thức):

``` {.c}
printf("%zu\n", sizeof(int));   // Prints 4 on my system
printf("%zu\n", sizeof(char));  // Prints 1 on all systems
```

Một điều quan trọng cần nhớ: `sizeof` là phép toán _thời điểm biên
dịch_ (compile-time)^[Trừ trường hợp variable length array, nhưng
chuyện đó để dịp khác.]. Kết quả của biểu thức được xác định toàn bộ
lúc biên dịch, chứ không phải lúc chạy.

Ta sẽ tận dụng điều này sau.
[i[`sizeof` operator]>]


## Điều khiển luồng

[i[Flow Control]<]
Boolean thì tốt, nhưng chẳng đi tới đâu nếu ta không điều khiển được
luồng chương trình. Hãy nhìn qua một số cấu trúc: `if`, `for`,
`while`, và `do-while`.

Trước hết, một ghi chú chung hướng về phía trước, về câu lệnh và khối
câu lệnh, gửi tới bạn bởi lập trình viên C thân thiện ở khu phố của
bạn:

Sau một thứ như `if` hay `while`, bạn có thể đặt một câu lệnh duy nhất
để thực thi, hoặc một khối các câu lệnh thực thi lần lượt theo thứ tự.

[i[`if` statement]<]
Bắt đầu với một câu lệnh đơn:

``` {.c}
if (x == 10) printf("x is 10\n");
```

Cái này cũng đôi khi được viết trên một dòng riêng. (Whitespace trong
C phần lớn không có ý nghĩa, không như Python.)

``` {.c}
if (x == 10)
    printf("x is 10\n");
```

Nhưng nếu bạn muốn nhiều thứ xảy ra do điều kiện thì sao? Bạn có thể
dùng dấu ngoặc nhọn ngoằn ngoèo để đánh dấu một _block_ hay _compound
statement_ (khối hay câu lệnh ghép).

``` {.c}
if (x == 10) {
    printf("x is 10\n");
    printf("And also this happens when x is 10\n");
}
```

Có một phong cách khá phổ biến là _luôn luôn_ dùng ngoặc nhọn ngay cả
khi không cần thiết:

``` {.c}
if (x == 10) {
    printf("x is 10\n");
}
```

Một số dev thấy code dễ đọc hơn và tránh được lỗi kiểu như ví dụ sau,
nhìn qua thì có vẻ cả hai dòng đều nằm trong khối `if`, nhưng thực ra
không phải:

``` {.c}
// BAD ERROR EXAMPLE

if (x == 10)
    printf("This happens if x is 10\n");
    printf("This happens ALWAYS\n");  // Surprise!! Unconditional!
```

`while` và `for` và các cấu trúc lặp khác hoạt động giống như các ví
dụ trên. Nếu bạn muốn làm nhiều việc trong vòng lặp hoặc sau `if`, cứ
bọc chúng trong ngoặc nhọn.

Nói cách khác, `if` sẽ chạy đúng một thứ ngay sau nó. Và "một thứ" đó
có thể là một câu lệnh đơn hoặc một khối các câu lệnh.
[i[`if` statement]>]


### Câu lệnh `if`-`else` {#ifstat}

[i[`if`-`else` statement]<]
Ta đã dùng `if` trong nhiều ví dụ rồi, vì có lẽ bạn đã thấy nó trong
ngôn ngữ nào đó, nhưng đây là thêm một ví dụ nữa:

``` {.c}
int i = 10;

if (i > 10) {
    printf("Yes, i is greater than 10.\n");
    printf("And this will also print if i is greater than 10.\n");
}

if (i <= 10) printf("i is less than or equal to 10.\n");
```

Trong code ví dụ, thông báo sẽ được in nếu `i` lớn hơn 10, còn không
thì chạy tiếp xuống dòng kế. Để ý các dấu ngoặc nhọn sau câu lệnh
`if`. Nếu điều kiện đúng, hoặc là câu lệnh/biểu thức đầu tiên ngay sau
`if` sẽ chạy, hoặc là khối code trong dấu ngoặc nhọn sau `if` sẽ chạy.
Hành vi _khối code_ (code block) này đúng với mọi câu lệnh.

Dĩ nhiên, vì C cũng vui theo kiểu này, bạn có thể làm gì đó khi điều
kiện sai bằng mệnh đề `else`:

``` {.c}
int i = 99;

if (i == 10)
    printf("i is 10!\n");
else {
    printf("i is decidedly not 10.\n");
    printf("Which irritates me a little, frankly.\n");
}
```

Và bạn thậm chí có thể xâu chuỗi để kiểm tra nhiều điều kiện khác
nhau, như này:

``` {.c}
int i = 99;

if (i == 10)
    printf("i is 10!\n");

else if (i == 20)
    printf("i is 20!\n");

else if (i == 99) {
    printf("i is 99! My favorite\n");
    printf("I can't tell you how happy I am.\n");
    printf("Really.\n");
}
    
else
    printf("i is some crazy number I've never heard of.\n");
```

Dù nếu đi hướng đó, nhớ xem câu lệnh [`switch`](#switch-statement) để
có giải pháp có khả năng tốt hơn. Cái gượng là `switch` chỉ làm việc
với so sánh bằng với hằng số. Cascade `if`-`else` ở trên có thể so
sánh bất đẳng, khoảng, biến, hay bất cứ thứ gì bạn dựng được trong
biểu thức điều kiện.
[i[`if`-`else` statement]>]

### Câu lệnh `while` {#whilestat}

[i[`while` statement]<]
`while` là cấu trúc lặp bình dân. Làm một việc trong khi biểu thức
điều kiện còn đúng.

Làm một cái nào!

``` {.c}
// Print the following output:
//
//   i is now 0!
//   i is now 1!
//   [ more of the same between 2 and 7 ]
//   i is now 8!
//   i is now 9!

int i = 0;

while (i < 10) {
    printf("i is now %d!\n", i);
    i++;
}

printf("All done!\n");
```

Thế là bạn có một vòng lặp cơ bản. C cũng có `for` có lẽ sẽ gọn hơn
cho ví dụ đó.

Một kiểu không hiếm gặp khi dùng `while` là lặp vô hạn, lặp khi điều
kiện luôn đúng:

``` {.c}
while (1) {
    printf("1 is always true, so this repeats forever.\n");
}
```
[i[`while` statement]>]


### Câu lệnh `do-while` {#dowhilestat}

[i[`do`-`while` statement]<]
Giờ đã thuần được `while`, hãy ngó qua ông anh họ gần của nó,
`do-while`.

Về cơ bản hai thằng giống nhau, chỉ khác là nếu điều kiện sai ngay lần
đầu, `do-while` vẫn chạy một lần, còn `while` không chạy lần nào.
Nói cách khác, phép kiểm tra xem có thực thi khối hay không xảy ra ở
_cuối_ khối với `do-while`. Còn với `while` là ở _đầu_ khối.

Xem ví dụ:

``` {.c}
// Using a while statement:

i = 10;

// this is not executed because i is not less than 10:
while(i < 10) {
    printf("while: i is %d\n", i);
    i++;
}

// Using a do-while statement:

i = 10;

// this is executed once, because the loop condition is not checked until
// after the body of the loop runs:

do {
    printf("do-while: i is %d\n", i);
    i++;
} while (i < 10);

printf("All done!\n");
```

Để ý rằng trong cả hai trường hợp, điều kiện lặp là sai ngay từ đầu.
Thế nên với `while`, vòng lặp thất bại và khối code sau đó không bao
giờ chạy. Còn với `do-while`, điều kiện được kiểm tra _sau khi_ khối
code chạy, nên nó luôn chạy ít nhất một lần. Ở đây, nó in thông báo,
tăng `i`, rồi kiểm tra thấy điều kiện sai, và chạy tiếp tới dòng "All
done!".

Bài học rút ra là: nếu bạn muốn vòng lặp chạy ít nhất một lần, bất kể
điều kiện lặp thế nào, hãy dùng `do-while`.

Tất cả ví dụ trên có lẽ đã tốt hơn nếu dùng `for`. Làm một cái đỡ tiền
định hơn: lặp cho đến khi một số ngẫu nhiên nhất định xuất hiện!

``` {.c .numberLines}
#include <stdio.h>   // For printf
#include <stdlib.h>  // For rand

int main(void)
{
    int r;

    do {
        r = rand() % 100; // Get a random number between 0 and 99
        printf("%d\n", r);
    } while (r != 37);    // Repeat until 37 comes up
}
```

Ghi chú bên lề: bạn có chạy cái đó nhiều lần không? Nếu có, bạn có để
ý chính dãy số đó lại xuất hiện. Lại. Và lại? Đó là vì `rand()` là một
bộ sinh số giả ngẫu nhiên, nó cần được _seed_ (gieo mầm) bằng một số
khác nhau để sinh dãy khác nhau. Xem hàm
[fl[`srand()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-srand]]
để biết chi tiết. [i[`do`-`while` statement]>]

### Câu lệnh `for` {#forstat}

[i[`for` statement]<]
Chào mừng đến với một trong những vòng lặp phổ biến nhất thế giới!
Vòng lặp `for`!

Đây là vòng lặp tuyệt vời nếu bạn biết trước số lần cần lặp.

Bạn có thể làm việc tương tự chỉ với `while`, nhưng `for` giúp code
sạch hơn.

Đây là hai đoạn code tương đương, để ý `for` chỉ là cách biểu diễn gọn
hơn:

``` {.c}
// Print numbers between 0 and 9, inclusive...

// Using a while statement:

i = 0;
while (i < 10) {
    printf("i is %d\n", i);
    i++;
}

// Do the exact same thing with a for-loop:

for (i = 0; i < 10; i++) {
    printf("i is %d\n", i);
}
```

Đúng vậy, thưa các bạn, hai đoạn làm chính xác cùng một việc. Nhưng
bạn có thể thấy `for` gọn hơn và dễ nhìn hơn. (Dân JavaScript lúc này
sẽ thấy rõ nguồn gốc C của nó.)

Nó được chia thành ba phần, ngăn bởi dấu chấm phẩy. Phần đầu là khởi
tạo, phần hai là điều kiện lặp, và phần ba là thứ xảy ra ở cuối khối
nếu điều kiện lặp còn đúng. Cả ba phần đều tuỳ chọn.

``` {.c}
for (initialize things; loop if this is true; do this after each loop)
```

Lưu ý rằng vòng lặp sẽ không chạy dù chỉ một lần nếu điều kiện lặp sai
ngay từ đầu.

> **`for`-loop fun fact!**
>
> Bạn có thể dùng toán tử dấu phẩy để làm nhiều việc trong từng vế của
> `for`!
>
> ``` {.c}
> for (i = 0, j = 999; i < 10; i++, j--) {
>     printf("%d, %d\n", i, j);
> }
> ```

<!-- ` -->
Một `for` rỗng sẽ chạy mãi mãi:

``` {.c}
for(;;) {  // "forever"
    printf("I will print this again and again and again\n" );
    printf("for all eternity until the heat-death of the universe.\n");

    printf("Or until you hit CTRL-C.\n");
}
```
[i[`for` statement]>]

### Câu lệnh `switch` {#switch-statement}

[i[`switch` statement]<]
Tuỳ ngôn ngữ bạn đang dùng, có thể bạn quen hoặc chưa quen với
`switch`, hoặc phiên bản C của nó có thể chặt chẽ hơn bạn tưởng. Đây
là câu lệnh cho phép bạn thực hiện nhiều hành động khác nhau tuỳ thuộc
vào giá trị của một biểu thức số nguyên.

Cơ bản là nó tính biểu thức ra một giá trị số nguyên, rồi nhảy đến
[i[`case` statement]<]`case` tương ứng với giá trị đó. Thực thi tiếp
tục từ điểm ấy. Nếu gặp câu lệnh `break`[i[`break` statement]<], thực
thi nhảy ra khỏi `switch`.

Đây là ví dụ, với một số dê cho trước, ta in ra cảm nhận về số dê đó.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int goat_count = 2;

    switch (goat_count) {
        case 0:
            printf("You have no goats.\n");
            break;

        case 1:
            printf("You have a singular goat.\n");
            break;

        case 2:
            printf("You have a brace of goats.\n");
            break;

        default:
            printf("You have a bona fide plethora of goats!\n");
            break;
    }
}
```

Trong ví dụ đó, `switch` nhảy tới `case 2` và chạy từ đó. Khi (nếu)
gặp `break`, nó nhảy ra khỏi `switch`.
[i[`break` statement]>]

Bạn cũng có thể thấy nhãn `default`[i[`default` label]] ở dưới cùng.
Cái này chạy khi không `case` nào khớp.

Mọi `case`, kể cả `default`, đều tuỳ chọn. Và chúng có thể xuất hiện
theo bất kỳ thứ tự nào, nhưng thông thường `default`, nếu có, được đặt
cuối cùng.[i[`case` statement]>]

Cho nên toàn bộ hoạt động như một cascade `if`-`else`:

``` {.c}
if (goat_count == 0)
    printf("You have no goats.\n");
else if (goat_count == 1)
    printf("You have a singular goat.\n");
else if (goat_count == 2)
    printf("You have a brace of goats.\n");
else
    printf("You have a bona fide plethora of goats!\n");
```

Với vài điểm khác biệt chủ chốt:

* `switch` thường nhảy tới đoạn code đúng nhanh hơn (dù spec không bảo
  đảm điều đó).
* `if`-`else` có thể làm các so sánh quan hệ như `<` và `>=`, cộng số
  dấu phẩy động và các kiểu khác, còn `switch` thì không.

Còn một thứ khá thú vị về switch đôi khi bạn sẽ thấy: _fall through_
(rơi xuyên qua).

[i[`break` statement]<]
Nhớ `break` làm ta nhảy ra khỏi switch chứ?

[i[Fall through]<]
Thế, chuyện gì xảy ra nếu ta _không_ `break`?

Hoá ra ta cứ tiếp tục chạy xuống `case` kế tiếp! Demo!

``` {.c}
switch (x) {
    case 1:
        printf("1\n");
        // Fall through!
    case 2:
        printf("2\n");
        break;
    case 3:
        printf("3\n");
        break;
}
```

Nếu `x == 1`, `switch` này trước hết trúng `case 1`, nó in `1`, nhưng
rồi cứ chạy tiếp xuống dòng code kế... in ra `2`!

Rồi, cuối cùng, ta gặp `break` nên nhảy ra khỏi `switch`.

nếu `x == 2`, thì ta chỉ trúng `case 2`, in `2`, và `break` như thường.

Không có `break` được gọi là _fall through_.

ProTip: _LUÔN LUÔN_ đặt comment trong code chỗ bạn chủ ý fall through,
như tôi đã làm phía trên. Nó sẽ cứu lập trình viên khác khỏi thắc mắc
bạn có cố ý làm thế không.
[i[Fall through]>]

Thực tế, đây là một trong những chỗ phổ biến phát sinh bug trong
chương trình C: quên đặt `break` trong `case`. Bạn phải đặt nó nếu
không muốn rơi tiếp xuống case kế^[Chuyện này được xem là nguy hiểm
đến mức những người thiết kế ngôn ngữ Go đã đặt `break` làm mặc định;
bạn phải dùng rõ ràng câu lệnh `fallthrough` của Go nếu muốn rơi sang
case kế.].
[i[`break` statement]>]

Ở trên tôi đã nói `switch` làm việc với kiểu số nguyên, cứ giữ như
thế. Đừng dùng số dấu phẩy động hay kiểu chuỗi trong đó. Một kẽ hở ở
đây là bạn có thể dùng kiểu ký tự vì ký tự bí mật là số nguyên. Nên
đoạn này hoàn toàn chấp nhận được:

``` {.c}
char c = 'b';

switch (c) {
    case 'a':
        printf("It's 'a'!\n");
        break;

    case 'b':
        printf("It's 'b'!\n");
        break;

    case 'c':
        printf("It's 'c'!\n");
        break;
}
```

Cuối cùng, bạn có thể dùng [i[`enum` keyword]]`enum` trong `switch` vì
chúng cũng là kiểu số nguyên. Nhưng để sau, ở chương `enum`.

[i[`switch` statement]>] [i[`break` statement]>] [i[Flow Control]>]

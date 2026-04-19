<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Chuỗi

[i[Strings]<]Cuối cùng! Chuỗi! Có gì đơn giản hơn được nữa?

Nhưng hoá ra chuỗi thật ra không phải chuỗi trong C. Đúng vậy đấy!
Chúng là con trỏ! Dĩ nhiên rồi!

Giống như mảng, chuỗi trong C _vừa đủ để tồn tại_.

Nhưng cứ xem nào, không phải chuyện ghê gớm lắm đâu.

## String Literal

[i[String literals]<]Trước khi bắt đầu, hãy nói về string literal
trong C. Đây là các chuỗi ký tự nằm trong dấu nháy _kép_ (`"`). (Nháy
đơn bọc quanh ký tự, và đó là con thú hoàn toàn khác.)

Ví dụ:

``` {.c}
"Hello, world!\n"
"This is a test."
"When asked if this string had quotes in it, she replied, \"It does.\""
```

Cái đầu tiên có newline ở cuối, khá phổ biến.

Cái cuối có dấu nháy kép nhúng bên trong, nhưng bạn thấy mỗi cái được
đặt trước (ta nói "được escape bởi") một dấu gạch chéo ngược (`\`) báo
cho biết một dấu nháy kép chữ thuộc về chuỗi tại điểm đó. Đây là cách
trình biên dịch C phân biệt giữa in dấu nháy kép và dấu nháy kép ở
cuối chuỗi.[i[String literals]>]

## Biến chuỗi

[i[String variables]<]Giờ ta đã biết cách làm một string literal, hãy
gán nó vào biến để làm gì đó với nó.

``` {.c}
char *s = "Hello, world!";
```

Để ý kiểu: pointer tới `char`. Biến chuỗi `s` thật ra là con trỏ tới
ký tự đầu tiên trong chuỗi đó, chính là `H`.

Và ta có thể in nó bằng format specifier `%s` (viết tắt cho "string"):

``` {.c}
char *s = "Hello, world!";

printf("%s\n", s);  // "Hello, world!"
```
[i[String variables]>]

## Biến chuỗi dưới dạng mảng

[i[String variables-->as arrays]<]
Một lựa chọn khác, gần như tương đương với cách dùng `char*` phía
trên:

``` {.c}
char s[14] = "Hello, world!";

// or, if we were properly lazy and have the compiler
// figure the length for us:

char s[] = "Hello, world!";
```

Nghĩa là bạn có thể dùng ký pháp mảng để truy cập các ký tự trong
chuỗi. Làm đúng thế để in tất cả ký tự trong chuỗi trên cùng một dòng:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    char s[] = "Hello, world!";

    for (int i = 0; i < 13; i++)
        printf("%c", s[i]);
    printf("\n");
}
```

Để ý ta dùng format specifier `%c` để in một ký tự đơn.

Và, xem cái này. Chương trình vẫn chạy tốt nếu ta đổi định nghĩa `s`
thành kiểu `char*`:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    char *s = "Hello, world!";   // char* here

    for (int i = 0; i < 13; i++)
        printf("%c", s[i]);    // But still use arrays here...?
    printf("\n");
}
```

Và ta vẫn có thể dùng ký pháp mảng để in ra! Bất ngờ nhỉ, nhưng chỉ vì
ta chưa nói về sự tương đương giữa mảng và con trỏ. Nhưng đây là thêm
một gợi ý nữa rằng sâu thẳm bên trong, mảng và con trỏ là cùng một
thứ.
[i[String variables-->as arrays]>]

## Khởi tạo chuỗi

[i[Strings-->initializers]<]
Ta đã thấy vài ví dụ khởi tạo biến chuỗi bằng string literal:

``` {.c}
char *s = "Hello, world!";
char t[] = "Hello, again!";
```

Nhưng hai cách khởi tạo này khác nhau một chút tinh tế. Một string
literal, tương tự một integer literal, có bộ nhớ được trình biên dịch
tự động quản lý giúp bạn! Với số nguyên, tức là một mẩu dữ liệu kích
thước cố định, trình biên dịch khá dễ quản lý. Nhưng chuỗi là con thú
số byte thay đổi, được trình biên dịch thuần hoá bằng cách quăng vào
một mẩu bộ nhớ và đưa cho bạn một con trỏ tới đó.

Cách viết này trỏ tới bất cứ nơi đâu chuỗi được đặt. Thường thì nơi
đó nằm trong vùng đất xa xôi so với phần còn lại của bộ nhớ chương
trình, bộ nhớ chỉ đọc, vì lý do liên quan đến hiệu năng và an toàn.

``` {.c}
char *s = "Hello, world!";
```

Nên nếu bạn thử đột biến chuỗi bằng:

``` {.c}
char *s = "Hello, world!";

s[0] = 'z';  // BAD NEWS: tried to mutate a string literal!
```

Hành vi là không xác định. Có lẽ, tuỳ hệ thống, sẽ dẫn đến crash.

Nhưng khai báo nó dưới dạng mảng thì khác. Trình biên dịch không cất
các byte đó ở phần khác của thành phố, chúng ở ngay cuối đường. Cái
này là một _bản sao_ có thể đột biến của chuỗi, cái ta có thể đổi tuỳ
thích:

``` {.c}
char t[] = "Hello, again!";  // t is an array copy of the string 
t[0] = 'z'; //  No problem

printf("%s\n", t);  // "zello, again!"
```

Nên nhớ nhé: nếu bạn có con trỏ tới một string literal, đừng cố đổi
nó! Và nếu bạn dùng chuỗi trong dấu nháy kép để khởi tạo mảng, đó
thật ra không phải là string literal.
[i[Strings-->initializers]>]

## Lấy chiều dài chuỗi

[i[Strings-->getting the length]<]
Bạn không thể, vì C không theo dõi hộ bạn. Và khi tôi nói "không thể",
tôi thật ra muốn nói "có thể"^[Dù đúng là C không theo dõi chiều dài
chuỗi.]. Có một hàm trong `<string.h>` tên là `strlen()` có thể được
dùng để tính chiều dài của bất kỳ chuỗi nào tính bằng byte^[Nếu bạn
dùng bộ ký tự cơ bản hoặc một bộ ký tự 8 bit, bạn quen với chuyện một
ký tự là một byte. Nhưng điều này không đúng với mọi bộ mã ký tự.].

``` {.c .numberLines}
#include <stdio.h>
#include <string.h>

int main(void)
{
    char *s = "Hello, world!";

    printf("The string is %zu bytes long.\n", strlen(s));
}
```

Hàm `strlen()` trả về kiểu `size_t`, là một kiểu số nguyên nên bạn có
thể dùng nó cho phép toán số nguyên. Ta in `size_t` bằng `%zu`.

Chương trình trên in:

``` {.default}
The string is 13 bytes long.
```

Tuyệt! Vậy là _có thể_ lấy chiều dài chuỗi!
[i[Strings-->getting the length]>]

Nhưng... nếu C không theo dõi chiều dài chuỗi ở đâu, làm sao nó biết
chuỗi dài bao nhiêu?

## Kết thúc chuỗi

[i[Strings-->termination]<]
C xử lý chuỗi hơi khác nhiều ngôn ngữ lập trình, và thật ra khác với
gần như mọi ngôn ngữ lập trình hiện đại.

Khi bạn làm một ngôn ngữ mới, về cơ bản bạn có hai lựa chọn để lưu
chuỗi trong bộ nhớ:

1. Lưu các byte của chuỗi cùng với một con số chỉ chiều dài chuỗi.

2. Lưu các byte của chuỗi, và đánh dấu điểm kết thúc chuỗi bằng một
   byte đặc biệt gọi là _terminator_ (kết thúc).

Nếu bạn muốn chuỗi dài hơn 255 ký tự, phương án 1 cần ít nhất hai byte
để lưu chiều dài. Còn phương án 2 chỉ cần một byte để kết thúc chuỗi.
Vậy là tiết kiệm được chút ít.

Dĩ nhiên, ngày nay nghe có vẻ lố bịch khi lo tiết kiệm một byte (hay
3, nhiều ngôn ngữ vui vẻ cho phép bạn có chuỗi dài 4 gigabyte). Nhưng
ngày xưa, đó là chuyện lớn hơn.

Nên C chọn phương án #2. Trong C, một "chuỗi" được định nghĩa bởi hai
đặc tính cơ bản:

* Một con trỏ tới ký tự đầu tiên trong chuỗi.
* Một byte có giá trị bằng không (hay ký tự `NUL`^[Cái này khác với
  con trỏ `NULL`, và tôi sẽ viết tắt nó thành `NUL` khi nói về ký tự
  so với `NULL` cho con trỏ.]) nằm đâu đó trong bộ nhớ sau con trỏ,
  báo hiệu kết thúc chuỗi.

Ký tự `NUL` có thể được viết trong code C là `\0`, dù bạn không hay
phải làm thế.

Khi bạn đặt chuỗi trong dấu nháy kép trong code, ký tự `NUL` được tự
động, ngầm định, đưa vào.

``` {.c}
char *s = "Hello!";  // Actually "Hello!\0" behind the scenes
```

Với chuyện này trong đầu, hãy viết hàm `strlen()` của riêng ta, đếm
các `char` trong chuỗi cho tới khi gặp `NUL`.

Quy trình là quét dọc chuỗi tìm một ký tự `NUL` duy nhất, đếm khi đi
qua^[Sau này ta sẽ học cách làm gọn hơn với pointer arithmetic.]:

``` {.c}
int my_strlen(char *s)
{
    int count = 0;

    while (s[count] != '\0')  // Single quotes for single char
        count++;

    return count;
}
```

Và đó về cơ bản là cách `strlen()` có sẵn làm chuyện này.
[i[Strings-->termination]>]

## Sao chép một chuỗi

[i[Strings-->copying]<]
Bạn không thể sao chép chuỗi qua toán tử gán (`=`). Tất cả những gì nó
làm là tạo một bản sao của con trỏ tới ký tự đầu tiên... nên bạn kết
thúc với hai con trỏ cùng trỏ tới một chuỗi:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    char s[] = "Hello, world!";
    char *t;

    // This makes a copy of the pointer, not a copy of the string!
    t = s;

    // We modify t
    t[0] = 'z';

    // But printing s shows the modification!
    // Because t and s point to the same string!

    printf("%s\n", s);  // "zello, world!"
}
```

Nếu bạn muốn tạo một bản sao của chuỗi, bạn phải chép từng byte, nghĩa
là bạn sẽ lấy từng byte của chuỗi từ một chỗ trong bộ nhớ và nhân đôi
chúng ở chỗ khác trong bộ nhớ. Chuyện này được làm dễ hơn nhờ hàm
`strcpy()`.[^440b]

[^440b]: Có một hàm khác tên `strncpy()` hạn chế số byte được sao
    chép. Có người nói bạn nên luôn dùng `strncpy()` vì bảo vệ chống
    tràn bộ đệm. Người khác nói bạn không bao giờ nên dùng
    `strncpy()` vì nó không nhất thiết kết thúc chuỗi của bạn, một
    cái bẫy tự bắn chân cực kỳ kinh dị khác. Nếu bạn thật sự muốn an
    toàn, có thể viết phiên bản `strncpy()` của riêng mình luôn luôn
    kết thúc chuỗi.

Trước khi sao chép chuỗi, đảm bảo bạn có chỗ để chép vào, tức mảng
đích sẽ chứa các ký tự phải dài ít nhất bằng chuỗi bạn đang chép.

``` {.c .numberLines}
#include <stdio.h>
#include <string.h>

int main(void)
{
    char s[] = "Hello, world!";
    char t[100];  // Each char is one byte, so plenty of room

    // This makes a copy of the string!
    strcpy(t, s);

    // We modify t
    t[0] = 'z';

    // And s remains unaffected because it's a different string
    printf("%s\n", s);  // "Hello, world!"

    // But t has been changed
    printf("%s\n", t);  // "zello, world!"
}
```

Để ý với `strcpy()`, con trỏ đích là đối số đầu tiên, con trỏ nguồn là
đối số thứ hai. Một mẹo nhớ tôi dùng là đó là thứ tự mà bạn sẽ đặt
`t` và `s` nếu phép gán `=` chạy được cho chuỗi, với nguồn ở bên phải
và đích ở bên trái.
[i[Strings-->copying]>][i[Strings]>]

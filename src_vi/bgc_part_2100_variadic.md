<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Hàm variadic

[i[Variadic functions]<]

_Variadic_ là từ kêu kêu để chỉ hàm nhận số đối số tuỳ ý.

Hàm thường nhận một số đối số cụ thể, ví dụ:

``` {.c}
int add(int x, int y)
{
    return x + y;
}
```

Bạn chỉ có thể gọi nó với đúng hai đối số tương ứng tham số `x` và
`y`.

``` {.c}
add(2, 3);
add(5, 12);
```

Nhưng nếu thử nhiều hơn, compiler không cho:

``` {.c}
add(2, 3, 4);  // ERROR
add(5);        // ERROR
```

Hàm variadic vượt qua giới hạn này ở một mức nào đó.

Ta đã thấy một ví dụ nổi tiếng trong `printf()`! Bạn có thể truyền
đủ kiểu thứ vào nó.

``` {.c}
printf("Hello, world!\n");
printf("The number is %d\n", 2);
printf("The number is %d and pi is %f\n", 2, 3.14159);
```

Nó có vẻ chẳng quan tâm bạn đưa bao nhiêu đối số!

Ừ, không hẳn. Không đối số nào sẽ cho lỗi:

``` {.c}
printf();  // ERROR
```

Điều này dẫn ta tới một giới hạn của hàm variadic trong C: chúng
phải có ít nhất một đối số.

Nhưng ngoài chuyện đó, chúng khá linh hoạt, thậm chí cho phép đối số
có kiểu khác nhau như `printf()` làm.

Xem chúng hoạt động sao nhé!

## Dấu ba chấm trong signature hàm

Vậy nó chạy thế nào, về cú pháp?

[i[`...` variadic arguments]<]

Việc bạn làm là đặt mọi đối số _bắt buộc_ phải truyền vào trước (và
nhớ phải có ít nhất một) và sau đó, bạn đặt `...`. Như vầy:

``` {.c}
void func(int a, ...)   // Literally 3 dots here
```

Đây là ít code để demo:

``` {.c}
#include <stdio.h>

void func(int a, ...)
{
    printf("a is %d\n", a);  // Prints "a is 2"
}

int main(void)
{
    func(2, 3, 4, 5, 6);
}
```

[i[`...` variadic arguments]>]

Rồi, hay, ta lấy được đối số đầu ở biến `a`, nhưng còn phần đối số
còn lại thì sao? Làm sao tới được chúng?

Đây là chỗ vui bắt đầu!

## Lấy các đối số dư

Bạn sẽ muốn include [i[`stdarg.h` header file]] `<stdarg.h>` để làm
mấy chuyện này.

[i[`va_list` type]<]

Trước tiên, ta sẽ dùng một biến đặc biệt kiểu `va_list` (variable
argument list) để theo dõi ta đang truy cập biến nào tại thời điểm
đó.

[i[`va_start()` macro]<]
[i[`va_arg()` macro]<]
[i[`va_end()` macro]<]

Ý tưởng là ta bắt đầu xử lý đối số bằng một lời gọi `va_start()`, xử
lý từng đối số một bằng `va_arg()`, rồi, khi xong, kết bằng
`va_end()`.

Khi bạn gọi `va_start()`, bạn cần truyền _tham số có tên cuối cùng_
(cái ngay trước `...`) để nó biết chỗ cần bắt đầu tìm các đối số
thêm.

Và khi bạn gọi `va_arg()` để lấy đối số kế, bạn phải cho nó biết
kiểu của đối số kế tiếp cần lấy.

Đây là demo cộng lại một số tuỳ ý các số nguyên. Đối số đầu là số
lượng số nguyên cần cộng. Ta sẽ dùng nó để biết phải gọi `va_arg()`
bao nhiêu lần.

``` {.c .numberLines}
#include <stdio.h>
#include <stdarg.h>

int add(int count, ...)
{
    int total = 0;
    va_list va;

    va_start(va, count);   // Start with arguments after "count"

    for (int i = 0; i < count; i++) {
        int n = va_arg(va, int);   // Get the next int

        total += n;
    }

    va_end(va);  // All done

    return total;
}

int main(void)
{
    printf("%d\n", add(4, 6, 2, -4, 17));  // 6 + 2 - 4 + 17 = 21
    printf("%d\n", add(2, 22, 44));        // 22 + 44 = 66
}
```

[i[`va_start()` macro]>]
[i[`va_end()` macro]>]

(Lưu ý khi gọi `printf()`, nó dùng số `%d` (hay bất cứ thứ gì) trong
chuỗi format để biết còn bao nhiêu đối số nữa!)

Nếu cú pháp của `va_arg()` trông lạ với bạn (vì có tên kiểu lơ lửng
trong đó), bạn không đơn độc. Chúng được cài đặt bằng macro
preprocessor để có được mọi phép màu cần thiết.

## Chức năng `va_list`

Cái biến `va_list` ta đang dùng ở trên là gì? Đó là biến mờ^[Nghĩa
là đám dev thấp cổ bé họng như ta không phải biết trong đó có gì hay
ý nghĩa gì. Spec không ra lệnh chi tiết nó là gì.] giữ thông tin về
việc ta sẽ lấy đối số nào kế tiếp bằng `va_arg()`. Thấy cách ta gọi
`va_arg()` lặp đi lặp lại đấy? Biến `va_list` là chỗ giữ chỗ đang
theo dõi tiến độ cho tới giờ.

[i[`va_start()` macro]<]

Nhưng ta phải khởi tạo biến đó bằng một giá trị hợp lý. Đó là lúc
`va_start()` ra sân.

Khi ta gọi `va_start(va, count)` ở trên, ta đang nói, "Khởi tạo biến
`va` để trỏ tới đối số biến _ngay sau_ `count`."

[i[`va_end()` macro]<]

Và đó là _lý do_ ta cần có ít nhất một biến có tên trong danh sách
đối số^[Thành thật mà nói, loại bỏ giới hạn này khỏi ngôn ngữ là khả
thi, nhưng ý tưởng là các macro `va_start()`, `va_arg()` và
`va_end()` có thể viết được bằng C. Và để làm được điều đó, ta cần
cách nào đó khởi tạo một pointer tới vị trí của tham số đầu. Và để
làm điều đó, ta cần _tên_ của tham số đầu. Sẽ cần mở rộng ngôn ngữ
để làm được việc này, và tới giờ ủy ban chưa tìm ra lý do chính
đáng.].

Một khi có pointer tới tham số ban đầu, bạn có thể dễ dàng lấy các
giá trị đối số sau bằng cách gọi `va_arg()` lặp đi lặp lại. Khi làm
vậy, bạn phải truyền vào biến `va_list` của mình (để nó tiếp tục
theo dõi bạn đang ở đâu), cùng với kiểu của đối số bạn sắp copy ra.

Tùy bạn, người lập trình, nghĩ ra kiểu bạn sẽ truyền cho `va_arg()`.
Trong ví dụ ở trên, ta chỉ làm `int`. Nhưng trong trường hợp
`printf()`, nó dùng format specifier để xác định kiểu nào cần lấy
kế tiếp.

Và khi xong, gọi `va_end()` để kết lại. Bạn **phải** (spec nói)
gọi cái này trên một biến `va_list` cụ thể trước khi bạn quyết định
gọi lại `va_start()` hay `va_copy()` trên nó lần nữa. Tôi biết ta
chưa nói về `va_copy()`.

Vậy tiến trình chuẩn là:

* `va_start()` để khởi tạo biến `va_list` của bạn
* Lặp lại `va_arg()` để lấy giá trị
* `va_end()` để kết biến `va_list` của bạn

[i[`va_start()` macro]>]
[i[`va_end()` macro]>]
[i[`va_arg()` macro]>]

[i[`va_copy()` macro]<]

Tôi cũng có nhắc `va_copy()` ở trên; nó làm bản sao biến `va_list`
của bạn ở đúng cùng trạng thái. Tức là, nếu bạn chưa bắt đầu dùng
`va_arg()` với biến nguồn, biến mới cũng chưa bắt đầu. Nếu bạn đã
ngốn 5 biến bằng `va_arg()` cho tới giờ, bản sao cũng phản ánh y
vậy.

`va_copy()` có thể hữu ích nếu bạn cần quét trước qua đối số nhưng
vẫn cần nhớ vị trí hiện tại.

[i[`va_copy()` macro]>]

## Hàm thư viện dùng `va_list`

[i[`va_list` type-->passing to functions]<]

Một trong những cách dùng khác của mấy cái này khá hay: viết biến
thể `printf()` tuỳ ý của riêng bạn. Sẽ đau đầu nếu phải xử mọi format
specifier đó phải không? Cả tỷ cái?

May thay, có các biến thể `printf()` nhận một `va_list` đang hoạt
động làm đối số. Bạn có thể dùng chúng để bọc lại và tự làm
`printf()` riêng!

[i[`vprintf()` function]<]

Các hàm này bắt đầu bằng chữ `v`, như `vprintf()`, `vfprintf()`,
`vsprintf()` và `vsnprintf()`. Về cơ bản là mọi bản hit kinh điển
của `printf()` chỉ thêm `v` đằng trước.

Hãy làm hàm `my_printf()` chạy y `printf()` chỉ khác là nhận thêm
một đối số đầu.

``` {.c .numberLines}
#include <stdio.h>
#include <stdarg.h>

int my_printf(int serial, const char *format, ...)
{
    va_list va;
    int rv;

    // Do my custom work
    printf("The serial number is: %d\n", serial);

    // Then pass the rest off to vprintf()
    va_start(va, format);
    rv = vprintf(format, va);
    va_end(va);

    return rv;
}

int main(void)
{
    int x = 10;
    float y = 3.2;

    my_printf(3490, "x is %d, y is %f\n", x, y);
}
```

Thấy ta làm gì đó chưa? Ở dòng 12-14 ta mở một biến `va_list` mới,
rồi cứ thế truyền thẳng vào `vprintf()`. Và nó biết ngay phải làm gì
với nó, vì nó có sẵn mọi đầu óc của `printf()` gài vào.

[i[`vprintf()` function]>]

Tuy vậy, ta vẫn phải gọi `va_end()` khi xong, nên đừng quên!

[i[`va_list` type-->passing to functions]>]
[i[`va_list` type]>]

## Bẫy macro variadic

Như tôi đã nhắc, `va_start()` và `va_end()` có thể là macro. Một hệ
quả của chuyện này có thể là chúng có tiềm năng mở ra một scope cục
bộ mới. (Tức là, nếu `va_start()` có `{` và `va_end()` chứa `}`.)

[i[`va_start()` macro-->scoping issues]>]

Nên ta cần cảnh giác với chuyện scope có thể gặp vấn đề. Lấy ví dụ
sau:

``` {.c}
va_start(va, format);          // may contain {
int rv = vprintf(format, va);
va_end(va);                    // may contain }

return rv;
```

Nếu `va_start()` mở scope mới, `rv` sẽ cục bộ trong scope đó rồi câu
`return` sẽ fail. Nhưng chuyện này sẽ âm thầm chỉ xảy ra trên các
compiler tình cờ làm vậy với macro `va`.

[i[Variadic functions]>]

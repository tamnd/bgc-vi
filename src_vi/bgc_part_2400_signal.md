<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Xử lý signal

[i[Signal handling]<]

Trước khi bắt đầu, tôi muốn khuyên bạn nên bỏ qua cả chương này và
dùng các hàm xử lý signal (rất có thể) ngon hơn của OS. Các hệ
Unix-like có họ hàm [i[`sigaction()` function]] `sigaction()`, còn
Windows thì có... thứ gì đó của nó^[Hình như Windows không làm signal
kiểu Unix ở tầng sâu, và chúng được giả lập cho các app console.].

Dẹp chuyện đó sang bên, vậy signal là gì?

## Signal là gì?

Một _signal_ được _raise_ khi có đủ kiểu sự kiện bên ngoài xảy ra.
Chương trình bạn có thể được cấu hình để bị ngắt nhằm _handle_
signal, và tuỳ chọn, chạy tiếp chỗ bị bỏ dở sau khi đã xử lý xong.

Nghĩ nó như một hàm được gọi tự động khi một trong các sự kiện ngoài
này xảy ra.

Các sự kiện này là gì? Trên hệ của bạn, có lẽ có kha khá, nhưng
trong spec C chỉ có vài cái:

[i[`SIGABRT` signal]]
[i[`SIGFPE` signal]]
[i[`SIGILL` signal]]
[i[`SIGINT` signal]]
[i[`SIGSEGV` signal]]
[i[`SIGTERM` signal]]

|Signal|Mô tả|
|-------|--------------------------------------------------------|
|`SIGABRT`|Kết thúc bất thường, thứ xảy ra khi `abort()` được gọi.|
|`SIGFPE`|Ngoại lệ dấu chấm động.|
|`SIGILL`|Lệnh không hợp lệ.|
|`SIGINT`|Ngắt, thường là kết quả của việc bấm `CTRL-C`.|
|`SIGSEGV`|"Segmentation Violation": truy cập bộ nhớ không hợp lệ.|
|`SIGTERM`|Yêu cầu kết thúc.|

Bạn có thể cài chương trình để bỏ qua, xử lý, hoặc cho chạy hành vi
mặc định đối với từng signal bằng hàm `signal()`.

## Xử lý signal với `signal()`

[i[`signal()` function]<]

Lời gọi `signal()` nhận hai tham số: signal cần quan tâm, và hành
động cần làm khi signal đó được raise.

Hành động có thể là một trong ba thứ:

* Một con trỏ tới hàm xử lý (handler).
* [i[`SIG_IGN` macro]<]`SIG_IGN` để bỏ qua signal.
* [i[`SIG_DFL` macro]]`SIG_DFL` để khôi phục handler mặc định cho signal.

Viết một chương trình mà bạn không `CTRL-C` ra nổi. (Đừng lo, trong
chương trình sau, bạn cũng có thể bấm `RETURN` để thoát.)

[i[`SIGINT` signal]<]

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

int main(void)
{
    char s[1024];

    signal(SIGINT, SIG_IGN);    // Ignore SIGINT, caused by ^C

    printf("Try hitting ^C... (hit RETURN to exit)\n");

    // Wait for a line of input so the program doesn't just exit
    fgets(s, sizeof s, stdin);
}
```

Để ý dòng 8, ta bảo chương trình bỏ qua `SIGINT`, signal ngắt được
raise khi `CTRL-C` được bấm. Bạn bấm bao nhiêu tuỳ thích, signal vẫn
bị ngó lơ. Nếu bạn comment dòng 8 đi, bạn sẽ thấy có thể `CTRL-C`
thoải mái và thoát chương trình tại chỗ.

[i[`SIGINT` signal]>]
[i[`SIG_IGN` macro]>]

## Viết signal handler

Tôi có nói rằng bạn cũng có thể viết một hàm handler được gọi khi
signal được raise.

Mấy cái này khá đơn giản, nhưng cũng rất bị giới hạn về năng lực khi
dính tới spec.

[i[`signal()` function]<]

Trước khi bắt đầu, xem prototype của `signal()`:

``` {.c}
void (*signal(int sig, void (*func)(int)))(int);
```

Dễ đọc chưa?

_SAI!_ `:)`

Dành chút để tháo nó ra cho quen tay.

`signal()` nhận hai đối số: một số nguyên `sig` đại diện cho signal,
và một con trỏ `func` tới handler (handler trả về `void` và nhận một
`int` làm đối số), tô đậm phía dưới:

``` {.c}
                sig          func
              |-----|  |---------------|
void (*signal(int sig, void (*func)(int)))(int);
```

[i[`signal()` function]>]

Về cơ bản, ta sẽ truyền vào số signal cần bắt, và truyền một con trỏ
tới hàm có dạng:

``` {.c}
void f(int x);
```

hàm đó sẽ làm phần bắt signal thực sự.

Giờ, còn phần còn lại của prototype thì sao? Về cơ bản đó là toàn bộ
kiểu trả về. Thấy không, `signal()` sẽ trả về bất cứ thứ gì bạn
truyền làm `func` khi thành công... tức là nó đang trả về một con
trỏ tới hàm trả về `void` và nhận `int` làm đối số.

``` {.c}
returned
function    indicates we're              and
returns     returning a                  that function
void        pointer to function          takes an int
|--|        |                                   |---|
void       (*signal(int sig, void (*func)(int)))(int);
```

Ngoài ra, nó có thể trả về [i[`SIG_ERR` macro]] `SIG_ERR` khi có
lỗi.

Làm một ví dụ bạn phải bấm `CTRL-C` hai lần mới thoát.

Tôi muốn nói rõ rằng chương trình này dính hành vi không xác định
(undefined behavior) ở vài chỗ. Nhưng nó chắc sẽ chạy với bạn, và
khó nghĩ ra demo di động mà không trivial.

[i[`signal()` function]<]
[i[`SIG_INT` signal]<]

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

int count = 0;

void sigint_handler(int signum)
{
    // The compiler is allowed to run:
    //
    //   signal(signum, SIG_DFL)
    //
    // when the handler is called. So we reset the handler here:
    signal(SIGINT, sigint_handler);

    (void)signum;   // Get rid of unused variable warning

    count++;                       // Undefined behavior
    printf("Count: %d\n", count);  // Undefined behavior

    if (count == 2) {
        printf("Exiting!\n");      // Undefined behavior
        exit(0);
    }
}

int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("Try hitting ^C...\n");

    for(;;);  // Wait here forever
}
```

[i[`SIG_INT` signal]>]

Một điều bạn sẽ để ý là ở dòng 14 ta reset signal handler. Đó là vì
C có quyền reset signal handler về hành vi [i[`SIG_DFL` macro]]
`SIG_DFL` trước khi chạy handler tuỳ chỉnh của bạn. Nói cách khác,
nó có thể chỉ chạy một lần. Nên ta reset ngay lập tức để bắt được
lần kế tiếp.

Ta bỏ qua giá trị trả về của `signal()` trong trường hợp này. Nếu ta
đã set một handler khác trước đó, nó sẽ trả về con trỏ tới handler
đó, mà ta có thể lấy kiểu này:

``` {.c}
// old_handler is type "pointer to function that takes a single
// int parameter and returns void":

void (*old_handler)(int);

old_handler = signal(SIGINT, sigint_handler);
```

[i[`signal()` function]>]

Nói thật tôi không rõ use case phổ biến cho chuyện này. Nhưng nếu
bạn cần handler cũ vì lý do nào đó, bạn có thể lấy theo cách đó.

Ghi chú nhanh về dòng 16, đó chỉ là cách báo compiler đừng warning
rằng ta không dùng biến này. Giống như nói, "Tôi biết tôi không dùng
nó, ông không cần cảnh báo tôi đâu."

Và cuối cùng bạn sẽ thấy tôi đã đánh dấu hành vi không xác định ở
vài chỗ. Xem thêm ở phần kế tiếp.

## Ta thực sự làm được gì?

Hoá ra ta khá bị giới hạn về những gì có thể và không thể làm trong
signal handler. Đây là một trong những lý do tôi bảo bạn đừng thèm
dính vào cái này và dùng signal handling của OS thay thế (ví dụ
[i[`sigaction()` function]] `sigaction()` cho các hệ Unix-like).

Wikipedia nói thẳng rằng thứ duy nhất thực sự di động bạn làm được
là gọi `signal()` với `SIG_IGN` hay `SIG_DFL`, thế thôi.

Đây là những gì ta **không thể** làm một cách di động:

[i[Signal handling--->limitations]<]

* Gọi bất cứ hàm thư viện chuẩn nào.
  * Như `printf()` chẳng hạn.
  * Tôi nghĩ gọi các hàm có thể restart/reentrant là tương đối an
    toàn, nhưng spec không cho phép cái đặc quyền đó.
* Lấy hay set giá trị từ một biến `static` cục bộ, scope file, hay
  thread-local.
  * Trừ khi nó là lock-free atomic object hoặc...
  * Bạn đang gán vào biến kiểu `volatile sig_atomic_t`.

[i[`sig_atomic_t` type]<]

Cái cuối đó, `sig_atomic_t`, là tấm vé để bạn đưa dữ liệu ra khỏi
signal handler. (Trừ khi bạn muốn dùng lock-free atomic object, vốn
nằm ngoài phạm vi phần này^[Gây bối rối là `sig_atomic_t` có trước
lock-free atomic và không phải cùng một thứ.].) Nó là kiểu số
nguyên, có thể có dấu hoặc không. Và nó bị giới hạn bởi thứ bạn có
thể nhét vào.

Bạn có thể xem giá trị min và max cho phép trong macro
`SIG_ATOMIC_MIN` và `SIG_ATOMIC_MAX`^[Nếu `sig_action_t` có dấu,
range sẽ ít nhất là `-127` tới `127`. Nếu không dấu, ít nhất `0`
tới `255`.].

Gây bối rối là spec cũng nói bạn không được "refer tới bất kỳ object
nào có static hay thread storage duration mà không phải lock-free
atomic object ngoại trừ bằng cách gán giá trị vào một object được
khai báo là `volatile sig_atomic_t` [...]"

Tôi hiểu ý này là bạn không thể đọc hay ghi bất cứ gì không phải
lock-free atomic object. Ngoài ra bạn có thể gán vào một object
`volatile sig_atomic_t`.

Nhưng bạn đọc từ nó được không? Thật lòng tôi không thấy lý do gì
không được, trừ việc spec rất chăm chỉ nhắc chuyện "gán vào". Nhưng
nếu bạn phải đọc nó và ra quyết định dựa trên đó, bạn có thể mở ra
chỗ cho race condition nào đó.

[i[Signal handling--->limitations]>]

Có cái đó trong đầu, ta có thể viết lại đoạn "bấm `CTRL-C` hai lần
để thoát" sao cho di động hơn chút, tuy output có kiệm lời hơn.

Đổi handler `SIGINT` của ta để không làm gì ngoại trừ tăng một giá
trị kiểu `volatile sig_atomic_t`. Nó sẽ đếm số lần `CTRL-C` đã được
bấm.

Rồi trong vòng lặp main, ta sẽ kiểm tra xem counter đó đã vượt quá
`2` chưa, và bail ra nếu có.

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

volatile sig_atomic_t count = 0;

void sigint_handler(int signum)
{
    (void)signum;                    // Unused variable warning

    signal(SIGINT, sigint_handler);  // Reset signal handler

    count++;                         // Undefined behavior
}

int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("Hit ^C twice to exit.\n");

    while(count < 2);
}
```

[i[`sig_atomic_t` type]>]

Lại hành vi không xác định? Tôi đọc đây là có, vì ta phải đọc giá
trị để tăng rồi lưu lại. Một thread khác có thể nghịch `count` và
làm ta phát cáu. Nhưng trong ví dụ đơn giản này, không có thread
khác làm chuyện đó, nên ta bỏ qua được và tận hưởng demo.

Nếu ta chỉ muốn trì hoãn thoát thêm một lần bấm `CTRL-C`, ta làm
được mà không khổ lắm. Nhưng thêm nữa thì cần mấy chuỗi hàm nhố
nhăng.

Cái ta sẽ làm là xử lý một lần, và handler sẽ reset signal về hành
vi mặc định (tức là thoát):

[i[`SIG_DFL` macro]<]

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

void sigint_handler(int signum)
{
    (void)signum;                      // Unused variable warning
    signal(SIGINT, SIG_DFL);           // Reset signal handler
}

int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("Hit ^C twice to exit.\n");

    while(1);
}
```

[i[`SIG_DFL` macro]>]

Sau này khi nhìn vào biến lock-free atomic, ta sẽ thấy cách sửa
phiên bản dùng `count` (giả sử biến lock-free atomic có sẵn trên hệ
cụ thể của bạn).

Đó là lý do ngay từ đầu tôi đã gợi ý bạn check signal system tích
hợp sẵn của OS như phương án nhiều khả năng ngon hơn.

## Bạn Hiền Không Để Bạn Hiền `signal()`

Lần nữa, dùng signal handling tích hợp sẵn của OS hay cái tương
đương. Nó không có trong spec, không di động bằng, nhưng có lẽ mạnh
hơn nhiều. Cộng thêm OS của bạn có lẽ định nghĩa một số signal không
có trong spec C. Và viết code di động dùng `signal()` dù sao cũng
khó.

[i[`signal()` function]>]
[i[Signal handling]>]

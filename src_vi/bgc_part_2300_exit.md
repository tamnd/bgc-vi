<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Thoát khỏi chương trình

[i[Exiting]<]

Hóa ra có khá nhiều cách để làm chuyện này, và còn cả cách cài "móc"
để hàm nào đó chạy khi chương trình thoát.

Trong chương này ta sẽ đào vào và xem chúng.

Ta đã nói về ý nghĩa của mã exit status trong phần [Exit
Status](#exit-status), nên nhảy ngược lại đó và xem lại nếu cần.

Mọi hàm trong phần này nằm ở `<stdlib.h>`.

## Thoát bình thường

Bắt đầu với các cách thoát thường, rồi nhảy sang vài cái hiếm và
quái hơn.

Khi bạn thoát chương trình bình thường, mọi luồng I/O mở được flush
và file tạm được xóa. Về cơ bản đây là lối thoát đẹp nơi mọi thứ
được dọn dẹp và xử lý. Đây là thứ bạn muốn làm gần như mọi lúc, trừ
khi có lý do khác.

### Trở về từ `main()`

[i[Exiting-->return from `main()`]<]

Nếu bạn để ý, `main()` có kiểu trả về là `int`... nhưng tôi hiếm
khi, nếu có, `return` bất cứ gì từ `main()`.

Đó là vì chỉ riêng `main()` (và tôi không thể nhấn mạnh đủ rằng
trường hợp đặc biệt này _chỉ_ áp dụng cho `main()` chứ không hàm nào
khác ở đâu) có _ngầm_ `return 0` nếu bạn rơi khỏi đuôi hàm.

Bạn có thể `return` từ `main()` tường minh bất cứ lúc nào bạn muốn,
và vài lập trình viên cảm thấy nó _Đúng_ hơn khi luôn có `return` ở
cuối `main()`. Nhưng nếu bạn bỏ đó, C sẽ đặt một cái đó vào giúp
bạn.

Vậy... đây là luật `return` cho `main()`:

* Bạn có thể trả về exit status từ `main()` bằng câu lệnh `return`.
  `main()` là hàm duy nhất có hành vi đặc biệt này. Dùng `return`
  trong bất kỳ hàm nào khác chỉ trả về từ hàm đó tới nơi gọi.
* Nếu bạn không `return` tường minh mà chỉ rơi khỏi đuôi của
  `main()`, y như bạn đã `return 0` hay `EXIT_SUCCESS`.

[i[Exiting-->return from `main()`]>]

### `exit()`

[i[Exiting-->return from `main()`]>]

Cái này cũng đã xuất hiện vài lần. Nếu bạn gọi `exit()` từ bất cứ
đâu trong chương trình, nó sẽ thoát tại điểm đó.

Đối số bạn truyền cho `exit()` là exit status.

### Cài exit handler với `atexit()`

[i[`atexit()` function]<]

Bạn có thể đăng ký các hàm được gọi khi chương trình thoát, dù bằng
cách return từ `main()` hay gọi hàm `exit()`.

Một lời gọi `atexit()` với tên hàm handler sẽ xong việc. Bạn có thể
đăng ký nhiều exit handler, và chúng sẽ được gọi theo thứ tự ngược
lại với thứ tự đăng ký.

Đây là ví dụ:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

void on_exit_1(void)
{
    printf("Exit handler 1 called!\n");
}

void on_exit_2(void)
{
    printf("Exit handler 2 called!\n");
}

int main(void)
{
    atexit(on_exit_1);
    atexit(on_exit_2);
    
    printf("About to exit...\n");
}
```

Và output là:

``` {.default}
About to exit...
Exit handler 2 called!
Exit handler 1 called!
```

[i[`atexit()` function]>]

## Thoát nhanh hơn với `quick_exit()`

[i[`quick_exit()` function]<]

Cái này tương tự thoát thường, trừ:

* File mở có thể không được flush.
* File tạm có thể không được xóa.
* Handler `atexit()` sẽ không được gọi.

Nhưng có cách để đăng ký exit handler: gọi `at_quick_exit()` tương
tự cách bạn gọi `atexit()`.

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

void on_quick_exit_1(void)
{
    printf("Quick exit handler 1 called!\n");
}

void on_quick_exit_2(void)
{
    printf("Quick exit handler 2 called!\n");
}

void on_exit(void)
{
    printf("Normal exit--I won't be called!\n");
}

int main(void)
{
    at_quick_exit(on_quick_exit_1);
    at_quick_exit(on_quick_exit_2);

    atexit(on_exit);  // This won't be called

    printf("About to quick exit...\n");

    quick_exit(0);
}
```

Cho ra output:

``` {.default}
About to quick exit...
Quick exit handler 2 called!
Quick exit handler 1 called!
```

Nó chạy y như `exit()`/`atexit()`, trừ việc flush file và dọn dẹp có
thể không được làm.

[i[`quick_exit()` function]>]

## Bắn nó từ quỹ đạo: `_Exit()`

[i[`_Exit()` function]<]

Gọi `_Exit()` thoát ngay lập tức, hết chuyện. Không có hàm callback
on-exit nào được thực thi. File sẽ không được flush. File tạm sẽ
không được xóa.

Dùng cái này nếu bạn phải thoát _ngay tức khắc_.

## Thoát đôi khi: `assert()`

Câu lệnh `assert()` được dùng để ép một điều gì đó phải đúng, không
thì chương trình sẽ thoát.

Dev thường dùng assert để bắt lỗi kiểu Never-Should-Happen (không
bao giờ nên xảy ra).

``` {.c}
#define PI 3.14159

assert(PI > 3);   // Sure enough, it is, so carry on
```

so với:

``` {.c}
goats -= 100;

assert(goats >= 0);  // Can't have negative goats
```

Trong trường hợp đó, nếu tôi cố chạy nó và `goats` tụt dưới `0`,
chuyện này xảy ra:

``` {.default}
goat_counter: goat_counter.c:8: main: Assertion `goats >= 0' failed.
Aborted
```

và tôi bị đá về dòng lệnh.

Cái này không thân thiện lắm với người dùng, nên nó chỉ được dùng
cho mấy thứ mà người dùng sẽ không bao giờ thấy. Và thường người ta
[tự viết macro assert riêng có thể tắt dễ hơn](#my-assert).

[i[`_Exit()` function]>]

## Thoát bất thường: `abort()`

[i[`abort()` function]<]

Bạn có thể dùng cái này nếu có gì đó sai khủng khiếp và bạn muốn báo
như vậy cho môi trường bên ngoài. Cái này cũng không nhất thiết dọn
dẹp file mở nào.

Tôi hiếm thấy cái này được dùng.

Hé lộ chút về _signal_: cái này thực ra hoạt động bằng cách raise
một [i[`SIGABRT` signal]] `SIGABRT` sẽ kết thúc tiến trình.

Chuyện gì xảy ra sau đó tùy hệ thống, nhưng trên các hệ Unix-like,
thường [flw[dump core|Core_dump]] khi chương trình kết thúc.

[i[`abort()` function]>]
[i[Exiting]>]

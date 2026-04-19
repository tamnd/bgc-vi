<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Ngày giờ

> "Time is an illusion. Lunchtime doubly so." \
> ---Ford Prefect, The Hitchhikers Guide to the Galaxy

[i[Date and time]<]

Cái này không quá phức tạp, nhưng ban đầu có thể hơi nản, cả với
các kiểu khác nhau có sẵn và cách ta chuyển qua lại giữa chúng.

Trộn thêm GMT (UTC) và local time và ta có mọi _Niềm Vui Thường
Lệ_™ mà người ta có với ngày giờ.

Và đương nhiên đừng bao giờ quên quy tắc vàng của ngày giờ: _Đừng
bao giờ cố viết chức năng ngày giờ của riêng bạn. Chỉ dùng cái thư
viện cho._

Thời gian quá phức tạp đối với các lập trình viên phàm phu. Nghiêm
túc, ta nợ một điểm mỗi người đã làm việc trên bất kỳ thư viện ngày
giờ nào, nên bỏ cái đó vào ngân sách.

## Thuật ngữ và thông tin nhanh

Vài thuật ngữ nhanh phòng khi bạn chưa nắm rõ.

* [i[Universal Coordinated Time]] **UTC**: Coordinated Universal
  Time là thời gian tuyệt đối được đồng thuận toàn cầu^[Trên Trái
  Đất, dù sao. Ai biết họ dùng hệ điên rồ gì _ngoài kia_...]. Mọi
  người trên hành tinh nghĩ bây giờ là cùng một thời điểm theo UTC...
  dù họ có giờ địa phương khác nhau.

* [i[Greenwich Mean time]] **GMT**: Greenwich Mean Time, về cơ bản
  giống UTC^[OK, đừng giết tôi! GMT về kỹ thuật là múi giờ còn UTC
  là hệ thời gian toàn cầu. Ngoài ra vài nước có thể chỉnh GMT cho
  tiết kiệm ánh sáng ban ngày, trong khi UTC không bao giờ được
  chỉnh cho tiết kiệm ánh sáng ban ngày.]. Bạn có lẽ muốn nói UTC,
  hay "giờ toàn cầu". Nếu bạn nói cụ thể về múi giờ GMT, nói GMT.
  Gây lẫn lộn là, nhiều hàm UTC của C có trước UTC và vẫn dùng tên
  Greenwich Mean Time. Khi bạn thấy thế, biết rằng C ý là UTC.

* [i[Local time]] **Local time**: giờ ở nơi máy tính đang chạy
  chương trình. Cái này được mô tả như một độ lệch so với UTC. Dù có
  nhiều múi giờ trên thế giới, đa số máy tính làm việc ở local time
  hoặc UTC.

[i[Universal Coordinated Time]<]

Theo quy tắc chung, nếu bạn đang mô tả sự kiện xảy ra một lần, như
một entry log, hay một vụ phóng tên lửa, hay khi con trỏ cuối cùng
cũng click trong đầu bạn, dùng UTC.

[i[Local time]<]

Mặt khác, nếu là chuyện gì đó xảy ra cùng giờ _ở mọi múi giờ_, như
đêm giao thừa hay giờ ăn tối, dùng local time.

Vì nhiều ngôn ngữ chỉ giỏi chuyển qua lại UTC và local time, bạn có
thể tự gây đau đầu rất nhiều nếu chọn lưu ngày theo dạng sai. (Hỏi
tôi sao tôi biết.)

[i[Local time]>]
[i[Universal Coordinated Time]>]


## Kiểu ngày

Có hai^[Thực ra là nhiều hơn hai.] kiểu chính trong C khi dính tới
ngày: `time_t` và `struct tm`.

Spec thực ra không nói nhiều về chúng:

* [i[`time_t` type]] `time_t`: kiểu thực có khả năng giữ một thời
  gian. Nên theo spec, cái này có thể là kiểu dấu chấm động hay kiểu
  số nguyên. Trong POSIX (các hệ Unix-like), nó là số nguyên. Cái
  này giữ _calendar time_. Bạn có thể nghĩ như giờ UTC.

* [i[`struct tm` type]] `struct tm`: giữ các thành phần của một
  calendar time. Đây là một _broken-down time_, tức là, các thành
  phần của thời gian, như giờ, phút, giây, ngày, tháng, năm, v.v.

[i[`time_t` type]<]

Trên nhiều hệ, `time_t` đại diện cho số giây kể từ
[flw[_Epoch_|Unix_time]]. Epoch theo một nghĩa là khởi đầu thời gian
theo góc nhìn của máy tính, thường là 1 tháng 1, 1970 UTC. `time_t`
có thể âm để đại diện cho thời gian trước Epoch. Windows hoạt động
tương tự Unix theo tôi thấy.

[i[`time_t` type]>]
[i[`struct tm` type]<]

Và trong `struct tm` có gì? Các field sau:

``` {.c}
struct tm {
    int tm_sec;    // seconds after the minute -- [0, 60]
    int tm_min;    // minutes after the hour -- [0, 59]
    int tm_hour;   // hours since midnight -- [0, 23]
    int tm_mday;   // day of the month -- [1, 31]
    int tm_mon;    // months since January -- [0, 11]
    int tm_year;   // years since 1900
    int tm_wday;   // days since Sunday -- [0, 6]
    int tm_yday;   // days since January 1 -- [0, 365]
    int tm_isdst;  // Daylight Saving Time flag
};
```

Lưu ý mọi thứ đều bắt đầu từ zero trừ ngày trong tháng.

Quan trọng là biết rằng bạn có thể đặt bất cứ giá trị nào vào các
kiểu này bạn muốn. Có các hàm giúp lấy thời gian _hiện tại_, nhưng
kiểu giữ _một_ thời gian, không phải _thời gian_.

[i[`struct tm` type]>]

Nên câu hỏi trở thành: "Làm sao khởi tạo dữ liệu các kiểu này, và
làm sao chuyển giữa chúng?"

## Khởi tạo và chuyển giữa các kiểu

[i[`time()` function]<]

Trước hết, bạn có thể lấy thời gian hiện tại và lưu nó vào `time_t`
với hàm `time()`.

``` {.c}
time_t now;  // Variable to hold the time now

now = time(NULL);  // You can get it like this...

time(&now);        // ...or this. Same as the previous line.
```

Tuyệt! Bạn có biến lấy được thời gian hiện tại.

[i[`time()` function]>]
[i[`ctime()` function]<]

Vui là, chỉ có một cách portable để in ra thứ có trong `time_t`, và
đó là hàm `ctime()` hiếm dùng, in giá trị theo local time:


``` {.c}
now = time(NULL);
printf("%s", ctime(&now));
```

Cái này trả chuỗi có dạng rất cụ thể bao gồm newline ở cuối:

``` {.default}
Sun Feb 28 18:47:25 2021
```

[i[`ctime()` function]>]

Nên cái đó hơi cứng nhắc. Nếu bạn muốn kiểm soát hơn, bạn nên chuyển
`time_t` đó thành `struct tm`.

### Chuyển `time_t` sang `struct tm`

[i[`time_t` type-->conversion to `struct tm`]<]

Có hai cách kỳ diệu để làm chuyển này:

[i[`localtime()` function]<]

* `localtime()`: hàm này chuyển `time_t` sang `struct tm` theo local
  time.

[i[`gmtime()` function]<]

* `gmtime()`: hàm này chuyển `time_t` sang `struct tm` theo UTC.
  (Thấy GMT xưa chui vào tên hàm đó chứ?)

[i[`asctime()` function]<]

Xem giờ hiện tại bằng cách in ra `struct tm` bằng hàm `asctime()`:

``` {.c}
printf("Local: %s", asctime(localtime(&now)));
printf("  UTC: %s", asctime(gmtime(&now)));
```

[i[`asctime()` function]>]
[i[`localtime()` function]>]
[i[`gmtime()` function]>]

Output (tôi ở múi giờ Pacific Standard):

``` {.default}
Local: Sun Feb 28 20:15:27 2021
  UTC: Mon Mar  1 04:15:27 2021
```

Một khi bạn có `time_t` trong `struct tm`, nó mở ra đủ loại cánh
cửa. Bạn có thể in thời gian theo đủ kiểu, tìm xem một ngày là thứ
mấy trong tuần, v.v. Hoặc chuyển nó ngược lại thành `time_t`.

Sẽ nói thêm về cái đó sớm!

[i[`time_t` type-->conversion to `struct tm`]>]

### Chuyển `struct tm` sang `time_t`

[i[`struct tm` type-->conversion to `time_t`]<]
[i[`mktime()` function]<]

Nếu bạn muốn đi theo chiều ngược, bạn có thể dùng `mktime()` để lấy
thông tin đó.

`mktime()` set giá trị của `tm_wday` và `tm_yday` giùm bạn, nên đừng
phí công điền chúng vì chúng sẽ bị ghi đè thôi.

Ngoài ra, bạn có thể set `tm_isdst` thành `-1` để nó tự quyết định
giùm bạn. Hoặc bạn có thể set thủ công thành true hay false.

``` {.c}
// Don't be tempted to put leading zeros on these numbers (unless you
// mean for them to be in octal)!

struct tm some_time = {
    .tm_year=82,   // years since 1900
    .tm_mon=3,     // months since January -- [0, 11]
    .tm_mday=12,   // day of the month -- [1, 31]
    .tm_hour=12,   // hours since midnight -- [0, 23]
    .tm_min=0,     // minutes after the hour -- [0, 59]
    .tm_sec=4,     // seconds after the minute -- [0, 60]
    .tm_isdst=-1,  // Daylight Saving Time flag
};

time_t some_time_epoch;

some_time_epoch = mktime(&some_time);

printf("%s", ctime(&some_time_epoch));
printf("Is DST: %d\n", some_time.tm_isdst);
```

Output:

``` {.default}
Mon Apr 12 12:00:04 1982
Is DST: 0
```

Khi bạn nạp thủ công một `struct tm` như vậy, nó nên là local time.
`mktime()` sẽ chuyển local time đó thành `time_t` calendar time.

[i[`mktime()` function]>]

Lạ là, tuy vậy, chuẩn không cho ta cách nạp một `struct tm` với thời
gian UTC và chuyển nó thành `time_t`. Nếu bạn muốn làm vậy với các
hệ Unix-like, thử hàm không chuẩn [i[`timegm()` Unix function]]
`timegm()`. Trên Windows, [i[`_mkgmtime()` Windows function]]
`_mkgmtime()`.

[i[`struct tm` type-->conversion to `time_t`]>]

## In ngày theo định dạng

Ta đã thấy vài cách in output ngày có định dạng lên màn hình. Với
`time_t` ta dùng `ctime()`, và với `struct tm` ta dùng `asctime()`.

``` {.c}
time_t now = time(NULL);
struct tm *local = localtime(&now);
struct tm *utc = gmtime(&now);

printf("Local time: %s", ctime(&now));     // Local time with time_t
printf("Local time: %s", asctime(local));  // Local time with struct tm
printf("UTC       : %s", asctime(utc));    // UTC with a struct tm
```

Nhưng nếu tôi nói với bạn, độc giả thân mến, rằng có cách kiểm soát
nhiều hơn cách ngày được in ra thì sao?

[i[`strftime()` function]<]

Chắc chắn, ta có thể câu từng field từ `struct tm`, nhưng có một hàm
tuyệt gọi là `strftime()` sẽ làm nhiều phần khó cho bạn. Nó giống
`printf()` chỉ khác là cho ngày!

Xem vài ví dụ. Trong mỗi cái, ta truyền vào buffer đích, số ký tự
tối đa để ghi, và rồi chuỗi format (theo phong cách của, nhưng không
giống, `printf()`) bảo `strftime()` thành phần nào của `struct tm`
cần in và in sao.

Bạn có thể thêm ký tự hằng khác để đưa vào output trong chuỗi
format, cũng như với `printf()`.

Ta lấy `struct tm` trong trường hợp này từ `localtime()`, nhưng bất
kỳ nguồn nào cũng ổn.

``` {.c .numberLines}
#include <stdio.h>
#include <time.h>

int main(void)
{
    char s[128];
    time_t now = time(NULL);

    // %c: print date as per current locale
    strftime(s, sizeof s, "%c", localtime(&now));
    puts(s);   // Sun Feb 28 22:29:00 2021

    // %A: full weekday name
    // %B: full month name
    // %d: day of the month
    strftime(s, sizeof s, "%A, %B %d", localtime(&now));
    puts(s);   // Sunday, February 28

    // %I: hour (12 hour clock)
    // %M: minute
    // %S: second
    // %p: AM or PM
    strftime(s, sizeof s, "It's %I:%M:%S %p", localtime(&now));
    puts(s);   // It's 10:29:00 PM

    // %F: ISO 8601 yyyy-mm-dd
    // %T: ISO 8601 hh:mm:ss
    // %z: ISO 8601 time zone offset
    strftime(s, sizeof s, "ISO 8601: %FT%T%z", localtime(&now));
    puts(s);   // ISO 8601: 2021-02-28T22:29:00-0800
}
```

Có _cả tấn_ format specifier in ngày cho `strftime()`, nên nhớ xem
chúng trong [fl[trang tham khảo
`strftime()`|https://beej.us/guide/bgclr/html/split/time.html#man-strftime]].

[i[`strftime()` function]>]

## Độ phân giải cao hơn với `timespec_get()`

[i[`timespec_get()` function]<]

Bạn có thể lấy số giây và nanosecond kể từ Epoch với
`timespec_get()`.

Có thể.

Các cài đặt có thể không có độ phân giải nanosecond (là một phần tỷ
giây) nên ai biết bạn sẽ có bao nhiêu chữ số có nghĩa, nhưng cứ thử
xem.

[i[`struct timespec` type]<]

`timespec_get()` nhận hai đối số. Một là con trỏ tới `struct
timespec` để giữ thông tin thời gian. Và cái kia là `base`, mà spec
cho phép bạn set thành `TIME_UTC` báo rằng bạn quan tâm tới số giây
kể từ Epoch. (Các cài đặt khác có thể cho bạn thêm lựa chọn cho
`base`.)

Và bản thân cấu trúc có hai field:

``` {.c}
struct timespec {
    time_t tv_sec;   // Seconds
    long   tv_nsec;  // Nanoseconds (billionths of a second)
};
```

Đây là ví dụ ta lấy thời gian và in ra cả giá trị số nguyên lẫn giá
trị dấu chấm động:

``` {.c}
struct timespec ts;

timespec_get(&ts, TIME_UTC);

printf("%ld s, %ld ns\n", ts.tv_sec, ts.tv_nsec);

double float_time = ts.tv_sec + ts.tv_nsec/1000000000.0;
printf("%f seconds since epoch\n", float_time);
```

Ví dụ output:

``` {.default}
1614581530 s, 806325800 ns
1614581530.806326 seconds since epoch
```

`struct timespec` cũng xuất hiện ở một số hàm threading cần có khả
năng chỉ định thời gian với độ phân giải đó.

[i[`struct timespec` type]>]
[i[`timespec_get()` function]>]

## Khác biệt giữa các thời gian

[i[Date and time-->differences]<]

Một lưu ý nhanh về lấy khác biệt giữa hai `time_t`: vì spec không
quy định kiểu đó biểu diễn thời gian sao, bạn có thể không thể chỉ
đơn giản trừ hai `time_t` và ra gì có nghĩa^[Bạn sẽ làm được trên
POSIX, nơi `time_t` chắc chắn là số nguyên. Không may cả thế giới
không phải POSIX, nên vậy đó.].

[i[`difftime()` function]<]

May thay bạn có thể dùng `difftime()` để tính khác biệt tính bằng
giây giữa hai ngày.

Trong ví dụ sau, ta có hai sự kiện xảy ra cách nhau một khoảng thời
gian, và ta dùng `difftime()` để tính khác biệt.

``` {.c .numberLines}
#include <stdio.h>
#include <time.h>

int main(void)
{
    struct tm time_a = {
        .tm_year=82,   // years since 1900
        .tm_mon=3,     // months since January -- [0, 11]
        .tm_mday=12,   // day of the month -- [1, 31]
        .tm_hour=4,    // hours since midnight -- [0, 23]
        .tm_min=00,    // minutes after the hour -- [0, 59]
        .tm_sec=04,    // seconds after the minute -- [0, 60]
        .tm_isdst=-1,  // Daylight Saving Time flag
    };

    struct tm time_b = {
        .tm_year=120,  // years since 1900
        .tm_mon=10,    // months since January -- [0, 11]
        .tm_mday=15,   // day of the month -- [1, 31]
        .tm_hour=16,   // hours since midnight -- [0, 23]
        .tm_min=27,    // minutes after the hour -- [0, 59]
        .tm_sec=00,    // seconds after the minute -- [0, 60]
        .tm_isdst=-1,  // Daylight Saving Time flag
    };

    time_t cal_a = mktime(&time_a);
    time_t cal_b = mktime(&time_b);

    double diff = difftime(cal_b, cal_a);

    double years = diff / 60 / 60 / 24 / 365.2425;  // close enough

    printf("%f seconds (%f years) between events\n", diff, years);
}
```

Output:

``` {.default}
1217996816.000000 seconds (38.596783 years) between events
```

Và bạn có rồi đó! Nhớ dùng `difftime()` để lấy khác biệt thời gian.
Dù bạn có thể chỉ trừ trên hệ POSIX, cứ giữ portable thôi.

[i[`difftime()` function]>]
[i[Date and time-->differences]>]
[i[Date and time]>]

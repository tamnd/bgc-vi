<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Môi trường bên ngoài

Khi bạn chạy một chương trình, thực ra là bạn đang nói chuyện với
shell, kiểu: "Này, chạy giùm cái này với." Rồi shell nói: "Được,"
rồi nó bảo hệ điều hành: "Này, anh tạo tiến trình mới rồi chạy cái
này giùm được không?" Và nếu mọi chuyện suôn sẻ, OS làm theo và
chương trình của bạn chạy.

Nhưng ngoài chương trình, trong shell có cả một thế giới mà từ trong
C có thể tương tác được. Ta sẽ ngó qua vài thứ trong chương này.

## Tham số dòng lệnh

[i[Command line arguments]<]

Nhiều tiện ích dòng lệnh nhận _tham số dòng lệnh_. Ví dụ, nếu ta
muốn xem mọi file kết thúc bằng `.txt`, ta có thể gõ đại loại thế
này trên hệ Unix-like:

``` {.zsh}
ls *.txt
```

(hoặc `dir` thay cho `ls` trên hệ Windows).

Trong trường hợp này, lệnh là `ls`, nhưng tham số của nó là mọi file
kết thúc bằng `.txt`^[Về mặt lịch sử, chương trình trên MS-DOS và
Windows làm chuyện này khác Unix. Ở Unix, shell sẽ _mở rộng_ ký tự
đại diện thành mọi file khớp trước khi chương trình của bạn thấy
được, còn mấy bản của Microsoft sẽ chuyển cả biểu thức ký tự đại
diện vào chương trình để tự xử. Dù sao, vẫn có tham số được chuyển
vào chương trình.].

Vậy làm sao để xem thứ gì được truyền vào chương trình từ dòng lệnh?

Giả sử ta có chương trình tên `add` cộng mọi số truyền trên dòng
lệnh rồi in kết quả:

``` {.zsh}
./add 10 30 5
45
```

Chắc chắn cái này sẽ kiếm đủ tiền trả hóa đơn đây!

Nhưng nghiêm túc, đây là công cụ hay để xem cách lấy tham số từ dòng
lệnh rồi xử lý chúng.

Đầu tiên, xem cách lấy chúng ra đã. Cho chuyện này, ta cần một
`main()` mới!

Đây là chương trình in ra tất cả tham số dòng lệnh. Ví dụ, nếu đặt
tên file thực thi là `foo`, ta chạy thế này:

``` {.zsh}
./foo i like turtles
```

và ta sẽ thấy output:

``` {.default}
arg 0: ./foo
arg 1: i
arg 2: like
arg 3: turtles
```

Hơi lạ, vì tham số thứ không là tên file thực thi. Nhưng quen thôi.
Các tham số còn lại thì theo sau trực tiếp.

Nguồn:

[i[`argc` parameter]<]
[i[`argv` parameter]<]
[i[`main()` function-->command line options]<]

``` {.c .numberLines}
#include <stdio.h>

int main(int argc, char *argv[])
{
    for (int i = 0; i < argc; i++) {
        printf("arg %d: %s\n", i, argv[i]);
    }
}
```

Oái! Có chuyện gì với chữ ký của `main()` thế? `argc` và `argv`^[Vì
chúng chỉ là tên tham số thông thường, bạn không nhất thiết phải gọi
là `argc` và `argv`. Nhưng đó là idiomatic đến mức nếu bạn sáng tạo,
dev C khác sẽ nhìn bạn bằng ánh mắt nghi hoặc thật sự đấy!] (đọc là
_arg-cee_ và _arg-vee_) là gì vậy?

Bắt đầu với cái dễ trước: `argc`. Đây là _argument count_, tức số
lượng tham số, bao gồm cả tên chương trình. Nếu bạn hình dung mọi
tham số như một mảng chuỗi, mà chúng đúng là vậy, thì `argc` là độ
dài của mảng đó, mà nó đúng là thế.

Và vậy chuyện ta làm trong vòng lặp là duyệt qua mọi `argv` và in
từng cái một, nên với input:

``` {.zsh}
./foo i like turtles
```

ta có output tương ứng:

``` {.default}
arg 0: ./foo
arg 1: i
arg 2: like
arg 3: turtles
```

Với chừng đó trong đầu, ta đủ đồ để làm chương trình cộng.

Kế hoạch:

* Xem mọi tham số dòng lệnh (qua khỏi `argv[0]`, tên chương trình)
* Đổi chúng sang số nguyên
* Cộng dồn vào tổng đang chạy
* In kết quả

Bắt tay vào!

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    int total = 0;

    for (int i = 1; i < argc; i++) {  // Start at 1, the first argument
        int value = atoi(argv[i]);    // Use strtol() for better error handling

        total += value;
    }

    printf("%d\n", total);
}
```

[i[`main()` function-->command line options]>]
[i[`argc` parameter]>]

Vài lần chạy thử:

``` {.zsh}
$ ./add
0
$ ./add 1
1
$ ./add 1 2
3
$ ./add 1 2 3
6
$ ./add 1 2 3 4
10
```

Dĩ nhiên nó có thể phun bậy nếu bạn truyền vào cái gì không phải số
nguyên, nhưng việc làm cứng cáp chuyện đó xin để lại làm bài tập cho
người đọc.

### `argv` cuối cùng là `NULL`

Một điều vui vui về `argv` là sau chuỗi cuối cùng là một con trỏ tới
`NULL`.

Tức là:

``` {.c}
argv[argc] == NULL
```

luôn đúng!

Chuyện này có vẻ vô nghĩa, nhưng hóa ra lại hữu ích ở vài chỗ, ta sẽ
xem một trong số đó ngay bây giờ.

### Dạng thay thế: `char **argv`

Nhớ rằng khi gọi hàm, C không phân biệt ký pháp mảng và ký pháp con
trỏ trong chữ ký hàm.

Tức là, hai thứ sau là như nhau:

``` {.c}
void foo(char a[])
void foo(char *a)
```

Lâu nay ta hình dung `argv` như một mảng chuỗi, tức là một mảng các
`char*`, nên cái này nghe hợp lý:

``` {.c}
int main(int argc, char *argv[])
```

nhưng vì sự tương đương đó, bạn cũng có thể viết:

``` {.c}
int main(int argc, char **argv)
```

Ừ, con trỏ trỏ tới con trỏ! Nếu thấy dễ hơn, cứ nghĩ nó như con trỏ
tới chuỗi. Nhưng thực ra, nó là con trỏ trỏ tới một giá trị mà giá
trị đó trỏ tới `char`.

Cũng nhớ rằng hai thứ này tương đương:

``` {.c}
argv[i]
*(argv + i)
```

nghĩa là bạn có thể làm số học con trỏ trên `argv`.

Vậy một cách khác để tiêu thụ tham số dòng lệnh có thể là đi dọc
mảng `argv` bằng cách tăng con trỏ lên cho tới khi chạm `NULL` ở
cuối.

Sửa chương trình cộng của ta để làm vậy:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    int total = 0;
    
    // Cute trick to get the compiler to stop warning about the
    // unused variable argc:
    (void)argc;

    for (char **p = argv + 1; *p != NULL; p++) {
        int value = atoi(*p);  // Use strtol() for better error handling

        total += value;
    }

    printf("%d\n", total);
}
```

Cá nhân tôi dùng ký pháp mảng để truy cập `argv`, nhưng tôi vẫn thấy
kiểu này lảng vảng đâu đó.

### Ít chuyện vui

[i[`argc` parameter]<]

Còn vài thứ về `argc` và `argv`.

* Vài môi trường có thể không đặt `argv[0]` là tên chương trình. Nếu
  nó không có, `argv[0]` sẽ là chuỗi rỗng. Tôi chưa bao giờ thấy
  chuyện này xảy ra.

* Spec thực ra khá thoáng trong chuyện implementation có thể làm gì
  với `argv` và các giá trị đó đến từ đâu. Nhưng mọi hệ tôi từng
  dùng đều hoạt động giống nhau, như ta đã bàn ở phần này.

* Bạn có thể sửa `argc`, `argv`, hoặc bất kỳ chuỗi nào mà `argv` trỏ
  tới. (Chỉ là đừng làm mấy chuỗi đó dài hơn kích thước sẵn có của
  nó!)

* Trên vài hệ Unix-like, sửa chuỗi `argv[0]` sẽ khiến output của
  `ps` thay đổi^[`ps`, Process Status, là lệnh Unix để xem tiến
  trình nào đang chạy lúc đó.].

  Thông thường, nếu bạn có chương trình tên `foo` chạy bằng `./foo`,
  bạn có thể thấy trong output của `ps`:

  ``` {.default}
   4078 tty1     S      0:00 ./foo
  ```

  Nhưng nếu sửa `argv[0]` thế này, cẩn thận để chuỗi mới `"Hi!  "`
  có cùng độ dài với chuỗi cũ `"./foo"`:

  ``` {.c}
  strcpy(argv[0], "Hi!  ");
  ```

  rồi chạy `ps` khi chương trình `./foo` còn đang chạy, ta sẽ thấy:
  
  ``` {.default}
   4079 tty1     S      0:00 Hi!  
  ```

  Hành vi này không có trong spec và phụ thuộc rất nhiều vào hệ
  thống.

[i[`argc` parameter]>]
[i[`argv` parameter]>]
[i[Command line arguments]>]

## Exit status {#exit-status}

[i[Exit status]<]

Bạn có để ý chữ ký của `main()` trả về kiểu `int` không? Chuyện đó
là sao? Nó liên quan tới thứ gọi là _exit status_, một số nguyên có
thể được trả lại chương trình đã khởi chạy chương trình của bạn để
báo mọi chuyện ra sao.

Có cả đống cách để chương trình thoát trong C, bao gồm `return` từ
`main()`, hay gọi một trong các biến thể `exit()`.

Tất cả các cách này nhận `int` làm tham số.

[i[`main()` function-->returning from]<]
Nhắc lề: bạn có thấy trong phần lớn các ví dụ của tôi, dù `main()`
lẽ ra phải trả về `int`, tôi thật ra không `return` gì hết? Trong
mọi hàm khác, chuyện này là bất hợp pháp, nhưng có một ngoại lệ đặc
biệt trong C: nếu luồng thực thi chạm đến cuối `main()` mà không tìm
thấy `return`, nó tự động làm `return 0`.
[i[`main()` function-->returning from]>]

Nhưng `0` đó nghĩa là gì? Các số khác ta có thể đặt vào đó là gì? Và
chúng được dùng ra sao?

Spec vừa rõ vừa mơ hồ về chuyện này, như thường lệ. Rõ vì nó nói ra
bạn có thể làm gì, mơ hồ vì nó không giới hạn gì mấy.

Chẳng còn cách nào khác là _cứ tiến lên_ và tìm ra!

Ta [flw[Inception|Inception]] một chút: hóa ra khi bạn chạy chương
trình, _bạn đang chạy nó từ một chương trình khác_.

Thường chương trình kia là kiểu [flw[shell|Shell_(computing)]] nào
đó mà tự nó không làm mấy ngoài chuyện khởi chạy chương trình khác.

Nhưng đây là quy trình nhiều giai đoạn, nhất là với shell dòng lệnh
thấy rõ:

1. Shell khởi chạy chương trình của bạn
2. Shell thường đi ngủ (với shell dòng lệnh)
3. Chương trình của bạn chạy
4. Chương trình kết thúc
5. Shell thức dậy và đợi lệnh tiếp theo

Bây giờ, có một mẩu thông tin liên lạc diễn ra giữa bước 4 và 5:
chương trình có thể trả về một _giá trị trạng thái_ mà shell có thể
hỏi. Thường giá trị này được dùng để báo chương trình thành công hay
thất bại, và nếu thất bại thì kiểu thất bại nào.

Giá trị này là cái ta vẫn `return` từ `main()`. Đó là status.

Giờ, spec C cho phép hai giá trị status khác nhau, có tên macro được
định nghĩa trong `<stdlib.h>`:

[i[`EXIT_SUCCESS` macro]<]
[i[`EXIT_FAILURE` macro]<]

|Status|Mô tả|
|-|-|
|`EXIT_SUCCESS` hay `0`|Chương trình kết thúc thành công.|
|`EXIT_FAILURE`|Chương trình kết thúc với lỗi.|

Hãy viết một chương trình ngắn nhân hai số từ dòng lệnh. Ta sẽ yêu
cầu bạn chỉ định chính xác hai giá trị. Nếu không, ta in thông báo
lỗi và thoát với status lỗi.

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    if (argc != 3) {
        printf("usage: mult x y\n");
        return EXIT_FAILURE;   // Indicate to shell that it didn't work
    }

    printf("%d\n", atoi(argv[1]) * atoi(argv[2]));

    return 0;  // same as EXIT_SUCCESS, everything was good.
}
```

[i[`EXIT_SUCCESS` macro]>]
[i[`EXIT_FAILURE` macro]>]

Giờ thử chạy, ta sẽ thấy đúng như ý cho đến khi truyền đúng số lượng
tham số dòng lệnh:

``` {.zsh}
$ ./mult
usage: mult x y

$ ./mult 3 4 5
usage: mult x y

$ ./mult 3 4
12
```

[i[Exit status-->obtaining from shell]<]

Nhưng cái đó không thật sự cho thấy exit status mà ta trả về, đúng
không? Ta có thể bắt shell in nó ra. Giả sử bạn đang dùng Bash hoặc
shell POSIX khác, có thể dùng `echo $?` để xem^[Trên Windows
`cmd.exe`, gõ `echo %errorlevel%`. Trong PowerShell, gõ
`$LastExitCode`.].

Thử xem:

``` {.zsh}
$ ./mult
usage: mult x y
$ echo $?
1

$ ./mult 3 4 5
usage: mult x y
$ echo $?
1

$ ./mult 3 4
12
$ echo $?
0
```

[i[Exit status-->obtaining from shell]>]

Thú vị! Ta thấy trên hệ của tôi, [i[`EXIT_FAILURE`
macro]]`EXIT_FAILURE` là `1`. Spec không nói rõ chuyện này, nên nó
có thể là số bất kỳ. Nhưng cứ thử, trên hệ của bạn chắc cũng là `1`.

### Các giá trị exit status khác

Status `0` chắc chắn nghĩa là thành công, nhưng mọi số nguyên khác,
kể cả âm, thì sao?

Ở đây ta đi ra khỏi spec C mà bước vào địa phận Unix. Nhìn chung,
`0` là thành công, còn số dương khác không là thất bại. Vậy bạn chỉ
có một kiểu thành công, nhưng nhiều kiểu thất bại. Bash nói exit
code nên nằm trong khoảng 0 đến 255, dù một số code đã được reserved. 

Nói ngắn, nếu muốn báo các exit status lỗi khác nhau trong môi
trường Unix, bạn có thể bắt đầu từ `1` và tăng dần.

Trên Linux, nếu bạn thử code nào nằm ngoài khoảng 0-255, nó sẽ AND
bitwise code với `0xff`, kẹp nó vào khoảng đó.

Bạn có thể script shell để dùng các code status này quyết định làm
gì tiếp theo.

[i[Exit status]>]

## Biến môi trường {#env-var}

[i[Environment variables]<]

Trước khi đi vào phần này, tôi phải cảnh báo rằng C không định
nghĩa biến môi trường là gì. Nên tôi sẽ mô tả hệ thống biến môi
trường hoạt động trên mọi nền tảng lớn tôi biết.

Về cơ bản, môi trường là chương trình sẽ chạy chương trình của bạn,
ví dụ shell bash. Và nó có thể có vài biến bash được định nghĩa.
Nếu bạn chưa biết, shell có thể tự tạo biến riêng. Mỗi shell mỗi
khác, nhưng trong bash bạn chỉ cần gõ `set` là nó hiện hết cho bạn.

Đây là trích đoạn từ 61 biến được định nghĩa trong bash shell của
tôi:

``` {.default}
HISTFILE=/home/beej/.bash_history
HISTFILESIZE=500
HISTSIZE=500
HOME=/home/beej
HOSTNAME=FBILAPTOP
HOSTTYPE=x86_64
IFS=$' \t\n'
```

Chú ý chúng ở dạng cặp key/value. Ví dụ, một key là `HOSTTYPE` và
giá trị là `x86_64`. Từ góc nhìn C, tất cả giá trị là chuỗi, dù
chúng là số^[Nếu bạn cần giá trị số, chuyển chuỗi bằng thứ như
`atoi()` hay `strtol()`.].

Vậy _dù sao_! Chuyện dài kể ngắn, bạn có thể lấy các giá trị này
từ bên trong chương trình C của bạn.

[i[`getenv()` function]<]

Viết chương trình dùng hàm chuẩn `getenv()` để tra một giá trị mà
bạn đặt trong shell.

`getenv()` sẽ trả về con trỏ tới chuỗi giá trị, hoặc `NULL` nếu
biến môi trường không tồn tại.

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *val = getenv("FROTZ");  // Try to get the value

    // Check to make sure it exists
    if (val == NULL) {
        printf("Cannot find the FROTZ environment variable\n");
        return EXIT_FAILURE;
    }

    printf("Value: %s\n", val);
}
```

[i[`getenv()` function]>]

Nếu chạy trực tiếp, tôi thấy thế này:

``` {.zsh}
$ ./foo
Cannot find the FROTZ environment variable
```

chuyện đó hợp lý, vì tôi chưa đặt nó.

Trong bash, tôi có thể đặt nó bằng^[Trong Windows CMD.EXE, dùng
`set FROTZ=value`. Trong PowerShell, dùng `$Env:FROTZ=value`.]:

``` {.zsh}
$ export FROTZ="C is awesome!"
```

Rồi khi chạy, tôi thấy:

``` {.zsh}
$ ./foo
Value: C is awesome!
```

Theo cách này, bạn có thể đặt dữ liệu trong biến môi trường, và có
thể lấy nó trong code C rồi thay đổi hành vi tương ứng.

### Đặt biến môi trường

Cái này không chuẩn, nhưng nhiều hệ có cách để đặt biến môi trường.

Nếu trên hệ Unix-like, tra tài liệu cho [i[`putenv()`
function]]`putenv()`, [i[`setenv()`]]`setenv()`, và [i[`unsetenv()`
function]]`unsetenv()`. Trên Windows, xem [i[`_putenv()`
function]]`_putenv()`.

### Biến môi trường thay thế trên Unix-like

Nếu bạn đang trên hệ Unix-like, nhiều khả năng bạn có thêm vài cách
nữa để truy cập biến môi trường. Lưu ý rằng dù spec chỉ ra đây là
phần mở rộng phổ biến, nó không thật sự là phần của chuẩn C. Nhưng
nó là phần của chuẩn POSIX.

[i[`environ` variable]<]

Một trong số đó là biến tên `environ` phải được khai báo thế này:

``` {.c}
extern char **environ;
```

Nó là một mảng chuỗi kết thúc bằng con trỏ `NULL`.

Bạn nên tự khai báo trước khi dùng, hoặc có thể tìm thấy nó trong
file header không chuẩn `<unistd.h>`.

Mỗi chuỗi ở dạng `"key=value"` nên bạn phải tự tách rồi phân tích
nếu muốn lấy key và value ra.

Đây là ví dụ lặp qua và in các biến môi trường bằng vài cách khác
nhau:

``` {.c .numberLines}
#include <stdio.h>

extern char **environ;  // MUST be extern AND named "environ"

int main(void)
{
    for (char **p = environ; *p != NULL; p++) {
        printf("%s\n", *p);
    }

    // Or you could do this:
    for (int i = 0; environ[i] != NULL; i++) {
        printf("%s\n", environ[i]);
    }
}
```

Cho ra một đống output như thế này:

``` {.default}
SHELL=/bin/bash
COLORTERM=truecolor
TERM_PROGRAM_VERSION=1.53.2
LOGNAME=beej
HOME=/home/beej
... etc ...
```

Dùng `getenv()` nếu có thể vì nó di động hơn. Nhưng nếu bạn phải
lặp qua các biến môi trường, dùng `environ` có thể là cách hợp.

[i[`environ` variable]>]
[i[`env` parameter]<]

Một cách không chuẩn khác để lấy biến môi trường là làm tham số của
`main()`. Nó hoạt động khá giống, nhưng bạn khỏi phải thêm biến
`extern` `environ`. [fl[Ngay cả spec POSIX cũng không hỗ trợ cái
này|https://pubs.opengroup.org/onlinepubs/9699919799/functions/exec.html]]
theo tôi biết, nhưng nó phổ biến ở chốn Unix.


``` {.c .numberLines}
#include <stdio.h>

int main(int argc, char **argv, char **env)  // <-- env!
{
    (void)argc; (void)argv;  // Suppress unused warnings

    for (char **p = env; *p != NULL; p++) {
        printf("%s\n", *p);
    }

    // Or you could do this:
    for (int i = 0; env[i] != NULL; i++) {
        printf("%s\n", env[i]);
    }
}
```

Giống như dùng `environ` nhưng _còn kém di động hơn_. Có mục tiêu
là tốt.

[i[`env` parameter]>]
[i[Environment variables]>]

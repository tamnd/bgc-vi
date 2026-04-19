<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# `goto`

[i[`goto` statement]<]

Câu lệnh `goto` được cả thế giới tôn sùng và có thể trình ra đây
không ai cãi được.

Đùa thôi! Qua năm tháng, đã có cả đống tranh cãi qua lại về việc
`goto` có [flw[bị coi là có hại|Goto#Criticism]] hay không (thường
là có).

Theo ý của programmer này, bạn nên dùng cấu trúc nào dẫn tới code
_tốt nhất_, có tính tới bảo trì và tốc độ. Và đôi khi cái đó có thể
là `goto`!

Trong chương này, ta sẽ xem `goto` hoạt động sao trong C, rồi ngó
qua vài trường hợp hay dùng^[Tôi muốn nói rõ rằng dùng `goto` trong
tất cả các trường hợp này đều tránh được. Bạn có thể dùng biến và
vòng lặp thay thế. Chỉ là có người thấy `goto` tạo code _tốt nhất_
trong những hoàn cảnh đó.].

## Một ví dụ đơn giản

[i[Labels]<]

Trong ví dụ này, ta sẽ dùng `goto` để bỏ qua một dòng code và nhảy
tới một _label_. Label là identifier có thể làm đích của `goto`, nó
kết thúc bằng dấu hai chấm (`:`).

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    printf("One\n");
    printf("Two\n");

    goto skip_3;

    printf("Three\n");

skip_3:

    printf("Five!\n");
}
```

Output là:

``` {.default}
One
Two
Five!
```

`goto` đẩy thực thi nhảy tới label đã chỉ định, bỏ qua mọi thứ ở
giữa.

Bạn có thể nhảy tiến hay lùi với `goto`.

``` {.c}
infinite_loop:
    print("Hello, world!\n");
    goto infinite_loop;
```

Label bị bỏ qua khi thực thi. Cái sau sẽ in cả ba số theo thứ tự y
như thể các label không có mặt:

``` {.c}
    printf("Zero\n");
label_1:
label_2:
    printf("One\n");
label_3:
    printf("Two\n");
label_4:
    printf("Three\n");
```

Như bạn đã để ý, quy ước phổ biến là căn lề label sát bên trái. Điều
này tăng khả năng đọc vì người đọc có thể quét nhanh để tìm đích.

Label có _function scope_. Tức là, dù chúng xuất hiện ở mức block
sâu bao nhiêu, bạn vẫn có thể `goto` chúng từ bất cứ đâu trong hàm.

Điều đó cũng có nghĩa là bạn chỉ có thể `goto` các label nằm trong
cùng hàm với `goto`. Label ở các hàm khác là ngoài scope theo góc
nhìn của `goto`. Và có nghĩa là bạn có thể dùng cùng tên label trong
hai hàm khác nhau, chỉ không được dùng cùng tên label trong cùng một
hàm.

[i[Labels]>]

## `continue` có label

[i[`goto` statement-->as labeled `continue`]<]

Ở vài ngôn ngữ, bạn thực sự có thể chỉ định label cho câu lệnh
`continue`. C không cho, nhưng bạn có thể dễ dàng dùng `goto` thay
thế.

Để thấy vấn đề, xem `continue` trong vòng lặp lồng này:

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d, %d\n", i, j);
        continue;   // Always goes to next j
    }
}
```

Như ta thấy, `continue` đó, giống như mọi `continue`, đi tới lần lặp
kế của vòng lặp bao quanh gần nhất. Nếu ta muốn `continue` ở vòng
lặp ngoài kế tiếp, vòng lặp với `i` thì sao?

Thì, ta có thể `break` để ra lại vòng lặp ngoài, đúng không?

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d, %d\n", i, j);
        break;     // Gets us to the next iteration of i
    }
}
```

Cái đó giải quyết được hai mức lồng. Nhưng rồi nếu ta lồng thêm vòng
nữa, ta hết lựa chọn. Còn cái này, nơi ta không có câu lệnh nào đưa
ta ra tới lần lặp kế của `i`?

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        for (int k = 0; k < 3; k++) {
            printf("%d, %d, %d\n", i, j, k);

            continue;  // Gets us to the next iteration of k
            break;     // Gets us to the next iteration of j
            ????;      // Gets us to the next iteration of i???

        }
    }
}
```

Câu lệnh `goto` cho ta lối!

``` {.c}
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            for (int k = 0; k < 3; k++) {
                printf("%d, %d, %d\n", i, j, k);

                goto continue_i;   // Now continuing the i loop!!
            }
        }
continue_i: ;
    }
```

Ta có `;` ở cuối đó, vì bạn không thể có label chỉ tới chỗ cuối
thuần của compound statement (hay trước một khai báo biến).

[i[`goto` statement-->as labeled `continue`]>]

## Thoát thân

[i[`goto` statement-->for bailing out]<]

Khi bạn đang lồng cực sâu giữa mớ code, bạn có thể dùng `goto` để
thoát ra theo cách thường sạch hơn là lồng thêm `if` và dùng biến
cờ.

``` {.c}
    // Pseudocode

    for(...) {
        for (...) {
            while (...) {
                do {
                    if (some_error_condition)
                        goto bail;

                } while(...);
            }
        }
    }

bail:
    // Cleanup here
```

Không có `goto`, bạn sẽ phải check cờ điều kiện lỗi trong tất cả các
vòng lặp để thoát hết.

[i[`goto` statement-->for bailing out]>]

## `break` có label

[i[`goto` statement-->as labeled `break`]<]

Tình huống rất giống với chuyện `continue` chỉ continue vòng lặp
trong cùng. `break` cũng chỉ break khỏi vòng lặp trong cùng.

``` {.c}
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d, %d\n", i, j);
            break;   // Only breaks out of the j loop
        }
    }

    printf("Done!\n");
```

Nhưng ta có thể dùng `goto` để break xa hơn:

``` {.c}
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d, %d\n", i, j);
            goto break_i;   // Now breaking out of the i loop!
        }
    }

break_i:

    printf("Done!\n");
```

[i[`goto` statement-->as labeled `break`]>]

## Dọn dẹp nhiều tầng

[i[`goto` statement-->multilevel cleanup]<]

Nếu bạn đang gọi nhiều hàm để khởi tạo nhiều hệ thống con và một
trong số đó fail, bạn chỉ nên de-initialize các cái mà bạn đã tới
được cho tới giờ.

Làm một ví dụ giả trong đó ta bắt đầu khởi tạo hệ thống và check xem
có cái nào trả về lỗi (ta dùng `-1` để báo lỗi). Nếu có, ta phải tắt
chỉ những hệ thống mà ta đã khởi tạo đến lúc đó.

``` {.c}
    if (init_system_1() == -1)
        goto shutdown;

    if (init_system_2() == -1)
        goto shutdown_1;

    if (init_system_3() == -1)
        goto shutdown_2;

    if (init_system_4() == -1)
        goto shutdown_3;

    do_main_thing();   // Run our program

    shutdown_system4();

shutdown_3:
    shutdown_system3();

shutdown_2:
    shutdown_system2();

shutdown_1:
    shutdown_system1();

shutdown:
    print("All subsystems shut down.\n");
```

Lưu ý rằng ta tắt theo thứ tự ngược với thứ tự khởi tạo hệ thống
con. Nên nếu hệ con 4 fail khi khởi động, nó sẽ tắt 3, 2, rồi 1 theo
thứ tự đó.

[i[`goto` statement-->multilevel cleanup]>]

## Tối ưu tail call

[i[`goto` statement-->tail call optimzation]<]
[i[Tail call optimzation-->with `goto`]<]

Kinda. Chỉ cho hàm đệ quy.

Nếu bạn chưa quen, [flw[Tail Call Optimization (TCO)|Tail_call]] là
cách không phí stack space khi gọi hàm khác trong các tình huống rất
cụ thể. Không may chi tiết nằm ngoài phạm vi guide này.

Nhưng nếu bạn có một hàm đệ quy bạn biết có thể được tối ưu theo
kiểu này, bạn có thể tận dụng kỹ thuật này. (Lưu ý bạn không thể
tail call hàm khác vì label có function scope.)

Làm ví dụ thẳng thắn, giai thừa.

Đây là phiên bản đệ quy không phải TCO, nhưng có thể!

``` {.c .numberLines}
#include <stdio.h>
#include <complex.h>

int factorial(int n, int a)
{
    if (n == 0)
        return a;

    return factorial(n - 1, a * n);
}

int main(void)
{
    for (int i = 0; i < 8; i++)
        printf("%d! == %ld\n", i, factorial(i, 1));
}
```

Để biến nó thành TCO, bạn có thể thay lời gọi bằng hai bước:

1. Set giá trị các tham số sang giá trị sẽ có ở lời gọi kế.
2. `goto` một label ở dòng đầu tiên của hàm.

Thử xem:

``` {.c .numberLines}
#include <stdio.h>

int factorial(int n, int a)
{
tco:  // add this

    if (n == 0)
        return a;

    // replace return by setting new parameter values and
    // goto-ing the beginning of the function

    //return factorial(n - 1, a * n);

    int next_n = n - 1;  // See how these match up with
    int next_a = a * n;  // the recursive arguments, above?

    n = next_n;   // Set the parameters to the new values
    a = next_a;

    goto tco;   // And repeat!
}

int main(void)
{
    for (int i = 0; i < 8; i++)
        printf("%d! == %d\n", i, factorial(i, 1));
}
```

Tôi đã dùng biến tạm phía trên để set giá trị kế của các tham số
trước khi nhảy về đầu hàm. Thấy chúng tương ứng với các đối số đệ
quy trong lời gọi đệ quy chưa?

Giờ, tại sao dùng biến tạm? Tôi có thể đã làm vầy thay thế:

``` {.c}
    a *= n;
    n -= 1;

    goto tco;
```

và cái đó thực tế chạy ổn. Nhưng nếu tôi bất cẩn đảo hai dòng code
đó:

``` {.c}
    n -= 1;  // BAD NEWS
    a *= n;
```

giờ ta gặp rắc rối. Ta sửa đổi `n` trước khi dùng nó để sửa `a`. Đó
là Tệ vì đó không phải cách nó chạy khi bạn gọi đệ quy. Dùng biến
tạm tránh được vấn đề này kể cả khi bạn không để ý. Và compiler khả
năng cao tối ưu chúng đi thôi.

[i[`goto` statement-->tail call optimzation]>]
[i[Tail call optimzation-->with `goto`]>]

## Khởi động lại system call bị ngắt

[i[`goto` statement-->restarting system calls]<]

Cái này nằm ngoài spec, nhưng thường thấy ở các hệ Unix-like.

Một số system call lâu có thể trả lỗi nếu bị ngắt bởi signal, và
`errno` sẽ được set thành `EINTR` để báo rằng syscall vẫn ổn, chỉ là
bị ngắt.

Trong các trường hợp đó, rất phổ biến việc lập trình viên muốn chạy
lại lời gọi và thử lại.

``` {.c}
retry:
    byte_count = read(0, buf, sizeof(buf) - 1);  // Unix read() syscall

    if (byte_count == -1) {            // An error occurred...
        if (errno == EINTR) {          // But it was just interrupted
            printf("Restarting...\n");
            goto retry;
        }
```

Nhiều hệ Unix-like có cờ `SA_RESTART` bạn có thể truyền cho
`sigaction()` để yêu cầu OS tự khởi động lại các syscall chậm thay
vì fail với `EINTR`.

Lại nữa, cái này đặc thù Unix và nằm ngoài chuẩn C.

Nói vậy, có thể dùng kỹ thuật tương tự bất cứ khi nào có hàm nào nên
được khởi động lại.

[i[`goto` statement-->restarting system calls]>]

## `goto` và preempt thread

[i[`goto` statement-->thread preemption]<]

Ví dụ này được lấy thẳng từ [_Operating Systems: Three Easy
Pieces_](http://www.ostep.org/), một cuốn sách tuyệt vời nữa từ các
tác giả cùng tư tưởng cũng cho rằng sách chất lượng nên được tải
miễn phí. Không phải tôi có quan điểm gì đâu.

``` {.c}
retry:

    pthread_mutex_lock(L1);

    if (pthread_mutex_trylock(L2) != 0) {
        pthread_mutex_unlock(L1);
        goto retry;
    }

    save_the_day();

    pthread_mutex_unlock(L2);
    pthread_mutex_unlock(L1);
```

Ở đó thread vui vẻ lấy được mutex `L1`, nhưng rồi tiềm năng fail khi
lấy tài nguyên thứ hai được bảo vệ bởi mutex `L2` (nếu một thread
khác không hợp tác đang giữ, chẳng hạn). Nếu thread của ta không lấy
được khoá `L2`, nó mở khoá `L1` rồi dùng `goto` để thử lại sạch sẽ.

Ta hy vọng thread anh hùng của ta rốt cuộc lấy được cả hai mutex và
cứu cả ngày, tránh được deadlock tà ác.

[i[`goto` statement-->thread preemption]>]

## `goto` và scope của biến

[i[`goto` statement-->variable scope]<]

Ta đã thấy label có function scope, nhưng chuyện lạ có thể xảy ra
nếu ta nhảy qua phần khởi tạo biến.

Xem ví dụ này nơi ta nhảy từ một chỗ mà biến `x` ngoài scope vào
giữa scope của nó (trong block).

``` {.c}
    goto label;

    {
        int x = 12345;

label:
        printf("%d\n", x);
    }
```

Cái này sẽ compile và chạy, nhưng cho tôi cảnh báo:

``` {.default}
warning: ‘x’ is used uninitialized in this function
```

Và rồi in ra `0` khi tôi chạy (kết quả có thể khác với bạn).

Về cơ bản chuyện đã xảy ra là ta nhảy vào scope của `x` (nên ok khi
tham chiếu nó trong `printf()`) nhưng ta nhảy qua dòng mà thực sự
khởi tạo nó thành `12345`. Nên giá trị không xác định.

Cách sửa dĩ nhiên là đưa phần khởi tạo ra _sau_ label theo cách nào
đó.

``` {.c}
    goto label;

    {
        int x;

label:
        x = 12345;
        printf("%d\n", x);
    }
```

Xem thêm một ví dụ nữa.

``` {.c}
    {
        int x = 10;

label:

        printf("%d\n", x);
    }

    goto label;
```

Chuyện gì xảy ra ở đây?

Lần đầu qua block, ta ngon. `x` là `10` và đó là cái được in.

Nhưng sau `goto`, ta nhảy vào scope của `x`, nhưng qua phần khởi tạo
của nó. Tức là ta vẫn có thể in nó, nhưng giá trị không xác định (vì
nó chưa được khởi tạo lại).

Trên máy tôi, nó in `10` lần nữa (mãi mãi), nhưng đó chỉ là may mắn.
Nó có thể in giá trị bất kỳ sau `goto` vì `x` không được khởi tạo.

[i[`goto` statement-->variable scope]>]

## `goto` và VLA

[i[`goto` statement-->with variable-length arrays]<]

Khi dính tới VLA và `goto`, có một quy tắc: bạn không thể nhảy từ
ngoài scope của một VLA vào trong scope của VLA đó.

Nếu tôi cố làm vầy:

``` {.c}
    int x = 10;

    goto label;

    {
        int v[x];

label:

        printf("Hi!\n");
    }
```

Tôi bị lỗi:

``` {.default}
error: jump into scope of identifier with variably modified type
```

Bạn có thể nhảy tới trước khai báo VLA, như vầy:

``` {.c}
    int x = 10;

    goto label;

    {
label:  ;
        int v[x];

        printf("Hi!\n");
    }
```

Vì cách đó VLA được cấp phát đúng cách trước khi chắc chắn bị giải
phóng khi ra khỏi scope.

[i[`goto` statement-->with variable-length arrays]>]
[i[`goto` statement]>]

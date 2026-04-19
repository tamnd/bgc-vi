<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Mảng {#arrays}

> _“Should array indices start at 0 or 1?  My compromise of 0.5 was
> rejected without, I thought, proper consideration.”_
>
> ---Stan Kelly-Bootle, computer scientist

[i[Arrays]<]
May thay, C có mảng. Ý tôi là, tôi biết nó được xem là ngôn ngữ cấp
thấp^[Ít ra là dạo này.] nhưng ít nhất nó có khái niệm mảng được tích
hợp sẵn. Và vì khá nhiều ngôn ngữ lấy cảm hứng cú pháp từ C, có lẽ bạn
đã quen với việc dùng `[` và `]` để khai báo và dùng mảng.

Nhưng C chỉ _vừa đủ_ có mảng thôi! Như ta sẽ thấy sau, mảng chỉ là
đường cú pháp (syntactic sugar) trong C, sâu thẳm bên trong chúng là
con trỏ và đủ thứ. _Hoảng lên đi!_ Nhưng bây giờ, cứ dùng chúng như
mảng đã. _Phù_.

## Ví dụ dễ

Xắn tay làm ví dụ luôn:

[i[Arrays-->indexing]<]
``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i;
    float f[4];  // Declare an array of 4 floats

    f[0] = 3.14159;  // Indexing starts at 0, of course.
    f[1] = 1.41421;
    f[2] = 1.61803;
    f[3] = 2.71828;

    // Print them all out:

    for (i = 0; i < 4; i++) {
        printf("%f\n", f[i]);
    }
}
```

Khi khai báo một mảng, bạn phải cho nó kích thước. Và kích thước phải
cố định^[Lại nữa, không hẳn, nhưng variable-length array, thứ mà tôi
không khoái lắm, là chuyện để dành dịp khác.].

Trong ví dụ trên, ta tạo một mảng 4 `float`. Giá trị trong dấu ngoặc
vuông ở khai báo cho ta biết điều đó.

Ở các dòng sau, ta truy cập các giá trị trong mảng, gán hoặc đọc,
lại bằng dấu ngoặc vuông.
[i[Arrays-->indexing]>]

Hy vọng cái này quen thuộc từ các ngôn ngữ bạn đã biết!

## Lấy chiều dài của mảng

[i[Arrays-->getting the length]<]
Bạn không thể... ở mức độ nào đó. C không ghi lại thông tin này^[Vì
mảng, sâu bên dưới, chỉ là con trỏ tới phần tử đầu tiên, không có
thông tin bổ sung nào ghi lại chiều dài.]. Bạn phải quản lý riêng
trong một biến khác.

Khi tôi nói "không thể", thật ra có một số hoàn cảnh bạn _có thể_. Có
một mẹo để lấy số phần tử của mảng trong scope mà mảng được khai báo.
Nhưng, nói chung, nó sẽ không hoạt động như bạn mong muốn nếu bạn
truyền mảng cho hàm^[Vì khi bạn truyền mảng cho hàm, thật ra bạn chỉ
đang truyền một con trỏ tới phần tử đầu tiên của mảng, chứ không phải
"toàn bộ" mảng.].

Xem cái mẹo này. Ý tưởng cơ bản là bạn lấy
[i[`sizeof` operator-->with arrays]<]`sizeof` mảng, rồi chia cho kích
thước của mỗi phần tử để ra chiều dài. Ví dụ, nếu một `int` là 4 byte,
và mảng dài 32 byte, vậy chắc chắn có chỗ cho $\frac{32}{4}$ hay $8$
`int` trong đó.

``` {.c}
int x[12];  // 12 ints

printf("%zu\n", sizeof x);     // 48 total bytes
printf("%zu\n", sizeof(int));  // 4 bytes per int

printf("%zu\n", sizeof x / sizeof(int));  // 48/4 = 12 ints!
```

Nếu là mảng `char`, thì `sizeof` mảng _chính là_ số phần tử, vì
`sizeof(char)` được định nghĩa là 1. Với bất cứ thứ gì khác, bạn phải
chia cho kích thước của mỗi phần tử.

Nhưng mẹo này chỉ hoạt động trong scope mà mảng được định nghĩa. Nếu
bạn truyền mảng cho hàm, nó không hoạt động. Ngay cả khi bạn làm cho
nó "to" trong signature của hàm:

``` {.c}
void foo(int x[12])
{
    printf("%zu\n", sizeof x);     // 8?! What happened to 48?
    printf("%zu\n", sizeof(int));  // 4 bytes per int

    printf("%zu\n", sizeof x / sizeof(int));  // 8/4 = 2 ints?? WRONG.
}
```

Đó là vì khi bạn "truyền" mảng cho hàm, bạn chỉ truyền một con trỏ tới
phần tử đầu tiên, và đó là thứ `sizeof` đo. Sẽ nói thêm ở mục
[Passing Single Dimensional Arrays to Functions](#passing1darrays) phía
dưới.

Một thứ nữa bạn có thể làm với `sizeof` và mảng là lấy kích thước của
một mảng có số phần tử cố định mà không cần khai báo mảng. Giống như
cách bạn lấy kích thước của `int` bằng `sizeof(int)`.

Ví dụ, để xem cần bao nhiêu byte cho một mảng 48 `double`, bạn có thể
làm:

``` {.c}
sizeof(double [48]);
```
[i[`sizeof` operator-->with arrays]>]
[i[Arrays-->getting the length]>]

## Khởi tạo mảng

[i[Array initializers]<]
Bạn có thể khởi tạo mảng bằng hằng số từ trước:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i;
    int a[5] = {22, 37, 3490, 18, 95};  // Initialize with these values

    for (i = 0; i < 5; i++) {
        printf("%d\n", a[i]);
    }
}
```

Đừng bao giờ để nhiều phần tử trong initializer hơn mức mảng chứa
được, không thì trình biên dịch sẽ khó chịu:

``` {.zsh}
foo.c: In function ‘main’:
foo.c:6:39: warning: excess elements in array initializer
    6 |     int a[5] = {22, 37, 3490, 18, 95, 999};
      |                                       ^~~
foo.c:6:39: note: (near initialization for ‘a’)
```

Nhưng (vui đây!) bạn có thể để _ít_ phần tử trong initializer hơn mức
mảng có. Các phần tử còn lại trong mảng sẽ được tự động khởi tạo bằng
zero. Điều này đúng với tất cả các dạng khởi tạo mảng: nếu bạn có
initializer, bất cứ thứ gì không được gán giá trị rõ ràng sẽ được đặt
thành zero.

``` {.c}
int a[5] = {22, 37, 3490};

// is the same as:

int a[5] = {22, 37, 3490, 0, 0};
```

Có một mẹo tắt phổ biến bạn hay thấy trong initializer khi muốn đặt
toàn bộ mảng về zero:

``` {.c}
int a[100] = {0};
```

Nghĩa là, "Đặt phần tử đầu tiên thành zero, rồi tự động đặt phần còn
lại cũng thành zero."

Bạn cũng có thể đặt phần tử cụ thể của mảng trong initializer, bằng
cách chỉ định chỉ số cho giá trị! Khi làm thế, C sẽ vui vẻ tiếp tục
khởi tạo các giá trị kế sau cho bạn đến khi initializer cạn, lấp phần
còn lại bằng `0`.

Để làm vậy, đặt chỉ số trong ngoặc vuông với `=` theo sau, rồi đặt
giá trị.

Đây là ví dụ ta dựng một mảng:

``` {.c}
int a[10] = {0, 11, 22, [5]=55, 66, 77};
```

Vì ta liệt kê chỉ số 5 là điểm bắt đầu cho `55`, dữ liệu kết quả trong
mảng là:

``` {.default}
0 11 22 0 0 55 66 77 0 0
```

Bạn cũng có thể đặt biểu thức hằng số đơn giản vào đó.

``` {.c}
#define COUNT 5

int a[COUNT] = {[COUNT-3]=3, 2, 1};
```

cho ra:

``` {.default}
0 0 3 2 1
```

Cuối cùng, bạn cũng có thể để C tự tính kích thước mảng từ
initializer, bằng cách bỏ trống kích thước:

``` {.c}
int a[3] = {22, 37, 3490};

// is the same as:

int a[] = {22, 37, 3490};  // Left the size off!
```
[i[Array initializers]>]

## Vượt biên!

[i[Arrays-->out of bounds]<]
C không ngăn bạn truy cập mảng vượt biên. Có khi còn không cảnh báo
luôn.

Chôm ví dụ phía trên và cứ thế in vượt qua cuối mảng. Nó chỉ có 5 phần
tử, nhưng cứ thử in 10 xem chuyện gì xảy ra:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i;
    int a[5] = {22, 37, 3490, 18, 95};

    for (i = 0; i < 10; i++) {  // BAD NEWS: printing too many elements!
        printf("%d\n", a[i]);
    }
}
```

Chạy trên máy tính của tôi, nó in:

``` {.default}
22
37
3490
18
95
32765
1847052032
1780534144
-56487472
21890
```

Hú hồn! Cái gì thế? Hoá ra in vượt qua cuối mảng dẫn đến thứ mà dân C
gọi là _undefined behavior_ (hành vi không xác định). Ta sẽ nói thêm
về con quái này sau, nhưng hiện giờ nó nghĩa là, "Bạn đã làm điều gì
đó xấu, và bất cứ điều gì cũng có thể xảy ra trong lúc chương trình
chạy."

Và "bất cứ điều gì", thường là như tìm thấy zero, tìm thấy số rác, hay
crash. Nhưng thật ra spec C nói trong trường hợp này trình biên dịch
được phép sinh code làm _bất cứ thứ gì_^[Trong những ngày xưa tốt đẹp
của MS-DOS trước khi memory protection ra đời, tôi đang viết một số
code C đặc biệt bạo lực, cố tình đâm đầu vào đủ loại undefined
behavior. Nhưng tôi biết mình đang làm gì, và mọi thứ vẫn chạy khá
tốt. Cho tới khi tôi lỡ bước làm treo máy và, sau khi reboot, phát
hiện toàn bộ cài đặt BIOS đã bị xoá sạch. Vui đáo để. (Gửi lời tới
@man vì những lúc vui đó.)].

Phiên bản ngắn: đừng làm bất cứ thứ gì gây ra undefined behavior. Bao
giờ^[Có rất nhiều thứ gây undefined behavior, không chỉ truy cập mảng
vượt biên. Đây chính là thứ làm ngôn ngữ C trở nên _sôi động_.].
[i[Arrays-->out of bounds]>]

## Mảng nhiều chiều

[i[Arrays-->multidimensional]<]
Bạn có thể thêm bao nhiêu chiều tuỳ thích cho mảng.

``` {.c}
int a[10];
int b[2][7];
int c[4][5][6];
```

Chúng được lưu trong bộ nhớ theo thứ tự [flw[row-major
order|Row-_and_column-major_order]]. Nghĩa là với mảng 2D, chỉ số
đầu tiên được liệt kê chỉ hàng, chỉ số thứ hai chỉ cột.

Bạn cũng có thể dùng initializer trên mảng nhiều chiều bằng cách lồng
chúng:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int row, col;

    int a[2][5] = {      // Initialize a 2D array
        {0, 1, 2, 3, 4},
        {5, 6, 7, 8, 9}
    };

    for (row = 0; row < 2; row++) {
        for (col = 0; col < 5; col++) {
            printf("(%d,%d) = %d\n", row, col, a[row][col]);
        }
    }
}
```

Cho output:

``` {.default}
(0,0) = 0
(0,1) = 1
(0,2) = 2
(0,3) = 3
(0,4) = 4
(1,0) = 5
(1,1) = 6
(1,2) = 7
(1,3) = 8
(1,4) = 9
```

Và bạn có thể khởi tạo với chỉ số rõ ràng:

``` {.c}
// Make a 3x3 identity matrix

int a[3][3] = {[0][0]=1, [1][1]=1, [2][2]=1};
```

sẽ dựng một mảng 2D như này:

``` {.default}
1 0 0
0 1 0
0 0 1
```
[i[Arrays-->multidimensional]>]

## Mảng và con trỏ

[i[Arrays-->as pointers]<]
[_Thản nhiên_] Thì... tôi có lẽ đã đề cập phía trên rằng sâu thẳm bên
trong mảng là con trỏ nhỉ? Giờ ta nên làm một cú lặn cạn vào chuyện
đó, để mọi thứ không bị hoàn toàn rối rắm. Sau này, ta sẽ nhìn kỹ mối
quan hệ thực sự giữa mảng và con trỏ, nhưng bây giờ tôi chỉ muốn xem
chuyện truyền mảng cho hàm.

### Lấy con trỏ tới một mảng

Tôi muốn kể bạn nghe một bí mật. Nói chung, khi một lập trình viên C
nói về con trỏ tới một mảng, họ đang nói về con trỏ _tới phần tử đầu
tiên_ của mảng^[Về kỹ thuật thì không chính xác, vì con trỏ tới một
mảng và con trỏ tới phần tử đầu tiên của mảng có kiểu khác nhau.
Nhưng ta sẽ đốt cầu khi đến đó.].

Nào, lấy một con trỏ tới phần tử đầu tiên của mảng.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int a[5] = {11, 22, 33, 44, 55};
    int *p;

    p = &a[0];  // p points to the array
                // Well, to the first element, actually

    printf("%d\n", *p);  // Prints "11"
}
```

Chuyện này quá phổ biến trong C, đến nỗi ngôn ngữ cho phép ta một cách
viết tắt:

``` {.c .numberLines}
p = &a[0];  // p points to the array

// is the same as:

p = a;      // p points to the array, but much nicer-looking!
```

Chỉ cần nhắc đến tên mảng đứng một mình là tương đương với lấy một con
trỏ tới phần tử đầu tiên của mảng! Ta sẽ dùng điều này rộng rãi trong
các ví dụ sắp tới.

Nhưng khoan đã, `p` chẳng phải là `int*` sao? Và `*p` cho ta `11`,
cùng giống `a[0]`? Đúúúúng. Bạn đang bắt đầu thấy thoáng qua mối quan
hệ giữa mảng và con trỏ trong C. (Ta sẽ nói nhiều hơn về chuyện này
trong chương [Pointers II](#pointers2), ở mục [Array/Pointer
Equivalence](#arraypointerequiv).)
[i[Arrays-->as pointers]>]

### Truyền mảng một chiều cho hàm {#passing1darrays}

[i[Arrays-->passing to functions]<]
Làm một ví dụ với mảng một chiều. Tôi sẽ viết vài hàm mà ta có thể
truyền mảng vào để làm việc khác nhau.

Chuẩn bị cho vài signature hàm làm đầu óc bùng nổ!

``` {.c .numberLines}
#include <stdio.h>

// Passing as a pointer to the first element
void times2(int *a, int len)
{
    for (int i = 0; i < len; i++)
        printf("%d\n", a[i] * 2);
}

// Same thing, but using array notation
void times3(int a[], int len)
{
    for (int i = 0; i < len; i++)
        printf("%d\n", a[i] * 3);
}

// Same thing, but using array notation with size
void times4(int a[5], int len)
{
    for (int i = 0; i < len; i++)
        printf("%d\n", a[i] * 4);
}

int main(void)
{
    int x[5] = {11, 22, 33, 44, 55};

    times2(x, 5);
    times3(x, 5);
    times4(x, 5);
}
```

Tất cả các cách liệt kê mảng làm parameter trong hàm đó đều giống hệt
nhau.

``` {.c}
void times2(int *a, int len)
void times3(int a[], int len)
void times4(int a[5], int len)
```

Trong cách dùng của dân C quen tay, cách đầu tiên phổ biến nhất, bỏ xa
các cách còn lại.

Và, thật ra, ở tình huống cuối, trình biên dịch thậm chí không quan
tâm bạn truyền số nào (miễn lớn hơn không^[C11 §6.7.6.2¶1 yêu cầu nó
phải lớn hơn không. Nhưng bạn có thể thấy code ngoài đời với mảng
chiều dài bằng không ở cuối các `struct`, và GCC đặc biệt dễ dãi
chuyện đó trừ khi bạn biên dịch với `-pedantic`. Mảng chiều dài bằng
không là một cơ chế kiểu hack để tạo struct có chiều dài biến đổi.
Không may, về kỹ thuật truy cập mảng như vậy là undefined behavior,
dù nó gần như chạy được ở mọi nơi. C99 chuẩn hoá một phương án thay
thế được định nghĩa rõ gọi là _flexible array members_, ta sẽ tán
gẫu chuyện đó sau.]). Nó không ép buộc gì cả.

Giờ sau khi đã nói vậy, kích thước mảng trong khai báo hàm thật ra
_có_ ý nghĩa khi bạn truyền mảng nhiều chiều vào hàm, nhưng hãy quay
lại chuyện đó sau.
[i[Arrays-->passing to functions]>]

### Thay đổi mảng trong hàm

[i[Arrays-->modifying within functions]<]
Ta đã nói mảng chỉ là con trỏ trá hình. Nghĩa là nếu bạn truyền mảng
cho hàm, nhiều khả năng bạn đang truyền con trỏ tới phần tử đầu tiên
của mảng.

Nhưng nếu hàm có con trỏ tới dữ liệu, nó có thể thao tác trên dữ liệu
đó! Vậy nên những thay đổi mà hàm làm với mảng sẽ thấy được ở phía
người gọi.

Đây là ví dụ ta truyền một con trỏ tới mảng cho hàm, hàm thao tác trên
các giá trị trong mảng đó, và các thay đổi đó thấy được ở phía người
gọi.

``` {.c .numberLines}
#include <stdio.h>

void double_array(int *a, int len)
{
    // Multiply each element by 2
    //
    // This doubles the values in x in main() since x and a both point
    // to the same array in memory!

    for (int i = 0; i < len; i++)
        a[i] *= 2;
}

int main(void)
{
    int x[5] = {1, 2, 3, 4, 5};

    double_array(x, 5);

    for (int i = 0; i < 5; i++)
        printf("%d\n", x[i]);  // 2, 4, 6, 8, 10!
}
```

Mặc dù ta truyền mảng vào parameter `a` kiểu `int*`, nhìn cách ta truy
cập nó bằng ký pháp mảng `a[i]`! Cááááái gì. Chuyện này hoàn toàn được
cho phép.

Sau này khi nói về sự tương đương giữa mảng và con trỏ, ta sẽ thấy
chuyện này hợp lý hơn nhiều. Giờ, đủ để biết rằng hàm có thể thay đổi
mảng mà thấy được ở phía người gọi.
[i[Arrays-->modifying within functions]>]

### Truyền mảng nhiều chiều cho hàm

[i[Arrays-->passing to functions]<]
Câu chuyện thay đổi một chút khi ta nói về mảng nhiều chiều. C cần
biết tất cả các chiều (trừ chiều đầu tiên) để có đủ thông tin biết tìm
giá trị ở đâu trong bộ nhớ.

Đây là ví dụ ta nói rõ tất cả các chiều:

``` {.c .numberLines}
#include <stdio.h>

void print_2D_array(int a[2][3])
{
    for (int row = 0; row < 2; row++) {
        for (int col = 0; col < 3; col++)
            printf("%d ", a[row][col]);
        printf("\n");
    }
}

int main(void)
{
    int x[2][3] = {
        {1, 2, 3},
        {4, 5, 6}
    };

    print_2D_array(x);
}
```

Nhưng trong trường hợp này, hai cách này^[Cách này cũng tương đương:
`void print_2D_array(int (*a)[3])`, nhưng đó là chuyện tôi không muốn
đi sâu ngay lúc này.] là tương đương:

``` {.c}
void print_2D_array(int a[2][3])
void print_2D_array(int a[][3])
```

Trình biên dịch thật ra chỉ cần chiều thứ hai để biết cần nhảy bao xa
trong bộ nhớ cho mỗi lần tăng chiều đầu tiên. Nói chung, nó cần biết
tất cả các chiều trừ chiều đầu tiên.

Ngoài ra, nhớ rằng trình biên dịch chỉ làm kiểm tra biên tối thiểu
lúc biên dịch (nếu bạn may mắn), và C không kiểm tra biên gì hết lúc
chạy. Không dây an toàn! Đừng crash bằng cách truy cập phần tử mảng
vượt biên!
[i[Arrays-->passing to functions]>] [i[Arrays]>]

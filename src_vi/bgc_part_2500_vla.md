<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Mảng độ dài biến đổi (VLA)

[i[Variable-length array]<]

C cung cấp cách khai báo mảng mà kích thước được xác định lúc chạy.
Cái này cho bạn lợi ích của việc chỉnh kích thước động lúc runtime
như với `malloc()`, nhưng không cần lo `free()` bộ nhớ sau đó.

Giờ, nhiều người không thích VLA. Chúng bị cấm trong Linux kernel
chẳng hạn. Ta sẽ đào sâu hơn về lý do đó [ở sau](#vla-general-issues).

[i[`__STDC_NO_VLA__` macro]<]

Đây là tính năng tuỳ chọn của ngôn ngữ. Macro `__STDC_NO_VLA__` được
set là `1` nếu VLA _không_ có. (Chúng bắt buộc trong C99, rồi thành
tuỳ chọn trong C11.)

``` {.c}
#if __STDC_NO_VLA__ == 1
   #error Sorry, need VLAs for this program!
#endif
```

[i[`__STDC_NO_VLA__` macro]>]

Nhưng vì cả GCC và Clang đều không buồn định nghĩa macro này, bạn có
thể chẳng đi được mấy với nó.

Nhảy vào với một ví dụ trước, rồi ta sẽ đi tìm con quỷ trong chi
tiết.

## Cơ bản

Một mảng thường được khai báo với kích thước hằng, như sau:

``` {.c}
int v[10];
```

[i[Variable-length array-->defining]<]

Nhưng với VLA, ta có thể dùng kích thước xác định lúc runtime để đặt
mảng, như sau:

``` {.c}
int n = 10;
int v[n];
```

Giờ, trông thì giống y như nhau, và ở nhiều mặt nó giống thật, nhưng
cái này cho bạn sự linh hoạt để tính kích thước cần, rồi lấy một
mảng chính xác kích thước đó.

Ta hãy hỏi người dùng nhập kích thước mảng, rồi lưu chỉ-số-nhân-10
vào mỗi phần tử mảng:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int n;
    char buf[32];

    printf("Enter a number: "); fflush(stdout);
    fgets(buf, sizeof buf, stdin);
    n = strtoul(buf, NULL, 10);

    int v[n];

    for (int i = 0; i < n; i++)
        v[i] = i * 10;

    for (int i = 0; i < n; i++)
        printf("v[%d] = %d\n", i, v[i]);
}
```

(Ở dòng 7, tôi có `fflush()` để ép dòng được xuất ra dù tôi không có
newline ở cuối.)

Dòng 12 là chỗ ta khai báo VLA, một khi thực thi đi qua dòng đó,
kích thước mảng được set bằng bất cứ giá trị nào `n` có tại thời
điểm đó. Độ dài mảng không thể đổi sau này.

Bạn có thể đặt biểu thức trong ngoặc vuông cũng được:

``` {.c}
int v[x * 100];
```

Vài hạn chế:

* Bạn không thể khai báo VLA ở file scope, và không thể tạo một VLA
  `static` trong block scope^[Đây là do VLA thường được cấp phát
  trên stack, còn biến `static` nằm trên heap. Và cả ý tưởng của VLA
  là chúng sẽ được giải phóng tự động khi stack frame bị pop ở cuối
  hàm.].
* Bạn không thể dùng initializer list để khởi tạo mảng.

Ngoài ra, nhập giá trị âm cho kích thước mảng sẽ gây hành vi không
xác định, dù sao thì cũng trong vũ trụ này.

[i[Variable-length array-->defining]>]

## `sizeof` và VLA

[i[Variable-length array-->and `sizeof()`]<]

Ta đã quen với việc `sizeof` cho ra kích thước tính bằng byte của
một object cụ thể, kể cả mảng. Và VLA cũng không ngoại lệ.

Khác biệt chính là `sizeof` trên VLA được chạy lúc _runtime_, còn
trên biến không có kích thước biến đổi thì được tính lúc _compile
time_.

Nhưng cách dùng vẫn vậy.

Bạn thậm chí có thể tính số phần tử trong VLA bằng trò mảng quen
thuộc:

``` {.c}
size_t num_elems = sizeof v / sizeof v[0];
```

Có một hàm ý tinh tế và đúng từ dòng trên: số học con trỏ chạy y như
bạn kỳ vọng với mảng thường. Nên cứ dùng thoả thích:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int n = 5;
    int v[n];

    int *p = v;

    *(p+2) = 12;
    printf("%d\n", v[2]);  // 12

    p[3] = 34;
    printf("%d\n", v[3]);  // 34
}
```

Giống như với mảng thường, bạn có thể dùng ngoặc với `sizeof()` để
lấy kích thước của một VLA giả-định mà không thực sự khai báo nó:

``` {.c}
int x = 12;

printf("%zu\n", sizeof(int [x]));  // Prints 48 on my system
```

[i[Variable-length array-->and `sizeof()`]>]

## VLA nhiều chiều

[i[Variable-length array-->multidimensional]<]

Bạn có thể tạo đủ kiểu VLA với một hoặc nhiều chiều được set làm
biến

``` {.c}
int w = 10;
int h = 20;

int x[h][w];
int y[5][w];
int z[10][w][20];
```

Lại nữa, bạn có thể điều hướng chúng y như mảng thường.

[i[Variable-length array-->multidimensional]>]

## Truyền VLA một chiều cho hàm

[i[Variable-length array-->passing to functions]<]

Truyền VLA đơn-chiều vào hàm không khác gì truyền mảng thường. Cứ
thế mà làm.

``` {.c .numberLines}
#include <stdio.h>

int sum(int count, int *v)
{
    int total = 0;

    for (int i = 0; i < count; i++)
        total += v[i];

    return total;
}

int main(void)
{
    int x[5];   // Standard array

    int a = 5;
    int y[a];   // VLA

    for (int i = 0; i < a; i++)
        x[i] = y[i] = i + 1;

    printf("%d\n", sum(5, x));
    printf("%d\n", sum(a, y));
}
```

Nhưng còn có thêm chút nữa. Bạn cũng có thể cho C biết rằng mảng là
VLA cụ thể kích thước nào đó bằng cách truyền kích thước đó trước
rồi ghi chiều đó vào danh sách tham số:

``` {.c}
int sum(int count, int v[count])
{
    // ...
}
```

[i[Variable-length array-->in function prototypes]<]
[i[`*` for VLA function prototypes]<]

Nhân tiện, có vài cách liệt kê prototype cho hàm trên; một trong số
đó dùng `*` nếu bạn không muốn chỉ cụ thể tên biến giữ giá trị trong
VLA. Nó chỉ báo rằng kiểu là VLA chứ không phải con trỏ thường.

Prototype VLA:

``` {.c}
void do_something(int count, int v[count]);  // With names
void do_something(int, int v[*]);            // Without names
```

Lại nữa, cái `*` đó chỉ dùng với prototype, trong thân hàm bạn sẽ
phải đặt kích thước tường minh.

[i[`*` for VLA function prototypes]>]
[i[Variable-length array-->in function prototypes]>]

Giờ, _đa chiều thôi_! Đây là chỗ vui bắt đầu.

## Truyền VLA đa chiều cho hàm

Y như ta đã làm với dạng thứ hai của VLA một chiều ở trên, nhưng lần
này ta truyền vào hai chiều và dùng chúng.

Trong ví dụ sau, ta dựng một ma trận bảng cửu chương chiều rộng và
chiều cao biến đổi, rồi truyền cho một hàm để in ra.

``` {.c .numberLines}
#include <stdio.h>

void print_matrix(int h, int w, int m[h][w])
{
    for (int row = 0; row < h; row++) {
        for (int col = 0; col < w; col++)
            printf("%2d ", m[row][col]);
        printf("\n");
    }
}

int main(void)
{
    int rows = 4;
    int cols = 7;

    int matrix[rows][cols];

    for (int row = 0; row < rows; row++)
        for (int col = 0; col < cols; col++)
            matrix[row][col] = row * col;

    print_matrix(rows, cols, matrix);
}
```

### VLA đa chiều một phần

Bạn có thể có một số chiều cố định và một số biến đổi. Giả sử ta có
một bản ghi độ dài cố định 5 phần tử, nhưng ta không biết có bao
nhiêu bản ghi.

``` {.c .numberLines}
#include <stdio.h>

void print_records(int count, int record[count][5])
{
    for (int i = 0; i < count; i++) {
        for (int j = 0; j < 5; j++)
            printf("%2d ", record[i][j]);
        printf("\n");
    }
}

int main(void)
{
    int rec_count = 3;
    int records[rec_count][5];

    // Fill with some dummy data
    for (int i = 0; i < rec_count; i++)
        for (int j = 0; j < 5; j++)
            records[i][j] = (i+1)*(j+2);

    print_records(rec_count, records);
}
```

[i[Variable-length array-->passing to functions]>]

## Tương thích với mảng thường

[i[Variable-length array-->with regular arrays]<]

Vì VLA y như mảng thường trong bộ nhớ, hoàn toàn cho phép truyền
chúng lẫn nhau... miễn là các chiều khớp.

Ví dụ, nếu ta có một hàm muốn mảng $3\times5$ cụ thể, ta vẫn có thể
truyền một VLA vào đó.

``` {.c}
int foo(int m[5][3]) {...}

\\ ...

int w = 3, h = 5;
int matrix[h][w];

foo(matrix);   // OK!
```

Tương tự, nếu bạn có một hàm VLA, bạn có thể truyền mảng thường vào:

``` {.c}
int foo(int h, int w, int m[h][w]) {...}

\\ ...

int matrix[3][5];

foo(3, 5, matrix);   // OK!
```

Coi chừng nhé: nếu chiều không khớp, bạn sẽ có hành vi không xác
định, rất có khả năng.

[i[Variable-length array-->with regular arrays]>]

## `typedef` và VLA

[i[Variable-length array-->with `typedef`]<]

Bạn có thể `typedef` một VLA, nhưng hành vi có thể không như bạn
mong đợi.

Về cơ bản, `typedef` tạo một kiểu mới với các giá trị như chúng tồn
tại tại thời điểm `typedef` được chạy.

Nên nó không hẳn là một `typedef` của VLA mà là một kiểu mảng kích
thước cố định mới với các chiều tại thời điểm đó.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int w = 10;

    typedef int goat[w];

    // goat is an array of 10 ints
    goat x;

    // Init with squares of numbers
    for (int i = 0; i < w; i++)
        x[i] = i*i;

    // Print them
    for (int i = 0; i < w; i++)
        printf("%d\n", x[i]);

    // Now let's change w...

    w = 20;

    // But goat is STILL an array of 10 ints, because that was the
    // value of w when the typedef executed.
}
```

Nên nó hành xử như một mảng kích thước cố định.

Nhưng bạn vẫn không thể dùng initializer list trên nó.

[i[Variable-length array-->with `typedef`]>]

## Bẫy nhảy lung tung

[i[Variable-length array-->with `goto`]<]

Bạn phải coi chừng khi dùng `goto` gần VLA vì nhiều thứ không hợp
lệ.

[i[Variable-length array-->with `goto`]>]
[i[Variable-length array-->with `longjmp()`]<]

Và khi bạn dùng `longjmp()` có trường hợp bạn có thể leak bộ nhớ với
VLA.

[i[Variable-length array-->with `longjmp()`]>]

Nhưng cả hai thứ này ta sẽ bàn trong chương riêng của chúng.


## Vấn đề chung {#vla-general-issues}

[i[Variable-length array-->controversy]<]

VLA đã bị cấm khỏi Linux kernel vì vài lý do:

* Nhiều chỗ chúng được dùng lẽ ra nên là kích thước cố định.
* Code đằng sau VLA chậm hơn (tới mức mà đa số không để ý, nhưng tạo
  khác biệt trong hệ điều hành).
* VLA không được hỗ trợ đồng đều bởi mọi trình biên dịch C.
* Kích thước stack bị giới hạn, và VLA nằm trên stack. Nếu đoạn code
  nào đó vô tình (hoặc ác ý) truyền giá trị lớn vào một hàm kernel
  cấp phát VLA, _Chuyện Xấu_™ có thể xảy ra.

Nhiều người khác online chỉ ra rằng không có cách nào phát hiện VLA
thất bại khi cấp phát, và chương trình dính vấn đề như vậy khả năng
chỉ có crash. Dù mảng kích thước cố định cũng có vấn đề y vậy, khả
năng cao hơn nhiều là ai đó lỡ tay làm _VLA Kích Thước Bất Thường_
hơn là ai đó vô tình khai báo một mảng cố định ví dụ 30 megabyte.

[i[Variable-length array-->controversy]>]
[i[Variable-length array]>]

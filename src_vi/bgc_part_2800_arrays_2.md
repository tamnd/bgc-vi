<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Mảng Phần II

Chương này ta sẽ đi qua vài thứ linh tinh thêm về mảng.

* Type qualifier với tham số mảng
* Từ khoá `static` với tham số mảng
* Initializer một phần cho mảng đa chiều

Chúng không phải cái hay thấy, nhưng ta sẽ liếc qua vì chúng là một
phần của spec mới hơn.

## Type qualifier cho mảng trong danh sách tham số

[i[Type qualifiers-->arrays in parameter lists]<]
[i[Arrays-->type qualifiers in parameter lists]<]

Nếu bạn nhớ từ trước, hai thứ này tương đương trong danh sách tham
số hàm:

``` {.c}
int func(int *p) {...}
int func(int p[]) {...}
```

Và bạn cũng có thể nhớ rằng bạn có thể thêm type qualifier vào biến
con trỏ như vầy:

``` {.c}
int *const p;
int *volatile p;
int *const volatile p;
// etc.
```

Nhưng làm sao làm được chuyện đó khi ta dùng ký pháp mảng trong danh
sách tham số?

Hoá ra nó đi vào trong ngoặc vuông. Và bạn có thể đặt count tuỳ chọn
ở sau. Hai dòng sau tương đương:

``` {.c}
int func(int *const volatile p) {...}
int func(int p[const volatile]) {...}
int func(int p[const volatile 10]) {...}
```

Nếu bạn có mảng đa chiều, bạn cần đặt type qualifier ở bộ ngoặc
vuông đầu.

[i[Type qualifiers-->arrays in parameter lists]>]
[i[Arrays-->type qualifiers in parameter lists]>]

## `static` cho mảng trong danh sách tham số

[i[Arrays-->`static` in parameter lists]<]

Tương tự, bạn có thể dùng từ khoá static trong mảng trong danh sách
tham số.

Đây là thứ tôi chưa từng thấy ngoài đời. Nó **luôn** theo sau bởi
một kích thước:

``` {.c}
int func(int p[static 4]) {...}
```

Điều này có nghĩa, trong ví dụ trên, compiler sẽ giả định mọi mảng
bạn truyền cho hàm sẽ có _ít nhất_ 4 phần tử.

Bất cứ gì khác là hành vi không xác định.

``` {.c}
int func(int p[static 4]) {...}

int main(void)
{
    int a[] = {11, 22, 33, 44};
    int b[] = {11, 22, 33, 44, 55};
    int c[] = {11, 22};

    func(a);  // OK! a is 4 elements, the minimum
    func(b);  // OK! b is at least 4 elements
    func(c);  // Undefined behavior! c is under 4 elements!
}
```

Cái này về cơ bản đặt kích thước tối thiểu của mảng bạn có thể có.

Lưu ý quan trọng: không có gì trong compiler cấm bạn truyền mảng nhỏ
hơn. Compiler khả năng sẽ không cảnh báo bạn, và nó cũng không phát
hiện lúc runtime.

Khi đặt `static` vào đó, bạn đang nói, "Tôi hứa danh dự gấp đôi là
tôi không bao giờ truyền vào mảng nhỏ hơn cái này." Và compiler nói,
"Ừ, được rồi," và tin bạn sẽ không làm thế.

Và rồi compiler có thể thực hiện một số tối ưu code nhất định, yên
tâm rằng bạn, lập trình viên, sẽ luôn làm đúng.

[i[Arrays-->`static` in parameter lists]>]

## Các initializer tương đương

[i[Arrays-->multidimensional initializers]<]

C hơi, nói thế nào nhỉ, _linh hoạt_ khi dính tới initializer của
mảng.

Ta đã thấy một chút rồi, khi giá trị thiếu được thay bằng zero.

Ví dụ, ta có thể khởi tạo mảng 5 phần tử thành `1,2,0,0,0` với:

``` {.c}
int a[5] = {1, 2};
```

Hoặc set cả mảng về zero với:

``` {.c}
int a[5] = {0};
```

Nhưng chuyện thú vị bắt đầu khi khởi tạo mảng đa chiều.

Làm một mảng 3 hàng, 2 cột:

``` {.c}
int a[3][2];
```

Viết chút code để khởi tạo và in kết quả:

``` {.c}
#include <stdio.h>

int main(void)
{
    int a[3][2] = {
        {1, 2},
        {3, 4},
        {5, 6}
    };

    for (int row = 0; row < 3; row++) {
        for (int col = 0; col < 2; col++)
            printf("%d ", a[row][col]);
        printf("\n");
    }
}
```

Và khi chạy, ta có kết quả như kỳ vọng:

``` {.default}
1 2
3 4
5 6
```

Hãy bỏ bớt vài phần tử initializer và xem chúng được set về zero:


``` {.c}
    int a[3][2] = {
        {1, 2},
        {3},    // Left off the 4!
        {5, 6}
    };
```

cho ra:

``` {.default}
1 2
3 0
5 6
```

Giờ bỏ cả phần tử giữa cuối cùng:

``` {.c}
    int a[3][2] = {
        {1, 2},
        // {3, 4},   // Just cut this whole thing out
        {5, 6}
    };
```

Và giờ ta có cái này, có thể không như bạn nghĩ:

``` {.default}
1 2
5 6
0 0
```

Nhưng nếu bạn dừng lại suy nghĩ, ta chỉ cung cấp đủ initializer cho
hai hàng, nên chúng được dùng cho hai hàng đầu. Và các phần tử còn
lại được khởi tạo thành zero.

Đến đây ổn. Nhìn chung, nếu ta bỏ bớt phần của initializer, compiler
set các phần tử tương ứng thành `0`.

Nhưng hãy làm _điên_ hơn.

``` {.c}
    int a[3][2] = { 1, 2, 3, 4, 5, 6 };
```

Cái gì---? Đó là mảng 2D, nhưng chỉ có initializer 1D!

Hoá ra chuyện đó hợp lệ (dù GCC sẽ cảnh báo nếu bật đúng warning).

Về cơ bản, nó bắt đầu điền phần tử ở hàng 0, rồi hàng 1, rồi hàng 2
từ trái sang phải.

Nên khi ta in, nó in theo thứ tự:

``` {.default}
1 2
3 4
5 6
```

Nếu ta bỏ vài cái:

``` {.c}
    int a[3][2] = { 1, 2, 3 };
```

chúng được điền `0`:

``` {.default}
1 2
3 0
0 0
```

Nên nếu bạn muốn điền cả mảng bằng `0`, cứ:

``` {.c}
    int a[3][2] = {0};
```

Nhưng khuyến nghị của tôi là nếu bạn có mảng 2D, dùng initializer
2D. Nó làm code dễ đọc hơn. (Trừ việc khởi tạo cả mảng bằng `0`,
trường hợp đó dùng `{0}` là idiom bất kể chiều của mảng.)

[i[Arrays-->multidimensional initializers]>]


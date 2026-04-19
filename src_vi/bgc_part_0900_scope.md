<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Scope {#scope}

[i[Scope]<]
Scope nói về chuyện biến nào nhìn thấy được trong ngữ cảnh nào.

## Block scope

[i[Scope-->block]<]
Đây là scope của gần như mọi biến dev định nghĩa. Nó bao gồm cả cái
mà ngôn ngữ khác có thể gọi là "function scope", tức biến khai báo
bên trong hàm.

Quy tắc cơ bản là nếu bạn khai báo biến trong một block được bao bởi
cặp ngoặc xoăn xoắn xuýt, scope của biến đó là block đó.

Nếu có block trong block, thì biến khai báo trong block _bên trong_
là local với block đó, không thấy được ở scope ngoài.

Khi scope của một biến kết thúc, biến đó không thể tham chiếu nữa, và
bạn có thể coi giá trị của nó đã bay vào [flw[bit
bucket|Bit_bucket]] khổng lồ trên trời.

Ví dụ với scope lồng nhau:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int a = 12;         // Local to outer block, but visible in inner block

    if  (a == 12) {
        int b = 99;     // Local to inner block, not visible in outer block

        printf("%d %d\n", a, b);  // OK: "12 99"
    }

    printf("%d\n", a);  // OK, we're still in a's scope

    printf("%d\n", b);  // ILLEGAL, out of b's scope
}
```

### Chỗ nào định nghĩa biến

Một sự thật vui là bạn có thể định nghĩa biến ở bất cứ đâu trong block,
trong mức độ hợp lý, chúng có scope của block đó, nhưng không dùng
được trước khi được định nghĩa.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i = 0;

    printf("%d\n", i);     // OK: "0"

    //printf("%d\n", j);   // ILLEGAL--can't use j before it's defined

    int j = 5;

    printf("%d %d\n", i, j);   // OK: "0 5"
}
```

Trước đây, C yêu cầu mọi biến phải định nghĩa trước bất kỳ code nào
trong block, nhưng chuẩn C99 không còn thế nữa.

### Che biến

[i[Variable hiding]<]
Nếu bạn có biến đặt tên giống nhau ở scope trong và scope ngoài, cái
ở scope trong được ưu tiên trong khi bạn đang chạy ở scope trong. Tức
là nó _che_ cái ở scope ngoài suốt thời gian tồn tại.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i = 10;

    {
        int i = 20;

        printf("%d\n", i);  // Inner scope i, 20 (outer i is hidden)
    }

    printf("%d\n", i);  // Outer scope i, 10
}
```

Có thể bạn đã để ý trong ví dụ đó tôi quẳng luôn một block vào ở dòng
7, chẳng có cả `for` hay `if` khởi sự! Chuyện này hoàn toàn hợp lệ.
Thỉnh thoảng dev muốn gom một mớ biến local lại cho một tính toán
nhanh và sẽ làm thế, nhưng hiếm khi thấy.
[i[Variable hiding]>]
[i[Scope-->block]>]

## File scope

[i[Scope-->file]<]
Nếu bạn định nghĩa biến ngoài block, biến đó có _file scope_. Nó nhìn
thấy được trong mọi hàm trong file xuất hiện sau nó, và được chia sẻ
giữa chúng. (Trường hợp ngoại lệ là nếu một block định nghĩa biến
cùng tên, nó sẽ che cái ở file scope.)

Đây là cái gần nhất với khái niệm bạn có thể coi là scope "global"
trong ngôn ngữ khác.

Ví dụ:

``` {.c .numberLines}
#include <stdio.h>

int shared = 10;    // File scope! Visible to the whole file after this!

void func1(void)
{
    shared += 100;  // Now shared holds 110
}

void func2(void)
{
    printf("%d\n", shared);  // Prints "110"
}

int main(void)
{
    func1();
    func2();
}
```

Chú ý rằng nếu `shared` được khai báo ở cuối file, nó sẽ không
compile. Nó phải được khai báo _trước_ bất kỳ hàm nào dùng nó.

Có những cách chỉnh thêm các item ở file scope, cụ thể với
[static](#static) và [extern](#extern), nhưng sẽ nói thêm sau.
[i[Scope-->file]>]

## Scope vòng lặp `for`

[i[Scope-->`for` loop]<]
Thật sự tôi không biết gọi nó là gì, vì C11 §6.8.5.3¶1 không cho nó
một tên riêng. Ta đã dùng vài lần trong sách này rồi. Nó là khi bạn
khai báo biến bên trong mệnh đề đầu của vòng `for`:

``` {.c}
for (int i = 0; i < 10; i++)
    printf("%d\n", i);

printf("%d\n", i);  // ILLEGAL--i is only in scope for the for-loop
```

Trong ví dụ đó, thời gian sống của `i` bắt đầu ngay lúc nó được định
nghĩa, và tiếp tục trong suốt vòng lặp.

Nếu thân vòng lặp nằm trong một block, các biến định nghĩa trong
`for` nhìn thấy được từ scope bên trong đó.

Dĩ nhiên là trừ khi scope trong đó che chúng đi. Ví dụ điên rồ này in
`999` năm lần:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    for (int i = 0; i < 5; i++) {
        int i = 999;  // Hides the i in the for-loop scope
        printf("%d\n", i);
    }
}
```
[i[Scope-->`for` loop]>]

## Ghi chú về function scope

[i[Scope-->function]<]
Spec C có đề cập _function scope_, nhưng nó được dùng chỉ với
_label_, thứ ta chưa bàn tới. Sẽ nói thêm hôm khác.
[i[Scope-->function]>]
[i[Scope]>]

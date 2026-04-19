<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Kiểu không hoàn chỉnh (Incomplete Types)

[i[Incomplete types]<]

Bạn có thể ngạc nhiên khi biết đoạn này build không lỗi:

``` {.c}
extern int a[];

int main(void)
{
    struct foo *x;
    union bar *y;
    enum baz *z;
}
```

Ta chưa hề cho kích thước của `a`. Và ta có con trỏ tới `struct`
`foo`, `bar`, và `baz` mà dường như không được khai báo ở đâu cả.

Và cảnh báo duy nhất tôi nhận được là `x`, `y`, và `z` không được
dùng.

Đây là các ví dụ về _kiểu không hoàn chỉnh_ (incomplete type).

Kiểu không hoàn chỉnh là kiểu mà kích thước (tức là kích thước
`sizeof` trả về) chưa biết. Cách nghĩ khác là kiểu bạn chưa khai báo
xong.

Bạn có thể có con trỏ tới kiểu không hoàn chỉnh, nhưng bạn không thể
dereference nó hay dùng số học con trỏ trên nó. Và bạn không thể
`sizeof` nó.

Vậy làm gì được với nó?

## Use case: cấu trúc tự tham chiếu

[i[Incomplete types-->self-referential `struct`s]<]

Tôi chỉ biết một use case thực sự: forward reference tới `struct`
hay `union` với các cấu trúc tự tham chiếu hay đồng phụ thuộc. (Tôi
sẽ dùng `struct` cho phần còn lại của các ví dụ này, nhưng tất cả
đều áp dụng ngang bằng cho `union`.)

Làm ví dụ kinh điển trước.

Nhưng trước đó, biết điều này! Khi bạn khai báo một `struct`,
`struct` đó không hoàn chỉnh cho tới khi dấu ngoặc nhọn đóng được
tới!

``` {.c}
struct antelope {              // struct antelope is incomplete here
    int leg_count;             // Still incomplete
    float stomach_fullness;    // Still incomplete
    float top_speed;           // Still incomplete
    char *nickname;            // Still incomplete
};                             // NOW it's complete.
```

Thì sao? Trông đủ hợp lý.

Nhưng nếu ta đang làm linked list thì sao? Mỗi node trong linked
list cần có tham chiếu tới node khác. Nhưng làm sao tạo tham chiếu
tới node khác nếu ta còn chưa khai báo xong cái node luôn?

Sự cho phép của C với kiểu không hoàn chỉnh làm điều đó thành khả
thi. Ta không thể khai báo một node, nhưng ta _có thể_ khai báo một
con trỏ tới nó, kể cả khi nó chưa hoàn chỉnh!

``` {.c}
struct node {
    int val;
    struct node *next;  // struct node is incomplete, but that's OK!
};
```

Dù `struct node` chưa hoàn chỉnh ở dòng 3, ta vẫn có thể khai báo
một con trỏ tới nó^[Cái này chạy vì trong C, con trỏ có cùng kích
thước bất kể kiểu dữ liệu chúng trỏ tới. Nên compiler không cần biết
kích thước `struct node` tại điểm này; nó chỉ cần biết kích thước
con trỏ.].

Ta có thể làm tương tự nếu ta có hai `struct` khác nhau tham chiếu
lẫn nhau:

``` {.c}
struct a {
    struct b *x;  // Refers to a `struct b`
};

struct b {
    struct a *x;  // Refers to a `struct a`
};
```

Ta không thể nào tạo được cặp cấu trúc đó nếu không có quy tắc thả
lỏng cho kiểu không hoàn chỉnh.

[i[Incomplete types-->self-referential `struct`s]>]

## Thông báo lỗi về kiểu không hoàn chỉnh

Bạn có đang nhận các lỗi kiểu này không?

``` {.default}
invalid application of ‘sizeof’ to incomplete type

invalid use of undefined type

dereferencing pointer to incomplete type
```

Thủ phạm có khả năng nhất: bạn có lẽ quên `#include` file header
khai báo kiểu đó.

## Các kiểu không hoàn chỉnh khác

Khai báo `struct` hay `union` không có thân tạo ra kiểu không hoàn
chỉnh, ví dụ `struct foo;`.

`enum` không hoàn chỉnh cho tới dấu ngoặc nhọn đóng.

`void` là kiểu không hoàn chỉnh.

Mảng khai báo `extern` không có kích thước là không hoàn chỉnh, ví
dụ:

``` {.c}
extern int a[];
```

Nếu là mảng không-`extern` không có kích thước có initializer theo
sau, nó không hoàn chỉnh cho tới dấu ngoặc nhọn đóng của
initializer.

## Use case: mảng trong file header

Có thể hữu ích khi khai báo kiểu mảng không hoàn chỉnh trong file
header. Trong trường hợp đó, phần lưu trữ thực (nơi mảng hoàn chỉnh
được khai báo) nên ở trong một file `.c` duy nhất. Nếu bạn đặt nó
trong file `.h`, nó sẽ bị nhân đôi mỗi lần file header được include.

Nên cái bạn có thể làm là tạo một file header với kiểu không hoàn
chỉnh tham chiếu tới mảng, như vầy:

``` {.c .numberLines}
// File: bar.h

#ifndef BAR_H
#define BAR_H

extern int my_array[];  // Incomplete type

#endif
```

Và trong file `.c`, thực sự định nghĩa mảng:

``` {.c .numberLines}
// File: bar.c

int my_array[1024];     // Complete type!
```

Rồi bạn có thể include header từ bao nhiêu chỗ tuỳ ý, và mỗi chỗ sẽ
tham chiếu tới cùng `my_array` nằm dưới.

``` {.c .numberLines}
// File: foo.c

#include <stdio.h>
#include "bar.h"    // includes the incomplete type for my_array

int main(void)
{
    my_array[0] = 10;

    printf("%d\n", my_array[0]);
}
```

Khi compile nhiều file, nhớ chỉ định mọi file `.c` cho compiler,
nhưng không cần file `.h`, ví dụ:

``` {.zsh}
gcc -o foo foo.c bar.c
```

## Hoàn chỉnh kiểu không hoàn chỉnh

Nếu bạn có kiểu không hoàn chỉnh, bạn có thể hoàn chỉnh nó bằng cách
định nghĩa `struct`, `union`, `enum`, hay mảng hoàn chỉnh trong cùng
scope.

``` {.c}
struct foo;        // incomplete type

struct foo *p;     // pointer, no problem

// struct foo f;   // Error: incomplete type!

struct foo {
    int x, y, z;
};                 // Now the struct foo is complete!

struct foo f;      // Success!
```

Lưu ý rằng dù `void` là kiểu không hoàn chỉnh, không có cách nào
hoàn chỉnh nó. Không phải ai đó nghĩ tới chuyện làm cái quái đó.
Nhưng nó cũng giải thích tại sao bạn có thể làm cái này:

``` {.c}
void *p;             // OK: pointer to incomplete type
```

và không thể làm cả hai cái này:

``` {.c}
void v;              // Error: declare variable of incomplete type

printf("%d\n", *p);  // Error: dereference incomplete type
```

Biết thêm càng tốt...

[i[Incomplete types]<]

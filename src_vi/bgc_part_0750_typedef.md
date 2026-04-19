<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# `typedef`: Tạo kiểu mới

[i[`typedef` keyword]<]
Thực ra không hẳn là tạo kiểu _mới_, mà là đặt tên mới cho kiểu đã có.
Thoạt nghe hơi vô nghĩa, nhưng ta có thể dùng nó để làm code gọn gàng
hơn hẳn.

## `typedef` về lý thuyết

Đại khái, bạn lấy một kiểu đã có rồi tạo alias (bí danh) cho nó bằng
`typedef`.

Như này:

``` {.c}
typedef int antelope;  // Make "antelope" an alias for "int"

antelope x = 10;       // Type "antelope" is the same as type "int"
```

Bạn có thể làm thế với bất kỳ kiểu có sẵn nào. Còn có thể tạo nhiều
kiểu bằng list ngăn bởi dấu phẩy:

``` {.c}
typedef int antelope, bagel, mushroom;  // These are all "int"
```

Tiện ghê, nhỉ? Đánh máy được `mushroom` thay cho `int`? Chắc bạn đang
_phấn khích lắm_ với tính năng này!

Được rồi Giáo sư Chế Giễu, chút nữa sẽ tới những cách dùng phổ biến
hơn.

### Scoping

[i[`typedef` keyword-->scoping rules]<]
`typedef` tuân theo [quy tắc scope](#scope) thông thường.

Vì vậy, rất hay gặp `typedef` ở file scope ("global") để mọi hàm đều
dùng được kiểu mới thoải mái.

## `typedef` trong thực tế

Đổi tên `int` thành cái khác thì cũng không thú vị lắm. Xem `typedef`
thường xuất hiện ở đâu.

### `typedef` và `struct` {#typedef-struct}

[i[`typedef` keyword-->with `struct`s]<]
Đôi khi một `struct` sẽ được `typedef` ra tên mới để khỏi phải gõ từ
`struct` đi `struct` lại.

``` {.c}
struct animal {
    char *name;
    int leg_count, speed;
};

//  original name      new name
//            |         |
//            v         v
//      |-----------| |----|
typedef struct animal animal;

struct animal y;  // This works
animal z;         // This also works because "animal" is an alias
```

Cá nhân tôi không thích kiểu này lắm. Tôi thích sự rõ ràng mà code có
được khi thêm chữ `struct` vào trước kiểu, lập trình viên biết ngay
mình đang xử lý cái gì. Nhưng vì nó phổ biến nên tôi đưa vào đây.

Giờ tôi muốn chạy lại đúng ví dụ đó theo cách bạn hay thấy. Ta sẽ đặt
`struct animal` _vào bên trong_ `typedef`. Có thể gộp hết vào như này:

``` {.c}
//  original name
//            |
//            v
//      |-----------|
typedef struct animal {
    char *name;
    int leg_count, speed;
} animal;                         // <-- new name

struct animal y;  // This works
animal z;         // This also works because "animal" is an alias
```

Y chang ví dụ trước, chỉ ngắn gọn hơn.

[i[`typedef` keyword-->with anonymous `struct`s]<]
Nhưng chưa hết! Còn một kiểu rút gọn phổ biến nữa bạn có thể thấy
trong code, dùng cái gọi là _anonymous structure_ (cấu trúc vô danh)^[
Ta sẽ nói thêm về chúng sau.]. Hoá ra ở nhiều chỗ bạn không cần đặt
tên cho structure, và dùng với `typedef` là một trong những chỗ đó.

Làm lại ví dụ với anonymous structure:

``` {.c}
//  Anonymous struct! It has no name!
//         |
//         v
//      |----|
typedef struct {
    char *name;
    int leg_count, speed;
} animal;                         // <-- new name

//struct animal y;  // ERROR: this no longer works--no such struct!
animal z;           // This works because "animal" is an alias
```

Một ví dụ khác, có thể gặp thứ kiểu như:

``` {.c}
typedef struct {
    int x, y;
} point;

point p = {.x=20, .y=40};

printf("%d, %d\n", p.x, p.y);  // 20, 40
```
[i[`typedef` keyword-->with anonymous `struct`s]>]
[i[`typedef` keyword-->with `struct`s]>]
[i[`typedef` keyword-->scoping rules]>]

### `typedef` và các kiểu khác

Không phải dùng `typedef` với một kiểu đơn giản như `int` là hoàn toàn
vô ích, nó giúp bạn trừu tượng hoá các kiểu để dễ đổi sau này.

Ví dụ, nếu `float` rải khắp code của bạn ở 100 tỉ chỗ, sẽ rất đau đầu
nếu đổi hết sang `double` khi sau này vì lý do nào đó bạn phải làm
thế.

Nhưng nếu bạn chuẩn bị một chút:

``` {.c}
typedef float app_float;

// and

app_float f1, f2, f3;
```

Thì sau này muốn đổi sang kiểu khác, chẳng hạn `long double`, bạn chỉ
cần đổi `typedef`:

``` {.c}
//        voila!
//      |---------|
typedef long double app_float;

// and no need to change this line:

app_float f1, f2, f3;  // Now these are all long doubles
```

### `typedef` và con trỏ

[i[`typedef` keyword-->with pointers]<]
Bạn có thể tạo một kiểu là con trỏ.

``` {.c}
typedef int *intptr;

int a = 10;
intptr x = &a;  // "intptr" is type "int*"
```

Tôi rất không thích kiểu này. Nó giấu đi chuyện `x` là kiểu con trỏ vì
bạn không thấy dấu `*` nào trong khai báo.

IMHO, tốt hơn là thể hiện rõ bạn đang khai báo kiểu con trỏ để các dev
khác nhìn thấy rõ và không nhầm `x` là kiểu không phải con trỏ.

Nhưng lần đếm gần nhất thì có chừng 832.007 người không đồng ý với
tôi.
[i[`typedef` keyword-->with pointers]>]

### `typedef` và cách viết hoa

Tôi đã thấy đủ loại cách viết hoa cho `typedef`.

``` {.c}
typedef struct {
    int x, y;
} my_point;          // lower snake case

typedef struct {
    int x, y;
} MyPoint;          // CamelCase

typedef struct {
    int x, y;
} Mypoint;          // Leading uppercase

typedef struct {
    int x, y;
} MY_POINT;          // UPPER SNAKE CASE
```

Đặc tả C11 không ép theo cách nào, và có ví dụ cả viết hoa hết lẫn
viết thường hết.

K&R2 chủ yếu dùng leading uppercase, nhưng cũng có vài ví dụ viết hoa
hết và snake case (với hậu tố `_t`).

Nếu bạn đang có style guide, theo nó. Nếu chưa, vớ lấy một cái rồi
theo.

## Mảng và `typedef`

[i[`typedef` keyword-->with arrays]<]
Cú pháp hơi kỳ, và theo kinh nghiệm của tôi thì hiếm gặp, nhưng bạn
có thể `typedef` một mảng với số phần tử xác định.

``` {.c}
// Make type five_ints an array of 5 ints
typedef int five_ints[5];

five_ints x = {11, 22, 33, 44, 55};
```

Tôi không thích vì nó giấu đi bản chất mảng của biến, nhưng làm được.
[i[`typedef` keyword-->with arrays]>]
[i[`typedef` keyword]>]

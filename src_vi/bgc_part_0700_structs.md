<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Struct {#structs}

[i[`struct` keyword]<]
Trong C, có một thứ gọi là `struct`, một kiểu do người dùng định nghĩa
được, giữ nhiều mẩu dữ liệu, có thể thuộc các kiểu khác nhau.

Đây là cách tiện lợi để gói nhiều biến vào làm một. Việc này có ích
khi truyền biến cho hàm (chỉ cần truyền một thay vì nhiều), và hữu
dụng để tổ chức dữ liệu và làm code dễ đọc hơn.

Nếu bạn đến từ ngôn ngữ khác, có thể bạn đã quen với khái niệm _class_
và _object_. Những thứ này không tồn tại sẵn trong C^[Mặc dù trong C
các mẩu riêng lẻ trong bộ nhớ như `int` được gọi là "object", chúng
không phải object theo nghĩa lập trình hướng đối tượng.]. Bạn có thể
nghĩ về `struct` như class chỉ có các trường dữ liệu, không có
method.

## Khai báo một struct

[i[`struct` keyword-->declaring]<]
Bạn có thể khai báo một `struct` trong code của mình như này:

``` {.c}
struct car {
    char *name;
    float price;
    int speed;
};
```

Chuyện này thường được làm ở scope toàn cục bên ngoài các hàm để
`struct` khả dụng toàn cục.

Khi làm vậy, bạn đang tạo một _kiểu_ (type) mới. Tên kiểu đầy đủ là
`struct car`. (Không phải chỉ `car`, viết thế không hoạt động.)

Chưa có biến nào kiểu đó, nhưng ta có thể khai báo vài biến:

``` {.c}
struct car saturn;  // Variable "saturn" of type "struct car"
```

Giờ ta có biến chưa khởi tạo `saturn`^[Saturn là một hãng xe phổ thông
khá được ưa chuộng ở Hoa Kỳ cho tới khi bị đóng cửa vì khủng hoảng
2008, buồn thay với những fan như chúng ta.] kiểu `struct car`.

Ta nên khởi tạo nó! Nhưng làm sao đặt giá trị cho từng trường riêng
lẻ?

Giống như nhiều ngôn ngữ khác lấy lại từ C, ta sẽ dùng toán tử chấm
(`.`) để truy cập từng trường.

``` {.c}
saturn.name = "Saturn SL/2";
saturn.price = 15999.99;
saturn.speed = 175;

printf("Name:           %s\n", saturn.name);
printf("Price (USD):    %f\n", saturn.price);
printf("Top Speed (km): %d\n", saturn.speed);
```

Ở các dòng đầu, ta đặt giá trị vào `struct car`, rồi ở đoạn sau, in
các giá trị đó ra.
[i[`struct` keyword-->declaring]>]

## Khởi tạo Struct {#struct-initializers}

[i[`struct` keyword-->initializers]<]
Ví dụ ở mục trước hơi cồng kềnh. Chắc phải có cách tốt hơn để khởi tạo
biến `struct`!

Bạn có thể làm với initializer bằng cách đặt giá trị cho các trường
_theo thứ tự chúng xuất hiện trong `struct`_ khi bạn định nghĩa biến.
(Cách này không chạy sau khi biến đã được định nghĩa, phải xảy ra
ngay lúc định nghĩa).

``` {.c}
struct car {
    char *name;
    float price;
    int speed;
};

// Now with an initializer! Same field order as in the struct declaration:
struct car saturn = {"Saturn SL/2", 16000.99, 175};

printf("Name:      %s\n", saturn.name);
printf("Price:     %f\n", saturn.price);
printf("Top Speed: %d km\n", saturn.speed);
```

Việc các trường trong initializer phải cùng thứ tự nghe hơi rùng mình.
Nếu ai đó đổi thứ tự trong `struct car`, có thể làm hỏng hết code
khác!

Ta có thể cụ thể hơn với initializer:

``` {.c}
struct car saturn = {.speed=175, .name="Saturn SL/2"};
```

Giờ nó độc lập với thứ tự trong khai báo `struct`. Code an toàn hơn
hẳn.

Tương tự như initializer của mảng, bất kỳ field designator nào bị bỏ
sót đều được khởi tạo về không (trong trường hợp này, đó là `.price`,
tôi đã bỏ qua).
[i[`struct` keyword-->initializers]>]

## Truyền Struct cho hàm

[i[`struct` keyword-->passing and returning]<]
Bạn có vài cách để truyền `struct` cho hàm.

1. Truyền bản thân `struct`.
2. Truyền con trỏ tới `struct`.

Nhớ rằng khi bạn truyền thứ gì đó cho hàm, một _bản sao_ của thứ đó
được tạo ra cho hàm thao tác, bất kể đó là bản sao của con trỏ, của
`int`, của `struct`, hay bất cứ thứ gì.

Về cơ bản có hai trường hợp bạn muốn truyền con trỏ tới `struct`:

1. Bạn cần hàm có thể thay đổi `struct` được truyền vào, và để những
   thay đổi đó hiện ra ở phía người gọi.
2. `struct` hơi lớn và chép nó lên stack tốn hơn là chép một con
   trỏ^[Một con trỏ có lẽ 8 byte trên hệ thống 64 bit.].

Vì hai lý do đó, truyền con trỏ tới `struct` cho hàm phổ biến hơn
nhiều, dù truyền cả `struct` thì không hề phạm luật.

Thử truyền con trỏ, làm một hàm cho phép bạn đặt trường `.price` của
`struct car`:

``` {.c .numberLines}
#include <stdio.h>

struct car {
    char *name;
    float price;
    int speed;
};

int main(void)
{
    struct car saturn = {.speed=175, .name="Saturn SL/2"};

    // Pass a pointer to this struct car, along with a new,
    // more realistic, price:
    set_price(&saturn, 799.99);

    printf("Price: %f\n", saturn.price);
}
```

Bạn nên có thể nghĩ ra signature của hàm `set_price()` chỉ bằng cách
nhìn kiểu của các đối số ta có ở đó.

`saturn` là `struct car`, nên `&saturn` phải là địa chỉ của `struct
car`, tức là con trỏ tới `struct car`, cụ thể là `struct car*`.

Và `799.99` là `float`.

Nên khai báo hàm phải như này:

``` {.c}
void set_price(struct car *c, float new_price)
```

Ta chỉ cần viết phần thân. Lần thử đầu tiên có thể là:

``` {.c}
void set_price(struct car *c, float new_price) {
    c.price = new_price;  // ERROR!!
}
```

Cách đó không chạy vì toán tử chấm chỉ chạy trên `struct`... nó không
chạy trên _con trỏ_ tới `struct`.

Được rồi, ta có thể dereference biến `c` để "de-pointer" nó để tới
bản thân `struct`. Dereference một `struct car*` cho ra `struct car`
mà con trỏ trỏ tới, ta nên có thể dùng toán tử chấm lên đó:

``` {.c}
void set_price(struct car *c, float new_price) {
    (*c).price = new_price;  // Works, but is ugly and non-idiomatic :(
}
```

Và chạy! Nhưng hơi lôi thôi khi gõ hết đám dấu ngoặc với dấu sao. C có
một thứ đường cú pháp gọi là _toán tử mũi tên_ (arrow operator) giúp
chuyện đó.
[i[`struct` keyword-->passing and returning]>]

## Toán tử mũi tên

[i[`->` arrow operator]<]
Toán tử mũi tên giúp tham chiếu tới các trường trong con trỏ tới
`struct`.

``` {.c}
void set_price(struct car *c, float new_price) {
    // (*c).price = new_price;  // Works, but non-idiomatic :(
    //
    // The line above is 100% equivalent to the one below:

    c->price = new_price;  // That's the one!
}
```

Vậy khi truy cập các trường, khi nào dùng chấm và khi nào dùng mũi
tên?

* Nếu bạn có `struct`, dùng chấm (`.`).
* Nếu bạn có con trỏ tới `struct`, dùng mũi tên (`->`).
[i[`->` arrow operator]>]

## Sao chép và trả về `struct`

[i[`struct` keyword-->copying]<]
Đây là phần dễ cho bạn!

Chỉ cần gán từ cái này sang cái kia!

``` {.c}
struct car a, b;

b = a;  // Copy the struct
```

Và trả về một `struct` (thay vì trả về một con trỏ tới nó) từ hàm
cũng tạo một bản sao tương tự vào biến nhận.

Đây không phải "deep copy"^[Một _deep copy_ đi theo các con trỏ trong
`struct` và chép cả dữ liệu chúng trỏ tới. Một _shallow copy_ chỉ chép
các con trỏ, chứ không chép thứ chúng trỏ tới. C không có sẵn chức
năng deep copy tích hợp nào.]. Tất cả các trường được chép y nguyên,
kể cả con trỏ tới các thứ.
[i[`struct` keyword-->copying]>]

## So sánh `struct`

[i[`struct` keyword-->comparing]<]
Chỉ có một cách an toàn duy nhất: so sánh từng trường một.

Bạn có thể nghĩ có thể dùng
[fl[`memcmp()`|https://beej.us/guide/bgclr/html/split/stringref.html#man-strcmp]],
nhưng nó không xử lý được trường hợp có thể có [padding
bytes](#struct-padding-bytes) nằm lẫn trong đó.

Nếu bạn xoá `struct` về không trước bằng
[fl[`memset()`|https://beej.us/guide/bgclr/html/split/stringref.html#man-memset]],
thì nó _có thể_ chạy, dù vẫn có thể có những phần tử lạ
[fl[không so sánh như bạn
mong|https://stackoverflow.com/questions/141720/how-do-you-compare-structs-for-equality-in-c]].
[i[`struct` keyword-->comparing]>] [i[`struct` keyword]>]

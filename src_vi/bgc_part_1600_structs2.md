<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# `struct` II: Nghịch `struct` vui hơn

[i[`struct` keyword]<]

Hóa ra còn khá nhiều thứ có thể làm với `struct` mà ta chưa bàn,
nhưng nó chỉ là một đống thứ linh tinh. Nên ta nhét hết vào chương
này.

Nếu bạn đã thạo căn bản về `struct`, bạn có thể làm tròn kiến thức
ở đây.

## Khởi tạo `struct` lồng nhau và mảng

[i[`struct` keyword-->initializers]<]

Nhớ cách bạn có thể [khởi tạo thành viên struct theo các dòng
này](#struct-initializers) không?

``` {.c}
struct foo x = {.a=12, .b=3.14};
```

Hóa ra ta có nhiều sức mạnh trong các initializer này hơn là lúc đầu
chia sẻ. Hấp dẫn!

Một chuyện là, nếu bạn có substructure lồng nhau như sau, bạn có thể
khởi tạo thành viên của substructure đó bằng cách đi theo tên biến
xuống dần:

``` {.c}
struct foo x = {.a.b.c=12};
```

Xem ví dụ:

``` {.c .numberLines}
#include <stdio.h>

struct cabin_information {
    int window_count;
    int o2level;
};

struct spaceship {
    char *manufacturer;
    struct cabin_information ci;
};

int main(void)
{
    struct spaceship s = {
        .manufacturer="General Products",
        .ci.window_count = 8,   // <-- NESTED INITIALIZER!
        .ci.o2level = 21
    };

    printf("%s: %d seats, %d%% oxygen\n",
        s.manufacturer, s.ci.window_count, s.ci.o2level);
}
```

Xem dòng 16-17! Đó là chỗ ta khởi tạo các thành viên của
`struct cabin_information` trong định nghĩa `s`, tức `struct
spaceship` của ta.

Và đây là lựa chọn khác cho cùng initializer đó, lần này ta làm một
thứ trông chuẩn hơn, nhưng cách nào cũng chạy:

``` {.c .numberLines startFrom="15"}
    struct spaceship s = {
        .manufacturer="General Products",
        .ci={
            .window_count = 8,
            .o2level = 21
        }
    };
```

Giờ, như thể thông tin ở trên còn chưa đủ ngoạn mục, ta cũng có thể
trộn initializer mảng vào đó luôn.

Sửa cái này để có mảng thông tin hành khách, và ta có thể xem
initializer hoạt động trong đó ra sao.

``` {.c .numberLines}
#include <stdio.h>

struct passenger {
    char *name;
    int covid_vaccinated; // Boolean
};

#define MAX_PASSENGERS 8

struct spaceship {
    char *manufacturer;
    struct passenger passenger[MAX_PASSENGERS];
};

int main(void)
{
    struct spaceship s = {
        .manufacturer="General Products",
        .passenger = {
            // Initialize a field at a time
            [0].name = "Gridley, Lewis",
            [0].covid_vaccinated = 0,

            // Or all at once
            [7] = {.name="Brown, Teela", .covid_vaccinated=1},
        }
    };

    printf("Passengers for %s ship:\n", s.manufacturer);

    for (int i = 0; i < MAX_PASSENGERS; i++)
        if (s.passenger[i].name != NULL)
            printf("    %s (%svaccinated)\n",
                s.passenger[i].name,
                s.passenger[i].covid_vaccinated? "": "not ");
}
```

[i[`struct` keyword-->initializers]>]

## `struct` vô danh

[i[`struct` keyword-->anonymous]<]

Đây là "struct không tên". Ta cũng có nhắc mấy cái này ở phần
[`typedef`](#typedef-struct), nhưng ta sẽ ôn lại ở đây.

Đây là `struct` thường:

``` {.c}
struct animal {
    char *name;
    int leg_count, speed;
};
```

Và đây là phiên bản vô danh tương đương:

``` {.c}
struct {              // <-- No name!
    char *name;
    int leg_count, speed;
};
```

Ừuuuu. Vậy ta có một `struct` không tên, không có cách nào dùng sau
này? Nghe có vẻ vô dụng.

Công nhận, trong ví dụ đó, đúng là thế. Nhưng ta vẫn có thể tận dụng
nó bằng vài cách.

Một cách hiếm, nhưng vì `struct` vô danh đại diện một kiểu, ta có
thể đặt vài tên biến ngay sau nó và dùng chúng.

``` {.c}
struct {                   // <-- No name!
    char *name;
    int leg_count, speed;
} a, b, c;                 // 3 variables of this struct type

a.name = "antelope";
c.leg_count = 4;           // for example
```

Nhưng cũng không hữu dụng mấy.

Phổ biến hơn nhiều là dùng `struct` vô danh với `typedef` để có thể
dùng sau (ví dụ để truyền biến cho hàm).

``` {.c}
typedef struct {                   // <-- No name!
    char *name;
    int leg_count, speed;
} animal;                          // New type: animal

animal a, b, c;

a.name = "antelope";
c.leg_count = 4;           // for example
```

Cá nhân tôi không dùng nhiều `struct` vô danh. Tôi thấy dễ chịu hơn
khi thấy cả `struct animal` trước tên biến trong khai báo.

Nhưng đó chỉ là, kiểu, ý kiến của tôi thôi, anh bạn.

[i[`struct` keyword-->anonymous]>]

## `struct` tự tham chiếu

[i[`struct` keyword-->self-referential]<]

Với bất kỳ cấu trúc dữ liệu dạng đồ thị nào, có con trỏ tới các
node/đỉnh nối với nó là hữu ích. Nhưng điều này nghĩa là trong định
nghĩa một node, bạn cần có con trỏ tới một node. Con gà và quả
trứng!

Nhưng hóa ra bạn làm được chuyện này trong C mà không gặp vấn đề gì.

Ví dụ, đây là một node của linked list:

``` {.c}
struct node {
    int data;
    struct node *next;
};
```

Quan trọng là `next` là con trỏ. Đây là điều cho phép cả mớ build
được. Dù compiler chưa biết nguyên `struct node` trông thế nào, mọi
con trỏ đều cùng kích thước.

Đây là một chương trình linked list ẩu để thử nó:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

struct node {
    int data;
    struct node *next;
};

int main(void)
{
    struct node *head;

    // Hackishly set up a linked list (11)->(22)->(33)
    head = malloc(sizeof(struct node));
    head->data = 11;
    head->next = malloc(sizeof(struct node));
    head->next->data = 22;
    head->next->next = malloc(sizeof(struct node));
    head->next->next->data = 33;
    head->next->next->next = NULL;

    // Traverse it
    for (struct node *cur = head; cur != NULL; cur = cur->next) {
        printf("%d\n", cur->data);
    }
}
```

Chạy nó in ra:

``` {.default}
11
22
33
```

[i[`struct` keyword-->self-referential]>]

## Flexible array member

[i[`struct` keyword-->flexible array members]<]

Ngày xửa ngày xưa, khi người ta còn đẽo code C từ gỗ, có người nghĩ
sẽ hay nếu có thể cấp phát `struct` mà có mảng độ dài biến đổi ở
cuối.

Tôi muốn nói rõ rằng phần đầu của đoạn này là cách cũ, và ta sẽ làm
cách mới ở phần sau.

Ví dụ, có thể bạn định nghĩa một `struct` để chứa chuỗi cùng độ dài
chuỗi đó. Nó sẽ có độ dài và một mảng để chứa dữ liệu. Có khi thế
này:

``` {.c}
struct len_string {
    int length;
    char data[8];
};
```

Nhưng cái đó có `8` được đóng cứng làm độ dài tối đa của chuỗi, mà
không nhiều lắm. Thế nếu ta làm gì đó _ngầu_ và chỉ cần `malloc()`
thêm không gian ở cuối sau struct, rồi để dữ liệu tràn vào không
gian đó?

Làm thế đi, rồi cấp phát thêm 40 byte:

``` {.c}
struct len_string *s = malloc(sizeof *s + 40);
```

Vì `data` là field cuối của `struct`, nếu ta làm tràn field đó, nó
chảy ra không gian mà ta đã cấp phát! Vì vậy, trò này chỉ hoạt động
nếu mảng ngắn là field _cuối_ của `struct`.

``` {.c}
// Copy more than 8 bytes!

strcpy(s->data, "Hello, world!");  // Won't crash. Probably.
```

[i[Arrays-->zero length]<]

Thật ra, có một cách lách thường gặp cho compiler để làm chuyện này,
bạn cấp phát một mảng độ dài không ở cuối:

``` {.c}
struct len_string {
    int length;
    char data[0];
};
```

Và rồi mỗi byte thừa bạn cấp phát đã sẵn sàng để dùng trong chuỗi
đó.

Vì `data` là field cuối của `struct`, nếu ta làm tràn field đó, nó
chảy ra không gian mà ta đã cấp phát!

``` {.c}
// Copy more than 8 bytes!

strcpy(s->data, "Hello, world!");  // Won't crash. Probably.
```

Nhưng dĩ nhiên, truy cập dữ liệu vượt cuối mảng đó là undefined
behavior! Trong thời hiện đại, ta không còn hạ mình làm kiểu man rợ
đó.

[i[Arrays-->zero length]>]

May cho ta, ta vẫn có hiệu quả tương tự với C99 trở về sau, nhưng
giờ là hợp pháp.

Chỉ việc đổi định nghĩa trên để mảng không có kích thước^[Kỹ thuật
mà nói, ta gọi nó có _incomplete type_.]:

``` {.c}
struct len_string {
    int length;
    char data[];
};
```

Vẫn thế, cái này chỉ chạy nếu flexible array member là field _cuối_
của `struct`.

Rồi ta có thể cấp phát bao nhiêu không gian tùy ý cho các chuỗi đó
bằng cách `malloc()` lớn hơn `struct len_string`, như trong ví dụ
này tạo một `struct len_string` mới từ chuỗi C:


``` {.c}
struct len_string *len_string_from_c_string(char *s)
{
    int len = strlen(s);

    // Allocate "len" more bytes than we'd normally need
    struct len_string *ls = malloc(sizeof *ls + len);

    ls->length = len;

    // Copy the string into those extra bytes
    memcpy(ls->data, s, len);

    return ls;
}
```

[i[`struct` keyword-->flexible array members]>]

## Byte đệm (padding) {#struct-padding-bytes}

[i[`struct` keyword-->padding bytes]<]

Cẩn thận rằng C được phép thêm byte đệm bên trong hoặc sau một
`struct` tùy ý nó. Bạn không thể tin rằng chúng sẽ liền kề nhau
trong bộ nhớ^[Dù vài compiler có tùy chọn ép chuyện này xảy ra, tra
`__attribute__((packed))` để xem cách làm với GCC.].

Xem chương trình này. Ta xuất hai số. Một là tổng `sizeof` của từng
kiểu field riêng lẻ. Cái kia là `sizeof` cả `struct`.

Lẽ ra ta kỳ vọng chúng bằng nhau. Kích thước của cái toàn thể là
tổng kích thước các phần, đúng không?

``` {.c .numberLines}
#include <stdio.h>

struct foo {
    int a;
    char b;
    int c;
    char d;
};

int main(void)
{
    printf("%zu\n", sizeof(int) + sizeof(char) + sizeof(int) + sizeof(char));
    printf("%zu\n", sizeof(struct foo));
}
```

Nhưng trên hệ của tôi, cái này xuất:

``` {.default}
10
16
```

Chúng không bằng nhau! Compiler đã thêm 6 byte đệm để giúp nó chạy
nhanh hơn. Có thể bạn nhận kết quả khác với compiler của bạn, nhưng
trừ khi bạn ép buộc, bạn không thể chắc không có đệm.

[i[`struct` keyword-->padding bytes]>]

## `offsetof`

[i[`offsetof()` macro]<]

Trong đoạn trước, ta thấy compiler có thể chêm byte đệm tùy ý vào
trong cấu trúc.

Nếu ta cần biết chúng ở đâu? Ta có thể đo bằng `offsetof`, định
nghĩa trong `<stddef.h>`.

Sửa code ở trên để in offset của từng field trong `struct`:

``` {.c .numberLines}
#include <stdio.h>
#include <stddef.h>

struct foo {
    int a;
    char b;
    int c;
    char d;
};

int main(void)
{
    printf("%zu\n", offsetof(struct foo, a));
    printf("%zu\n", offsetof(struct foo, b));
    printf("%zu\n", offsetof(struct foo, c));
    printf("%zu\n", offsetof(struct foo, d));
}
```

Với tôi, cái này xuất:

``` {.default}
0
4
8
12
```

cho biết ta đang dùng 4 byte cho mỗi field. Hơi lạ, vì `char` chỉ
có 1 byte, đúng không? Compiler đang đặt 3 byte đệm sau mỗi `char`
để mọi field đều dài 4 byte. Chắc chuyện này sẽ chạy nhanh hơn trên
CPU của tôi.

[i[`offsetof()` macro]>]

<!--

6.7.2.1

15 Within a structure object, the non-bit-field members and the units in which bit-fields reside have addresses that increase in the order in which they are declared. A pointer to a structure object, suitably converted, points to its initial member (or if that member is a bit-field, then to the unit in which it resides), and vice versa. There may be unnamed padding within a structure object, but not at its beginning.

6.2.7 Compatible type and composite type

1 Two types have compatible type if their types are the same.

6.5

7 An object shall have its stored value accessed only by an lvalue expression that has one of the following types:

- a type compatible with the effective type of the object

-->

## OOP giả {#fake-oop}

Có một trò hơi lạm dụng, kiểu kiểu OOP, mà bạn có thể làm với
`struct`.

Vì con trỏ tới `struct` trùng với con trỏ tới phần tử đầu tiên của
`struct`, bạn có thể thoải mái ép kiểu con trỏ tới `struct` sang con
trỏ tới phần tử đầu tiên.

Điều này có nghĩa là ta có thể dựng tình huống thế này:

``` {.c}
struct parent {
    int a, b;
};

struct child {
    struct parent super;  // MUST be first
    int c, d;
};
```

Rồi ta có thể truyền con trỏ tới `struct child` cho một hàm mong đợi
_hoặc_ con trỏ tới `struct parent`!

Vì `struct parent super` là phần tử đầu của `struct child`, con trỏ
tới bất kỳ `struct child` nào cũng trùng với con trỏ tới field
`super` đó^[Nhân tiện, `super` không phải từ khóa. Tôi chỉ mượn vài
thuật ngữ OOP thôi.].

Dựng ví dụ luôn. Ta làm `struct` như trên, rồi truyền con trỏ tới
`struct child` cho hàm cần con trỏ tới `struct parent`... và vẫn
chạy.

``` {.c .numberLines}
#include <stdio.h>

struct parent {
    int a, b;
};

struct child {
    struct parent super;  // MUST be first
    int c, d;
};

// Making the argument `void*` so we can pass any type into it
// (namely a struct parent or struct child)
void print_parent(void *p)
{
    // Expects a struct parent--but a struct child will also work
    // because the pointer points to the struct parent in the first
    // field:
    struct parent *self = p;

    printf("Parent: %d, %d\n", self->a, self->b);
}

void print_child(struct child *self)
{
    printf("Child: %d, %d\n", self->c, self->d);
}

int main(void)
{
    struct child c = {.super.a=1, .super.b=2, .c=3, .d=4};

    print_child(&c);
    print_parent(&c);  // Also works even though it's a struct child!
}
```

Thấy chuyện ta làm ở dòng cuối `main()` chứ? Ta gọi `print_parent()`
mà truyền `struct child*` làm tham số! Dù `print_parent()` cần tham
số trỏ tới `struct parent`, ta _thoát_ được vì field đầu của
`struct child` là `struct parent`.

Vẫn vậy, cái này chạy được vì con trỏ tới `struct` có cùng giá trị
với con trỏ tới field đầu trong `struct` đó.

Tất cả dựa trên phần này của spec:

> **§6.7.2.1¶15** [...] A pointer to a structure object, suitably
> converted, points to its initial member [...], and vice versa.

và

> **§6.5¶7** An object shall have its stored value accessed only by an
> lvalue expression that has one of the following types:
>
> * a type compatible with the effective type of the object
> * [...]

và giả định của tôi rằng "suitably converted" nghĩa là "ép kiểu sang
kiểu hiệu lực của phần tử đầu".

## Bit-field

[i[`struct` keyword-->bit fields]<]

Theo kinh nghiệm của tôi, mấy cái này ít khi dùng, nhưng bạn có thể
thấy đây đó, đặc biệt trong ứng dụng tầng thấp nơi người ta dồn bit
vào không gian lớn hơn.

Xem đoạn code để minh họa trường hợp dùng:

``` {.c .numberLines}
#include <stdio.h>

struct foo {
    unsigned int a;
    unsigned int b;
    unsigned int c;
    unsigned int d;
};

int main(void)
{
    printf("%zu\n", sizeof(struct foo));
}
```

Với tôi, cái này in `16`. Hợp lý, vì `unsigned` là 4 byte trên hệ
của tôi.

Nhưng nếu ta biết mọi giá trị sẽ được lưu trong `a` và `b` đều chứa
được trong 5 bit, và giá trị trong `c` và `d` chứa được trong 3 bit?
Tổng cộng mới 16 bit. Sao lại phải dành 128 bit cho chúng nếu ta chỉ
dùng 16?

Ta có thể nói với C làm-ơn-thử-gói các giá trị này lại. Ta có thể
chỉ định số bit tối đa mà giá trị có thể chiếm (từ 1 lên kích thước
kiểu chứa).

Làm bằng cách đặt dấu hai chấm sau tên field, rồi tới độ rộng field
tính bằng bit.

``` {.c .numberLines startFrom="3"}
struct foo {
    unsigned int a:5;
    unsigned int b:5;
    unsigned int c:3;
    unsigned int d:3;
};
```

Giờ khi tôi hỏi C `struct foo` lớn cỡ nào, nó nói 4! Trước là 16
byte, giờ chỉ còn 4. Nó đã "gói" 4 giá trị đó vào 4 byte, tiết kiệm
bộ nhớ gấp bốn.

Đánh đổi dĩ nhiên là field 5-bit chỉ chứa giá trị 0-31 và field
3-bit chỉ chứa giá trị 0-7. Nhưng đời sau cùng vẫn là về thỏa hiệp.

### Bit-field không liền kề

Một cái bẫy: C chỉ gộp bit-field **liền kề** thôi. Nếu chúng bị ngắt
bởi non-bit-field, bạn không tiết kiệm được gì:

``` {.c}
struct foo {            // sizeof(struct foo) == 16 (for me)
    unsigned int a:1;   // since a is not adjacent to c.
    unsigned int b;
    unsigned int c:1;
    unsigned int d;
};
```

Trong ví dụ đó, vì `a` không liền kề `c`, chúng đều được "gói" vào
`int` riêng của mình.

Nên ta có một `int` cho mỗi `a`, `b`, `c`, `d`. Vì `int` của tôi là
4 byte, tổng cộng 16 byte.

Sắp xếp lại nhanh tiết kiệm không gian từ 16 byte xuống 12 byte
(trên hệ của tôi):

``` {.c}
struct foo {            // sizeof(struct foo) == 12 (for me)
    unsigned int a:1;
    unsigned int c:1;
    unsigned int b;
    unsigned int d;
};
```

Và giờ, vì `a` kế bên `c`, compiler đặt chúng vào một `int` duy
nhất.

Nên ta có một `int` cho `a` và `c` gộp, và một `int` mỗi cái cho
`b` và `d`. Tổng cộng 3 `int`, hay 12 byte.

Đặt hết bitfield chung với nhau để compiler gộp chúng.

### `int` có dấu hay không dấu

Nếu bạn chỉ khai báo bit-field là `int`, các compiler khác nhau sẽ
xử lý nó là `signed` hoặc `unsigned`. Giống tình huống với `char`.

Hãy rõ ràng về dấu khi dùng bit-field.

### Bit-field không tên

Trong vài tình huống cụ thể, bạn có thể cần dành một số bit vì lý do
phần cứng, nhưng không cần dùng chúng trong code.

Ví dụ, giả sử bạn có một byte mà 2 bit trên có ý nghĩa, 1 bit dưới
có ý nghĩa, còn 5 bit giữa không được bạn dùng^[Giả sử `char` 8 bit,
tức là `CHAR_BIT == 8`.].

Ta _có thể_ làm thế này:

``` {.c}
struct foo {
    unsigned char a:2;
    unsigned char dummy:5;
    unsigned char b:1;
};
```

Và cái đó chạy, trong code ta dùng `a` và `b`, không bao giờ dùng
`dummy`. Nó chỉ ở đó để ăn hết 5 bit để chắc rằng `a` và `b` ở đúng
vị trí "yêu cầu" (theo ví dụ giả định này) trong byte.

C cho ta một cách dọn cái này: _bit-field không tên_. Bạn chỉ cần
bỏ tên (`dummy`) trong trường hợp này, và C hoàn toàn hài lòng cho
hiệu quả tương tự:

``` {.c}
struct foo {
    unsigned char a:2;
    unsigned char :5;   // <-- unnamed bit-field!
    unsigned char b:1;
};
```

### Bit-field không tên độ rộng zero

Thêm tí thứ bí hiểm ngoài đây... Giả sử bạn đang gói bit vào một
`unsigned int`, và bạn cần vài bit-field liền kề được gói vào
`unsigned int` _tiếp theo_.

Tức là, nếu bạn làm thế này:

``` {.c}
struct foo {
    unsigned int a:1;
    unsigned int b:2;
    unsigned int c:3;
    unsigned int d:4;
};
```

compiler gói hết tất cả vào một `unsigned int` duy nhất. Nhưng nếu
bạn cần `a` và `b` trong một `int`, và `c` và `d` trong một cái
khác?

Có giải pháp: đặt bit-field không tên độ rộng `0` ở chỗ bạn muốn
compiler bắt đầu lại việc gói bit vào `int` khác:

``` {.c}
struct foo {
    unsigned int a:1;
    unsigned int b:2;
    unsigned int :0;   // <--Zero-width unnamed bit-field
    unsigned int c:3;
    unsigned int d:4;
};
```

Nó tương tự như ngắt trang tường minh trong word processor. Bạn nói
với compiler: "Dừng gói bit vào `unsigned` này, và bắt đầu gói vào
cái tiếp theo."

Bằng cách thêm bit-field không tên độ rộng zero ở chỗ đó, compiler
đặt `a` và `b` vào một `unsigned int`, và `c` với `d` vào một
`unsigned int` khác. Tổng cộng hai, cỡ 8 byte trên hệ của tôi
(`unsigned int` mỗi cái 4 byte).

[i[`struct` keyword-->bit fields]>]

## Union

[i[`union` keyword]<]

Về cơ bản mấy cái này giống `struct`, chỉ khác là các field chồng
lên nhau trong bộ nhớ. `union` sẽ chỉ đủ lớn cho field lớn nhất, và
bạn chỉ dùng được một field mỗi lần.

Đây là cách tái sử dụng cùng không gian bộ nhớ cho các kiểu dữ liệu
khác nhau.

Bạn khai báo chúng y như `struct`, chỉ đổi sang `union`. Xem cái
này:

``` {.c}
union foo {
    int a, b, c, d, e, f;
    float g, h;
    char i, j, k, l;
};
```

Đấy, có một đống field. Nếu đây là `struct`, hệ tôi sẽ nói cần 36
byte để chứa tất cả.

Nhưng đây là `union`, nên mọi field đó chồng lên cùng đoạn bộ nhớ.
Field lớn nhất là `int` (hoặc `float`), chiếm 4 byte trên hệ tôi.
Và đúng thế, nếu tôi hỏi `sizeof` của `union foo`, nó nói 4!

Đánh đổi là bạn chỉ dùng được di động một trong các field đó mỗi
lần. Tuy nhiên...

### Union và type punning {#union-type-punning}

[i[`union` keyword-->type punning]<]

Bạn có thể ghi vào một field của `union` rồi đọc từ field khác,
nhưng không di động!

Làm vậy gọi là [flw[type punning|Type_punning]], và bạn dùng nó nếu
bạn thật sự biết mình đang làm gì, thường là trong kiểu lập trình
tầng thấp nào đó.

Vì các thành viên của union chia sẻ cùng bộ nhớ, ghi vào một thành
viên tất yếu ảnh hưởng các thành viên khác. Và nếu bạn đọc từ cái đã
được ghi vào cái khác, bạn sẽ có những hiệu ứng kỳ lạ.

``` {.c .numberLines}
#include <stdio.h>

union foo {
    float b;
    short a;
};

int main(void)
{
    union foo x;

    x.b = 3.14159;

    printf("%f\n", x.b);  // 3.14159, fair enough

    printf("%d\n", x.a);  // But what about this?
}
```

Trên hệ của tôi, cái này in ra:

```
3.141590
4048
```

vì dưới mui xe, biểu diễn đối tượng cho float `3.14159` giống hệt
biểu diễn đối tượng cho short `4048`. Trên hệ của tôi. Kết quả của
bạn có thể khác.

[i[`union` keyword-->type punning]>]

### Con trỏ tới `union`

[i[`union` keyword-->pointers to]<]

Nếu bạn có con trỏ tới `union`, bạn có thể ép kiểu con trỏ đó sang
bất kỳ kiểu nào của các field trong `union` đó và lấy giá trị ra
theo cách đó.

Trong ví dụ này, ta thấy `union` có `int` và `float` trong đó. Và
ta lấy con trỏ tới `union`, nhưng ép sang kiểu `int*` và `float*`
(cast để làm im compiler). Rồi nếu ta dereference chúng, ta thấy
chúng có giá trị ta đã lưu trực tiếp trong `union`.

``` {.c .numberLines}
#include <stdio.h>

union foo {
    int a, b, c, d, e, f;
    float g, h;
    char i, j, k, l;
};

int main(void)
{
    union foo x;

    int *foo_int_p = (int *)&x;
    float *foo_float_p = (float *)&x;

    x.a = 12;
    printf("%d\n", x.a);           // 12
    printf("%d\n", *foo_int_p);    // 12, again

    x.g = 3.141592;
    printf("%f\n", x.g);           // 3.141592
    printf("%f\n", *foo_float_p);  // 3.141592, again
}
```

Điều ngược lại cũng đúng. Nếu ta có con trỏ tới một kiểu bên trong
`union`, ta có thể ép sang con trỏ tới `union` và truy cập các thành
viên của nó.

``` {.c}
union foo x;
int *foo_int_p = (int *)&x;             // Pointer to int field
union foo *p = (union foo *)foo_int_p;  // Back to pointer to union

p->a = 12;  // This line the same as...
x.a = 12;   // this one.
```

Tất cả chuyện này chỉ cho bạn biết rằng, dưới mui xe, mọi giá trị
trong một `union` bắt đầu cùng chỗ trong bộ nhớ, và đó cũng là chỗ
cả `union` ở.

[i[`union` keyword-->pointers to]>]

### Common initial sequence trong union

[i[`union` keyword-->common initial sequences]<]

Nếu bạn có một `union` các `struct`, và tất cả `struct` đó bắt đầu
bằng một _common initial sequence_, truy cập các thành viên của
sequence đó từ bất kỳ thành viên nào của `union` là hợp lệ.

Gì cơ?

Đây là hai `struct` với common initial sequence:

``` {.c}
struct a {
	int x;     //
	float y;   // Common initial sequence

	char *p;
};

struct b {
	int x;     //
	float y;   // Common initial sequence

	double *p;
	short z;
};
```

Bạn thấy chưa? Là chúng bắt đầu bằng `int` tiếp theo là `float`, đó
là common initial sequence. Các thành viên trong sequence của các
`struct` phải là kiểu tương thích. Và ta thấy với `x` và `y`, là
`int` và `float`.

Giờ xây một union của mấy cái này:

``` {.c}
union foo {
	struct a sa;
	struct b sb;
};
```

Quy tắc này nói cho ta rằng ta được đảm bảo các thành viên của
common initial sequence có thể hoán đổi cho nhau trong code. Tức là:

* `f.sa.x` giống `f.sb.x`.

* `f.sa.y` giống `f.sb.y`.

Vì field `x` và `y` đều nằm trong common initial sequence.

Ngoài ra, tên của các thành viên trong common initial sequence không
quan trọng, chỉ quan trọng kiểu phải giống nhau.

Tất cả gộp lại cho ta cách an toàn để thêm vài thông tin chia sẻ
giữa các `struct` trong `union`. Ví dụ hay nhất của chuyện này có
lẽ là dùng một field để xác định kiểu `struct` nào trong tất cả các
`struct` của `union` đang "được dùng".

Tức là, nếu ta không được phép làm thế và ta truyền `union` cho một
hàm, hàm đó làm sao biết thành viên nào của `union` là cái nó nên
nhìn?

Xem các `struct` này. Để ý common initial sequence:

``` {.c .numberLines}
#include <stdio.h>

struct common {
	int type;   // common initial sequence
};

struct antelope {
	int type;   // common initial sequence

	int loudness;
};

struct octopus {
	int type;   // common initial sequence

	int sea_creature;
	float intelligence;
};

```

Giờ quẳng chúng vào `union`:

``` {.c .numberLines startFrom="20"}
union animal {
	struct common common;
	struct antelope antelope;
	struct octopus octopus;
};

```

Ngoài ra, làm ơn chiều tôi hai `#define` sau cho ví dụ:

``` {.c .numberLines startFrom="26"}
#define ANTELOPE 1
#define OCTOPUS  2

```

Tới giờ, chẳng có gì đặc biệt xảy ra ở đây. Có vẻ field `type` hoàn
toàn vô dụng.

Nhưng giờ ta làm hàm chung in `union animal`. Nó phải cách nào đó
biết được mình đang nhìn vào `struct antelope` hay `struct octopus`.

Nhờ phép thuật của common initial sequence, nó có thể tra kiểu
animal ở bất kỳ chỗ nào cho một `union animal x` cụ thể:

``` {.c}
int type = x.common.type;    // or...
int type = x.antelope.type;  // or...
int type = x.octopus.type;
```

Tất cả đều trỏ đến cùng giá trị trong bộ nhớ.

Và, như bạn có thể đoán, `struct common` ở đó để code có thể nhìn
kiểu một cách tổng quát mà không phải nhắc tới con vật cụ thể.

Xem code để in `union animal`:

``` {.c .numberLines startFrom="29"}
void print_animal(union animal *x)
{
	switch (x->common.type) {
		case ANTELOPE:
			printf("Antelope: loudness=%d\n", x->antelope.loudness);
			break;

		case OCTOPUS:
			printf("Octopus : sea_creature=%d\n", x->octopus.sea_creature);
			printf("          intelligence=%f\n", x->octopus.intelligence);
			break;
		
		default:
			printf("Unknown animal type\n");
	}

}

int main(void)
{
	union animal a = {.antelope.type=ANTELOPE, .antelope.loudness=12};
	union animal b = {.octopus.type=OCTOPUS, .octopus.sea_creature=1,
	                                   .octopus.intelligence=12.8};

	print_animal(&a);
	print_animal(&b);
}
```

Xem cách ở dòng 29 ta chỉ truyền vào `union`, ta không biết kiểu
animal `struct` nào đang được dùng bên trong.

Nhưng không sao! Vì ở dòng 31 ta kiểm tra kiểu xem là antelope hay
octopus. Rồi ta có thể nhìn vào đúng `struct` để lấy thành viên.

Hoàn toàn có thể có hiệu quả tương tự chỉ dùng `struct`, nhưng bạn
có thể làm thế này nếu muốn hiệu quả tiết kiệm bộ nhớ của `union`.

[i[`union` keyword-->common initial sequences]>]

## Union và struct vô danh

[i[`union` keyword-->and unnamed `struct`s]<]

Bạn biết cách có `struct` vô danh, thế này:

``` {.c}
struct {
    int x, y;
} s;
```

Cái đó định nghĩa biến `s` thuộc kiểu `struct` vô danh (vì `struct`
không có tag tên), với thành viên `x` và `y`.

Nên những chuyện kiểu này là hợp lệ:

``` {.c}
s.x = 34;
s.y = 90;

printf("%d %d\n", s.x, s.y);
```

Hóa ra bạn có thể thả `struct` vô danh vào `union` y như bạn nghĩ:

``` {.c}
union foo {
    struct {       // unnamed!
        int x, y;
    } a;

    struct {       // unnamed!
        int z, w;
    } b;
};
```

Rồi truy cập chúng như bình thường:

``` {.c}
union foo f;

f.a.x = 1;
f.a.y = 2;
f.b.z = 3;
f.b.w = 4;
```

Không sao!

[i[`union` keyword-->and unnamed `struct`s]>]

## Truyền và trả `struct` và `union`

[i[`union` keyword-->passing and returning]<]
[i[`struct` keyword-->passing and returning]<]

Bạn có thể truyền `struct` hoặc `union` cho hàm theo giá trị (thay
vì con trỏ tới nó), một bản sao của đối tượng đó sẽ được tạo cho
tham số như khi gán thông thường.

Bạn cũng có thể trả `struct` hoặc `union` từ hàm và nó cũng được
truyền ngược lại theo giá trị.

``` {.c .numberLines}
#include <stdio.h>

struct foo {
    int x, y;
};

struct foo f(void)
{
    return (struct foo){.x=34, .y=90};
}

int main(void)
{
    struct foo a = f();  // Copy is made

    printf("%d %d\n", a.x, a.y);
}
```

Chuyện vui: nếu làm thế, bạn có thể dùng toán tử `.` ngay trên lời
gọi hàm:

``` {.c .numberLines startFrom="16"}
    printf("%d %d\n", f().x, f().y);
```

(Dĩ nhiên ví dụ đó gọi hàm hai lần, không hiệu quả.)

Và điều tương tự đúng với việc trả con trỏ tới `struct` và `union`,
chỉ cần nhớ dùng toán tử mũi tên `->` trong trường hợp đó.

[i[`union` keyword-->passing and returning]>]
[i[`struct` keyword-->passing and returning]>]
[i[`union` keyword]>]
[i[`struct` keyword]>]

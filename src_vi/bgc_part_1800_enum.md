<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Kiểu liệt kê: `enum`

[i[`enum` enumerated types]<]

C cho ta thêm một cách nữa để có giá trị số nguyên hằng đặt tên:
`enum`.

Ví dụ:

``` {.c}
enum {
  ONE=1,
  TWO=2
};

printf("%d %d", ONE, TWO);  // 1 2
```

Ở vài chỗ, nó có thể tốt hơn, hoặc khác, so với dùng `#define`. Mấy
khác biệt chính:

* `enum` chỉ có thể là kiểu số nguyên.
* `#define` thì định nghĩa được bất cứ thứ gì.
* `enum` thường hiện ra bằng tên ký hiệu trong debugger.
* Số `#define` chỉ hiện ra dưới dạng số thô, khó biết ý nghĩa lúc
  debug.

Vì chúng là kiểu số nguyên, chúng dùng được ở bất cứ đâu mà số
nguyên dùng được, kể cả kích thước mảng và câu `case`.

Mổ xẻ thêm nhé.

## Hành vi của `enum`

### Đánh số

[i[`enum` enumerated types-->numbering order]<]

`enum` được đánh số tự động trừ khi bạn ghi đè.

Chúng bắt đầu từ `0`, rồi tự tăng dần lên, theo mặc định:

``` {.c}
enum {
    SHEEP,  // Value is 0
    WHEAT,  // Value is 1
    WOOD,   // Value is 2
    BRICK,  // Value is 3
    ORE     // Value is 4
};

printf("%d %d\n", SHEEP, BRICK);  // 0 3
```

Bạn có thể ép giá trị số nguyên cụ thể, như ta đã thấy ở trên:

``` {.c}
enum {
  X=2,
  Y=18,
  Z=-2
};
```

Trùng giá trị cũng không sao:

``` {.c}
enum {
  X=2,
  Y=2,
  Z=2
};
```

Nếu giá trị bị bỏ đi, việc đánh số tiếp tục đếm theo hướng dương từ
giá trị nào được chỉ định gần nhất. Ví dụ:

``` {.c}
enum {
  A,    // 0, default starting value
  B,    // 1
  C=4,  // 4, manually set
  D,    // 5
  E,    // 6
  F=3,  // 3, manually set
  G,    // 4
  H     // 5
}
```

[i[`enum` enumerated types-->numbering order]>]

### Dấu phẩy đuôi

Cái này hoàn toàn ổn, nếu bạn thích kiểu đó:

``` {.c}
enum {
  X=2,
  Y=18,
  Z=-2,   // <-- Trailing comma
};
```

Mấy thập kỷ gần đây nó phổ biến hơn trong các ngôn ngữ khác, nên bạn
có thể thấy vui khi gặp lại.

### Phạm vi

[i[`enum` enumerated types-->scope]<]

`enum` có scope đúng như bạn kỳ vọng. Nếu ở file scope, cả file thấy
nó. Nếu trong một block, nó cục bộ trong block đó.

Rất thường gặp chuyện `enum` được định nghĩa trong file header để có
thể `#include` vào ở file scope.

[i[`enum` enumerated types-->scope]>]

### Style

Như bạn đã để ý, rất phổ biến việc khai báo ký hiệu `enum` bằng chữ
hoa (với gạch dưới).

Đây không phải yêu cầu, nhưng là một idiom rất, rất phổ biến.

## `enum` của bạn là một kiểu

Đây là chuyện quan trọng cần biết về `enum`: chúng là một kiểu,
tương tự cách `struct` là một kiểu.

Bạn có thể gán cho chúng một tên tag để về sau tham chiếu kiểu và
khai báo biến của kiểu đó.

Mà này, vì `enum` là kiểu số nguyên, sao không xài luôn `int`?

Trong C, lý do tốt nhất là để code rõ ràng, đây là cách đẹp, có
kiểu, để diễn tả suy nghĩ của bạn trong code. C (khác với C++) thật
ra không ép giá trị phải nằm trong phạm vi của một `enum` cụ thể.

Làm ví dụ khai báo biến `r` kiểu `enum resource` có thể giữ các giá
trị đó:

``` {.c}
// Named enum, type is "enum resource"

enum resource {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
};

// Declare a variable "r" of type "enum resource"

enum resource r = BRICK;

if (r == BRICK) {
    printf("I'll trade you a brick for two sheep.\n");
}
```

Bạn cũng có thể `typedef` cái này, dĩ nhiên, dù cá nhân tôi không
thích.


``` {.c}
typedef enum {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
} RESOURCE;

RESOURCE r = BRICK;
```

Một lối tắt khác hợp lệ nhưng hiếm là khai báo biến ngay khi khai
báo `enum`:

``` {.c}
// Declare an enum and some initialized variables of that type:

enum {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
} r = BRICK, s = WOOD;
```

Bạn cũng có thể đặt tên cho `enum` để về sau dùng lại, đây có lẽ là
điều bạn muốn làm trong phần lớn trường hợp:


``` {.c}
// Declare an enum and some initialized variables of that type:

enum resource {   // <-- type is now "enum resource"
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
} r = BRICK, s = WOOD;
```

Ngắn gọn, `enum` là cách hay để viết code đẹp, có scope, có kiểu,
sạch sẽ.

[i[`enum` enumerated types]>]

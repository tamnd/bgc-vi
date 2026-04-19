<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Pointer III: Pointer tới pointer và hơn thế

Đây là chỗ ta nói về cách dùng pointer ở mức trung cấp và nâng cao.
Nếu bạn chưa nắm chắc pointer, xem lại các chương trước về
[pointer](#pointers) và [pointer arithmetic](#pointers2) trước khi
bắt đầu mấy thứ này.

## Pointer tới pointer

[i[Pointers-->to pointers]<]

Nếu bạn có thể có pointer tới một biến, và một biến có thể là
pointer, thì bạn có thể có pointer tới một biến mà bản thân nó là
pointer không?

Có chứ! Cái này là pointer tới pointer, và nó được giữ trong biến
kiểu pointer-pointer.

Trước khi mổ xẻ, tôi muốn cố tạo _cảm giác trực giác_ về cách
pointer tới pointer hoạt động.

Nhớ là pointer chỉ là một con số. Nó là con số đại diện cho một chỉ
số trong bộ nhớ máy tính, thường là chỉ số giữ một giá trị mà ta
đang quan tâm vì lý do nào đó.

Cái pointer đó, là một con số, thì cũng phải được lưu ở đâu đó. Và
chỗ đó là bộ nhớ, như mọi thứ khác^[Có chút lắt léo với giá trị được
lưu chỉ trong thanh ghi, nhưng ở đây ta có thể bỏ qua an toàn. Với
lại spec C không có quan điểm gì về mấy chuyện "thanh ghi" đó ngoài
từ khóa `register`, mà phần mô tả của nó cũng không nhắc tới thanh
ghi.].

Nhưng vì nó được lưu trong bộ nhớ, nó phải có một chỉ số nơi nó được
lưu, đúng không? Cái pointer đó phải có một chỉ số trong bộ nhớ nơi
nó nằm. Và chỉ số đó là một con số. Đó là địa chỉ của pointer. Đó là
pointer tới pointer.

Hãy bắt đầu với một pointer thường tới `int`, trở lại từ các chương
trước:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int x = 3490;  // Type: int
    int *p = &x;   // Type: pointer to an int

    printf("%d\n", *p);  // 3490
}
```

Khá đơn giản, nhỉ? Ta có hai kiểu được đại diện: `int` và `int*`, và
ta dựng `p` để trỏ tới `x`. Rồi ta có thể dereference `p` ở dòng 8
và in ra giá trị `3490`.

Nhưng, như ta đã nói, ta có thể có pointer tới bất kỳ biến nào, vậy
có phải nghĩa là ta có thể có pointer tới `p` không?

Nói cách khác, biểu thức này có kiểu gì?


``` {.c}
int x = 3490;  // Type: int
int *p = &x;   // Type: pointer to an int

&p  // <-- What type is the address of p? AKA a pointer to p?
```

Nếu `x` là `int`, thì `&x` là pointer tới `int` mà ta đã lưu trong
`p` có kiểu `int*`. Theo được chứ? (Đọc lại đoạn này cho đến khi
hiểu!)

Và do đó `&p` là pointer tới `int*`, hay còn gọi là "pointer tới
pointer tới `int`". Hay là "`int`-pointer-pointer".

Hiểu rồi chứ? (Đọc lại đoạn trước cho đến khi hiểu!)

Ta viết kiểu này với hai dấu sao: `int **`. Xem nó trong hành động.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int x = 3490;  // Type: int
    int *p = &x;   // Type: pointer to an int
    int **q = &p;  // Type: pointer to pointer to int

    printf("%d %d\n", *p, **q);  // 3490 3490
}
```

Ta bịa ra vài địa chỉ giả cho các giá trị trên làm ví dụ và xem ba
biến đó có thể trông thế nào trong bộ nhớ. Các giá trị địa chỉ dưới
đây do tôi bịa cho có ví dụ:

|Biến|Lưu tại địa chỉ|Giá trị lưu ở đó|
|-|-|-|
|`x`|`28350`|`3490`, giá trị từ code|
|`p`|`29122`|`28350`, địa chỉ của `x`!|
|`q`|`30840`|`29122`, địa chỉ của `p`!|

Thật vậy, hãy thử thật trên máy tôi^[Trên máy bạn rất có thể ra số
khác.] và in các giá trị pointer bằng `%p` và tôi sẽ lập bảng đó lại
với tham chiếu thật (in bằng hex).

|Biến|Lưu tại địa chỉ|Giá trị lưu ở đó|
|-|-|-|
|`x`|`0x7ffd96a07b94`|`3490`, giá trị từ code|
|`p`|`0x7ffd96a07b98`|`0x7ffd96a07b94`, địa chỉ của `x`!|
|`q`|`0x7ffd96a07ba0`|`0x7ffd96a07b98`, địa chỉ của `p`!|

Bạn có thể thấy mấy địa chỉ này giống nhau trừ byte cuối, nên chỉ
cần để ý byte đó.

Trên hệ của tôi, `int` chiếm 4 byte, vì vậy ta thấy địa chỉ tăng
thêm 4 từ `x` sang `p`^[Spec không nói gì chuyện cái này sẽ luôn
hoạt động kiểu này, nhưng tình cờ trên máy tôi nó vậy.] rồi tăng
thêm 8 từ `p` sang `q`. Trên hệ của tôi, mọi pointer chiếm 8 byte.

Có quan trọng chuyện nó là `int*` hay `int**` không? Cái nào nhiều
byte hơn cái nào? Không hề! Nhớ rằng mọi pointer đều là địa chỉ, tức
là chỉ số vào bộ nhớ. Và trên máy tôi ta có thể biểu diễn một chỉ số
bằng 8 byte, chẳng liên quan cái gì được lưu ở chỉ số đó.

Giờ để ý xem ta đã làm gì ở dòng 9 của ví dụ trước: ta _dereference
hai lần_ `q` để quay lại được `3490`.

Đây là điểm quan trọng về pointer và pointer tới pointer:

* Bạn có thể lấy pointer tới bất cứ thứ gì bằng `&` (kể cả tới một
  pointer!)
* Bạn có thể lấy thứ mà một pointer đang trỏ tới bằng `*` (kể cả
  một pointer!)

Vậy bạn có thể nghĩ `&` được dùng để tạo pointer, còn `*` thì ngược
lại, đi theo chiều ngược với `&`, để tới được thứ được trỏ tới.

Về kiểu, mỗi lần bạn `&`, cái đó thêm một mức pointer vào kiểu.

|Nếu bạn có|Rồi bạn chạy|Kiểu kết quả là|
|:-|:-:|:-|
|`int x`|`&x`|`int *`|
|`int *x`|`&x`|`int **`|
|`int **x`|`&x`|`int ***`|
|`int ***x`|`&x`|`int ****`|

Và mỗi lần bạn dereference (`*`), nó làm ngược lại:

|Nếu bạn có|Rồi bạn chạy|Kiểu kết quả là|
|:-|:-:|:-|
|`int ****x`|`*x`|`int ***`|
|`int ***x`|`*x`|`int **`|
|`int **x`|`*x`|`int *`|
|`int *x`|`*x`|`int`|

Lưu ý bạn có thể dùng nhiều `*` liên tiếp để dereference nhanh, y
như trong ví dụ code với `**q` ở trên. Mỗi cái bóc đi một lớp gián
tiếp.

|Nếu bạn có|Rồi bạn chạy|Kiểu kết quả là|
|:-|:-:|:-|
|`int ****x`|`***x`|`int *`|
|`int ***x`|`**x`|`int *`|
|`int **x`|`**x`|`int`|

Tổng quát, `&*E == E`^[Kể cả khi `E` là `NULL`, lạ thay.].
Dereference "hoàn tác" address-of.

Nhưng `&` thì không chạy kiểu đó, bạn chỉ làm từng cái một được, và
phải lưu kết quả vào một biến trung gian:

``` {.c}
int x = 3490;     // Type: int
int *p = &x;      // Type: int *, pointer to an int
int **q = &p;     // Type: int **, pointer to pointer to int
int ***r = &q;    // Type: int ***, pointer to pointer to pointer to int
int ****s = &r;   // Type: int ****, you get the idea
int *****t = &s;  // Type: int *****
```

[i[Pointers-->to pointers]>]

### Pointer-pointer và `const`

[i[Pointers-->to pointers, `const`]<]

Nếu bạn còn nhớ, khai báo pointer kiểu vầy:

``` {.c}
int *const p;
```

nghĩa là bạn không thể sửa `p`. Cố `p++` sẽ cho lỗi lúc compile.

Nhưng nó hoạt động ra sao với `int **` hay `int ***`? `const` đi đâu
và nghĩa là gì?

Bắt đầu từ phần đơn giản. `const` ngay cạnh tên biến ám chỉ chính
biến đó. Vậy nếu bạn muốn một `int***` mà không thể đổi, bạn có thể
làm vầy:

``` {.c}
int ***const p;

p++;  // Not allowed
```

Nhưng đây là chỗ mọi thứ hơi lạ.

Lỡ ta gặp tình huống này thì sao:

``` {.c .numberLines}
int main(void)
{
    int x = 3490;
    int *const p = &x;
    int **q = &p;
}
```

Khi tôi build cái đó, tôi nhận được cảnh báo:

``` {.default}
warning: initialization discards ‘const’ qualifier from pointer target type
    7 |     int **q = &p;
      |               ^
```

Chuyện gì vậy? Compiler đang báo ta rằng ta có một biến `const`, và
ta đang gán giá trị của nó vào biến khác không `const` theo cùng
cách. Tính "`const`" bị bỏ đi, mà có lẽ đó không phải thứ ta muốn.

Kiểu của `p` là `int *const p`, nên `&p` có kiểu `int *const *`. Và
ta cố gán cái đó vào `q`.

Nhưng `q` là `int **`! Một kiểu với tính `const` khác ở dấu `*` đầu
tiên! Nên ta nhận cảnh báo là `const` trong `int *const *` của `p`
đang bị bỏ qua và vứt đi.

Ta có thể sửa bằng cách đảm bảo kiểu của `q` ít nhất cũng `const`
bằng `p`.

``` {.c}
int x = 3490;
int *const p = &x;
int *const *q = &p;
```

Và giờ ta vui rồi.

Ta có thể làm `q` `const` hơn nữa. Hiện tại ở trên, ta đang nói,
"`q` bản thân không `const`, nhưng thứ nó trỏ tới là `const`." Nhưng
ta có thể làm cả hai `const`:

``` {.c}
int x = 3490;
int *const p = &x;
int *const *const q = &p;  // More const!
```

Và chạy ngon. Giờ ta không thể sửa `q`, hay cái pointer mà `q` trỏ
tới.

[i[Pointers-->to pointers, `const`]>]

## Giá trị nhiều byte {#multibyte-values}

[i[Pointers-->to multibyte values]<]

Ta đã bóng gió chuyện này ở vài chỗ trước đây, nhưng rõ ràng không
phải mọi giá trị đều lưu vừa trong một byte bộ nhớ. Mọi thứ chiếm
nhiều byte bộ nhớ (giả sử chúng không phải `char`). Bạn có thể biết
chiếm bao nhiêu byte bằng `sizeof`. Và bạn có thể biết địa chỉ nào
trong bộ nhớ là byte _đầu tiên_ của đối tượng bằng toán tử chuẩn
`&`, luôn trả về địa chỉ của byte đầu.

Và đây là sự thật thú vị khác! Nếu bạn duyệt qua các byte của bất kỳ
đối tượng nào, bạn nhận được _biểu diễn đối tượng_ của nó. Hai thứ
có cùng biểu diễn đối tượng trong bộ nhớ thì bằng nhau.

Nếu bạn muốn duyệt biểu diễn đối tượng, bạn nên làm với pointer tới
`unsigned char`.

Hãy làm phiên bản riêng của
[fl[`memcpy()`|https://beej.us/guide/bgclr/html/split/stringref.html#man-memcpy]]
làm đúng chuyện này:

``` {.c}
void *my_memcpy(void *dest, const void *src, size_t n)
{
    // Make local variables for src and dest, but of type unsigned char

    const unsigned char *s = src;
    unsigned char *d = dest;

    while (n-- > 0)   // For the given number of bytes
        *d++ = *s++;  // Copy source byte to dest byte

    // Most copy functions return a pointer to the dest as a convenience
    // to the caller

    return dest;
}
```

(Trong đó cũng có ví dụ post-increment và post-decrement hay để bạn
nghiên cứu.)

Quan trọng mà lưu ý là phiên bản ở trên có lẽ kém hiệu quả hơn cái
đi kèm với hệ của bạn.

Nhưng bạn có thể truyền pointer tới bất cứ thứ gì vào nó, và nó sẽ
copy các đối tượng đó. Có thể là `int*`, `struct animal*`, hay bất
cứ gì.

Làm thêm ví dụ nữa in ra các byte biểu diễn đối tượng của một
`struct` để ta xem có padding trong đó không và nó có giá trị
gì^[Compiler C không bị bắt buộc phải chèn byte padding, và giá trị
của bất cứ byte padding nào được chèn là không xác định.].

``` {.c .numberLines}
#include <stdio.h>

struct foo {
    char a;
    int b;
};

int main(void)
{
    struct foo x = {0x12, 0x12345678};
    unsigned char *p = (unsigned char *)&x;

    for (size_t i = 0; i < sizeof x; i++) {
        printf("%02X\n", p[i]);
    }
}
```

Cái ta có đó là `struct foo` được dựng sao cho khuyến khích compiler
chèn byte padding vào (dù nó không phải làm). Rồi ta lấy `unsigned
char *` tới byte đầu của biến `struct foo` tên `x`.

Từ đó, ta chỉ cần biết `sizeof x` là có thể lặp qua từng ấy byte, in
các giá trị ra (bằng hex cho dễ đọc).

Chạy cái này cho ra một đống số. Tôi chú thích bên dưới để chỉ ra
các giá trị được lưu ở đâu:

``` {.default}
12  | x.a == 0x12

AB  |
BF  | padding bytes with "random" value
26  |

78  | x.b == 0x12345678
56  |
34  |
12  |
```

Trên mọi hệ, `sizeof(char)` là 1, và ta thấy byte đầu tiên ở đầu
output giữ giá trị `0x12` mà ta đã lưu ở đó.

Rồi ta có vài byte padding, với tôi, mấy cái này thay đổi giữa các
lần chạy.

Cuối cùng, trên hệ của tôi, `sizeof(int)` là 4, và ta thấy 4 byte đó
ở cuối. Chú ý chúng chính là các byte trong giá trị hex `0x12345678`,
nhưng kỳ lạ là ngược thứ tự^[Cái này sẽ khác nhau tùy kiến trúc,
nhưng hệ của tôi _little endian_, nghĩa là byte nhỏ nhất của số được
lưu trước. Hệ _big endian_ sẽ có `12` trước và `78` sau. Nhưng spec
không ra lệnh gì về biểu diễn này.].

Vậy đó là một cái nhìn hé lộ vào các byte của một thực thể phức tạp
hơn trong bộ nhớ.

[i[Pointers-->to multibyte values]>]

## Pointer `NULL` và số 0

[i[`NULL` pointer-->zero equivalence]<]

Mấy thứ này có thể dùng thay qua lại:

* `NULL`
* `0`
* `'\0'`
* `(void *)0`

Cá nhân tôi, tôi luôn dùng `NULL` khi tôi muốn nói `NULL`, nhưng bạn
có thể thỉnh thoảng thấy các biến thể khác. Dù `'\0'` (byte với mọi
bit bằng 0) cũng so sánh ra bằng, nhưng đem so nó với một pointer
thì _kỳ cục_, bạn nên so `NULL` với pointer. (Dĩ nhiên, nhiều lần
khi xử lý chuỗi, bạn so sánh _cái mà pointer đang trỏ tới_ với
`'\0'`, và cái đó thì đúng.)

`0` được gọi là _null pointer constant_, và khi so sánh hay gán vào
một pointer khác, nó được chuyển thành null pointer cùng kiểu.

[i[`NULL` pointer-->zero equivalence]>]

## Pointer như số nguyên

[i[Pointers-->as integers]<]

Bạn có thể cast pointer thành số nguyên và ngược lại (vì pointer chỉ
là chỉ số vào bộ nhớ), nhưng có lẽ bạn chỉ cần làm điều này nếu đang
làm mấy thứ phần cứng mức thấp. Kết quả của những trò như vậy là
implementation-defined, nên không portable. Và _chuyện lạ_ có thể
xảy ra.

Tuy nhiên, C có đảm bảo một chuyện: bạn có thể chuyển một pointer
sang kiểu `uintptr_t` và bạn sẽ có thể chuyển nó về lại pointer mà
không mất dữ liệu.

`uintptr_t` được định nghĩa trong `<stdint.h>`^[Đây là tính năng tùy
chọn, nên nó có thể không có, nhưng có lẽ có.].

Thêm nữa, nếu bạn muốn có dấu, bạn có thể dùng `intptr_t` với tác
dụng y vậy.

[i[Pointers-->as integers]>]

## Cast pointer sang pointer khác

[i[Pointers-->casting]<]

Chỉ có một cách chuyển pointer an toàn:

1. Chuyển sang `intptr_t` hoặc `uintptr_t`.
2. Chuyển sang và từ `void*`.

HAI! Có hai cách chuyển pointer an toàn.

3. Chuyển sang và từ `char*` (hoặc `signed char*`/`unsigned char*`).

BA! Có ba cách chuyển an toàn!

4. Chuyển sang và từ pointer tới `struct` và pointer tới thành viên
   đầu tiên của nó, và ngược lại.

BỐN! Có bốn cách chuyển an toàn!

Nếu bạn cast sang pointer kiểu khác rồi truy cập đối tượng nó trỏ
tới, hành vi là không xác định do một thứ gọi là _strict aliasing_.

_Aliasing_ thuần túy nghĩa là khả năng có nhiều hơn một cách để truy
cập cùng một đối tượng. Các điểm truy cập là alias của nhau.

_Strict aliasing_ nói bạn chỉ được phép truy cập một đối tượng qua
pointer tới _kiểu tương thích_ với đối tượng đó.

Ví dụ, cái này chắc chắn được phép:

``` {.c}
int a = 1;
int *p = &a;
```

`p` là pointer tới `int`, và nó trỏ tới một kiểu tương thích, cụ thể
là `int`, nên ta ngon.

Nhưng cái dưới không ổn vì `int` và `float` không phải là các kiểu
tương thích:

``` {.c}
int a = 1;
float *p = (float *)&a;
```

Đây là chương trình demo làm chút aliasing. Nó lấy biến `v` kiểu
`int32_t` và alias nó thành pointer tới một `struct words`. `struct`
đó có hai `int16_t`. Các kiểu này không tương thích, nên ta đang vi
phạm luật strict aliasing. Compiler sẽ giả định rằng hai pointer này
không bao giờ trỏ tới cùng một đối tượng, nhưng ta đang làm cho
chúng trỏ tới. Điều đó thật hư của ta.

Hãy xem có làm vỡ được gì không.

``` {.c .numberLines}
#include <stdio.h>
#include <stdint.h>

struct words {
    int16_t v[2];
};

void fun(int32_t *pv, struct words *pw)
{
    for (int i = 0; i < 5; i++) {
        (*pv)++;

        // Print the 32-bit value and the 16-bit values:

        printf("%x, %x-%x\n", *pv, pw->v[1], pw->v[0]);
    }
}

int main(void)
{
    int32_t v = 0x12345678;

    struct words *pw = (struct words *)&v;  // Violates strict aliasing

    fun(&v, pw);
}
```

Thấy cách tôi truyền hai pointer không tương thích vào `fun()` chứ?
Một kiểu là `int32_t*` và cái kia là `struct words*`.

Nhưng cả hai trỏ tới cùng đối tượng: giá trị 32-bit được khởi tạo
bằng `0x12345678`.

Vậy nếu ta nhìn các field trong `struct words`, ta sẽ thấy hai nửa
16-bit của con số đó. Đúng không?

Và trong vòng lặp `fun()`, ta tăng pointer tới `int32_t`. Chỉ vậy.
Nhưng vì `struct` trỏ tới cùng bộ nhớ đó, nó cũng sẽ được cập nhật
thành cùng giá trị.

Vậy chạy thử và ta nhận được cái này, với giá trị 32-bit bên trái và
hai phần 16-bit bên phải. Phải khớp nhau^[Tôi in hai giá trị 16-bit
theo thứ tự đảo vì tôi đang ở máy little-endian và làm vậy dễ đọc
hơn ở đây.]:

``` {.default}
12345679, 1234-5679
1234567a, 1234-567a
1234567b, 1234-567b
1234567c, 1234-567c
1234567d, 1234-567d
```

và nó có khớp... _CHO TỚI NGÀY MAI!_

Thử compile bằng GCC với `-O3` và `-fstrict-aliasing`:

``` {.default}
12345679, 1234-5678
1234567a, 1234-5679
1234567b, 1234-567a
1234567c, 1234-567b
1234567d, 1234-567c
```

Lệch nhau một đơn vị! Mà chúng trỏ tới cùng bộ nhớ! Sao có thể?
Giải đáp: aliasing bộ nhớ kiểu đó là hành vi không xác định. _Bất cứ
chuyện gì cũng có thể xảy ra_, nhưng không phải theo nghĩa tốt.

Nếu code của bạn vi phạm luật strict aliasing, việc nó chạy hay
không phụ thuộc vào cách ai đó quyết định compile nó. Và đó là điều
đáng tiếc vì nó ngoài tầm kiểm soát của bạn. Trừ khi bạn là một thứ
thần thánh toàn năng nào đó.

Khó có khả năng vậy, tiếc.

GCC có thể bị ép không dùng luật strict aliasing bằng
`-fno-strict-aliasing`. Compile chương trình demo ở trên với `-O3`
và cờ này khiến output đúng như mong đợi.

Cuối cùng, _type punning_ là dùng pointer của các kiểu khác nhau để
nhìn cùng dữ liệu. Trước khi có strict aliasing, mấy chuyện kiểu này
khá phổ biến:

``` {.c}
int a = 0x12345678;
short b = *((short *)&a);   // Violates strict aliasing
```

Nếu bạn muốn làm type punning (tương đối) an toàn, xem phần
[Union và Type Punning](#union-type-punning).

[i[Pointers-->casting]>]

## Hiệu của pointer {#ptr_differences}

[i[Pointers-->subtracting]<]

Như bạn đã biết từ phần pointer arithmetic, bạn có thể trừ một
pointer từ pointer khác^[Giả sử chúng trỏ tới cùng một đối tượng
mảng.] để có hiệu giữa chúng theo số phần tử mảng.

Giờ _kiểu của cái hiệu đó_ tùy implementation quyết định, nên có thể
khác nhau giữa các hệ.

[i[`ptrdiff_t` type]<]

Để portable hơn, bạn có thể lưu kết quả vào biến kiểu `ptrdiff_t`
định nghĩa trong `<stddef.h>`.

``` {.c}
int cats[100];

int *f = cats + 20;
int *g = cats + 60;

ptrdiff_t d = g - f;  // difference is 40
```

[i[`ptrdiff_t` type-->printing]<]
Và bạn có thể in nó bằng cách thêm `t` đầu format specifier cho số
nguyên:

``` {.c}
printf("%td\n", d);  // Print decimal: 40
printf("%tX\n", d);  // Print hex:     28
```

[i[`ptrdiff_t` type-->printing]>]
[i[`ptrdiff_t` type]>]
[i[Pointers-->subtracting]>]

## Pointer tới hàm

[i[Pointers-->to functions]<]

Hàm chỉ là tập hợp lệnh máy trong bộ nhớ, nên không có lý do gì ta
không lấy được pointer tới lệnh đầu tiên của hàm.

Và rồi gọi nó.

Điều này có thể hữu ích khi truyền một pointer tới hàm vào một hàm
khác như đối số. Rồi hàm thứ hai có thể gọi bất cứ cái gì được truyền
vào.

Tuy nhiên, phần khó với mấy cái này là C cần biết kiểu của biến là
pointer tới hàm.

Và nó thật sự muốn biết mọi chi tiết.

Kiểu như "đây là pointer tới hàm nhận hai đối số `int` và trả về
`void`".

Viết hết mớ đó ra sao để khai báo được biến?

Hóa ra nó trông rất giống function prototype, chỉ thêm vài cặp ngoặc:

``` {.c}
// Declare p to be a pointer to a function.
// This function returns a float, and takes two ints as arguments.

float (*p)(int, int);
```

Lưu ý bạn không cần đặt tên cho tham số. Nhưng bạn có thể nếu muốn,
chúng chỉ bị bỏ qua.

``` {.c}
// Declare p to be a pointer to a function.
// This function returns a float, and takes two ints as arguments.

float (*p)(int a, int b);
```

Giờ ta đã biết cách khai báo biến, làm sao biết gán gì vào? Làm sao
lấy địa chỉ của một hàm?

Hóa ra có lối tắt y như lấy pointer tới mảng: bạn có thể chỉ viết
tên hàm trần không có ngoặc. (Bạn có thể thêm `&` đằng trước nếu
thích, nhưng không cần và không idiomatic.)

Một khi có pointer tới hàm, bạn có thể gọi nó bằng cách thêm ngoặc
và danh sách đối số.

Làm ví dụ đơn giản tôi đặt hẳn alias cho hàm bằng cách dựng một
pointer tới nó. Rồi ta gọi nó.

Code này in ra `3490`:

``` {.c .numberLines}
#include <stdio.h>

void print_int(int n)
{
    printf("%d\n", n);
}

int main(void)
{
    // Assign p to point to print_int:

    void (*p)(int) = print_int;

    p(3490);          // Call print_int via the pointer
}
```

Lưu ý cách kiểu của `p` đại diện cho giá trị trả về và kiểu tham số
của `print_int`. Bắt buộc phải thế, không thì C sẽ phàn nàn về kiểu
pointer không tương thích.

Thêm một ví dụ nữa cho thấy ta có thể truyền pointer tới hàm như đối
số cho hàm khác thế nào.

Ta sẽ viết hàm nhận vài đối số số nguyên, cộng với pointer tới hàm
nào đó thao tác trên hai đối số đó. Rồi nó in kết quả.

``` {.c .numberLines}
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int mult(int a, int b)
{
    return a * b;
}

void print_math(int (*op)(int, int), int x, int y)
{
    int result = op(x, y);

    printf("%d\n", result);
}

int main(void)
{
    print_math(add, 5, 7);   // 12
    print_math(mult, 5, 7);  // 35
}
```

Dành tí thời gian tiêu hóa chuyện đó. Ý ở đây là ta sẽ truyền một
pointer tới hàm vào `print_math()`, và nó sẽ gọi hàm đó để làm vài
phép toán.

Bằng cách này ta có thể đổi hành vi của `print_math()` bằng cách
truyền hàm khác vào. Bạn thấy ta làm thế ở dòng 22-23 khi truyền vào
pointer tới hàm `add` và `mult`, theo thứ tự.

Giờ, ở dòng 13, tôi nghĩ ai cũng đồng ý signature của `print_math()`
là một cảnh ngoạn mục. Và, tin hay không, cái này thực ra còn khá
thẳng thớm so với vài thứ bạn có thể dựng ra^[Ngôn ngữ Go lấy cảm
hứng cú pháp khai báo kiểu từ điều ngược lại với cái C làm.].

Nhưng hãy tiêu hóa nó. Hóa ra chỉ có ba tham số, nhưng chúng hơi
khó thấy:

``` {.c}
//                      op             x      y
//              |-----------------|  |---|  |---|
void print_math(int (*op)(int, int), int x, int y)
```

Cái đầu, `op`, là pointer tới hàm nhận hai `int` làm đối số và trả
về `int`. Cái này khớp signature của cả `add()` lẫn `mult()`.

Cái thứ hai và ba, `x` và `y`, chỉ là tham số `int` chuẩn.

Chậm và có chủ đích, hãy để mắt bạn đi qua signature rồi xác định
các phần. Một thứ luôn nhảy ra với tôi là chuỗi `(*op)(`, cặp ngoặc
và dấu sao. Đó là dấu hiệu nó là pointer tới hàm.

Cuối cùng, nhảy lại chương _Pointer II_ để xem ví dụ
pointer-tới-hàm [dùng `qsort()` có sẵn](#qsort-example).

[i[Pointers-->to functions]>]

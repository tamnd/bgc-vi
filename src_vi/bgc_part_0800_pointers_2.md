<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Pointers II: Số học con trỏ {#pointers2}

[i[Pointers-->arithmetic]<]

Đến lúc lao vào sâu hơn với một loạt chủ đề mới về con trỏ! Nếu bạn
chưa nắm vững con trỏ, [xem lại mục đầu trong sách về chủ đề
này](#pointers).

## Số học con trỏ

Hoá ra bạn có thể làm toán trên con trỏ, cụ thể là cộng và trừ.

Nhưng như thế có nghĩa là gì?

Ngắn gọn, nếu bạn có con trỏ tới một kiểu, cộng 1 vào con trỏ sẽ
chuyển nó tới item kế tiếp của kiểu đó nằm ngay sau trong bộ nhớ.

Điều **quan trọng** cần nhớ là khi di chuyển con trỏ và nhìn vào các
chỗ khác nhau trong bộ nhớ, ta phải đảm bảo con trỏ luôn trỏ đến một
chỗ hợp lệ trước khi dereference. Nếu đang lang thang đâu đó ngoài
đồng cỏ và cố xem ở đó có gì, hành vi là undefined và chương trình
thường sẽ crash.

Chuyện này hơi kiểu gà-với-trứng so với [Array/Pointer Equivalence ở
dưới](#arraypointerequiv), nhưng ta vẫn sẽ thử.

### Cộng vào con trỏ

Đầu tiên, lấy một mảng số.

``` {.c}
int a[5] = {11, 22, 33, 44, 55};
```

Rồi lấy con trỏ tới phần tử đầu tiên của mảng:

``` {.c}
int a[5] = {11, 22, 33, 44, 55};

int *p = &a[0];  // Or "int *p = a;" works just as well
```

Rồi in giá trị ở đó bằng cách dereference con trỏ:

``` {.c}
printf("%d\n", *p);  // Prints 11
```

Giờ dùng số học con trỏ để in phần tử kế tiếp trong mảng, phần tử ở
index 1:

``` {.c}
printf("%d\n", *(p + 1));  // Prints 22!!
```

Chuyện gì vừa xảy ra? C biết `p` là con trỏ tới một `int`. Nó biết
`sizeof` của một `int`^[Nhớ rằng toán tử `sizeof` cho biết kích cỡ
tính bằng byte của một đối tượng trong bộ nhớ.] và biết phải nhảy bao
nhiêu byte để tới `int` kế tiếp sau cái đầu!

Thực ra, ví dụ trước có thể viết hai cách tương đương:

``` {.c}
printf("%d\n", *p);        // Prints 11
printf("%d\n", *(p + 0));  // Prints 11
```

vì cộng `0` vào con trỏ cho ra cùng một con trỏ.

Rút ra gì? Ta có thể duyệt các phần tử của mảng theo cách này thay vì
dùng mảng:

``` {.c}
int a[5] = {11, 22, 33, 44, 55};

int *p = &a[0];  // Or "int *p = a;" works just as well

for (int i = 0; i < 5; i++) {
    printf("%d\n", *(p + i));  // Same as p[i]!
}
```

Và nó chạy giống hệt như dùng ký hiệu mảng! Oooo! Đến gần hơn với
array/pointer equivalence rồi! Sẽ nói thêm ở phần sau của chương.

Nhưng thực chất chuyện gì đang xảy ra ở đây? Nó hoạt động thế nào?

Nhớ từ đầu rằng bộ nhớ giống như một mảng lớn, mỗi index của mảng lưu
một byte?

Và index vào bộ nhớ có vài tên gọi:

* Index vào bộ nhớ
* Location
* Address (địa chỉ)
* _Pointer!_ (con trỏ)

Vậy con trỏ là index vào bộ nhớ, ở một chỗ nào đó.

Lấy ví dụ ngẫu nhiên, giả sử số 3490 được lưu ở địa chỉ ("index")
23.237.489.202. Nếu ta có một con trỏ `int` tới số 3490 đó, giá trị
của con trỏ đó là 23.237.489.202... bởi vì con trỏ là địa chỉ bộ nhớ.
Cùng một thứ, chỉ khác cách gọi.

Giờ giả sử ta có thêm số nữa, 4096, được lưu ngay sau 3490 ở địa chỉ
23.237.489.210 (cao hơn 3490 là 8 vì mỗi `int` trong ví dụ này dài 8
byte).

Nếu cộng `1` vào con trỏ, thực ra nó nhảy tới trước `sizeof(int)` byte
để tới `int` kế. Nó biết nhảy chừng đó vì là con trỏ `int`. Nếu là
con trỏ `float`, nó sẽ nhảy tới trước `sizeof(float)` byte để tới
float kế!

Vậy bạn có thể nhìn `int` kế tiếp bằng cách cộng `1` vào con trỏ, cái
sau đó bằng cách cộng `2`, v.v.

### Thay đổi con trỏ

Ở mục trước ta thấy cách cộng một số nguyên vào con trỏ. Lần này, ta
sẽ _tự sửa chính con trỏ_.

Bạn có thể cộng (hoặc trừ) trực tiếp giá trị số nguyên vào (hoặc từ)
bất kỳ con trỏ nào!

Làm lại ví dụ đó, nhưng có vài thay đổi. Đầu tiên, tôi sẽ thêm `999`
vào cuối dãy số để làm sentinel (giá trị canh). Giá trị đó sẽ báo cho
ta biết đâu là cuối dữ liệu.

``` {.c}
int a[] = {11, 22, 33, 44, 55, 999};  // Add 999 here as a sentinel

int *p = &a[0];  // p points to the 11
```

Và ta cũng có `p` trỏ tới phần tử ở index `0` của `a`, tức `11`, giống
như trước.

Giờ, bắt đầu _tăng_ `p` để nó trỏ tới các phần tử tiếp theo của mảng.
Ta làm vậy cho đến khi `p` trỏ tới `999`, tức là cho đến khi
`*p == 999`:

``` {.c}
while (*p != 999) {       // While the thing p points to isn't 999
    printf("%d\n", *p);   // Print it
    p++;                  // Move p to point to the next int!
}
```

Điên ghê, nhỉ?

Chạy thử, đầu tiên `p` trỏ tới `11`. Rồi tăng `p`, nó trỏ tới `22`,
rồi lại tăng, trỏ tới `33`. Cứ thế cho đến khi trỏ tới `999` thì
thoát.

### Trừ con trỏ

[i[Pointers-->subtracting]<]
Bạn có thể trừ một giá trị từ con trỏ để lui về địa chỉ trước đó, y
như ta cộng vào vậy.

Nhưng ta cũng có thể trừ hai con trỏ để tìm khoảng cách giữa chúng,
chẳng hạn ta có thể tính giữa hai `int*` có bao nhiêu `int`. Điểm lưu
ý là chuyện này chỉ hoạt động trong cùng một mảng^[Hoặc chuỗi, thực
ra là mảng `char`. Hơi kỳ là bạn cũng có thể có con trỏ tham chiếu
tới _một chỗ sau_ phần cuối mảng mà vẫn làm toán với nó được. Chỉ là
không được dereference khi nó ở đó.], nếu các con trỏ trỏ tới thứ
khác, bạn nhận undefined behavior.

Nhớ chuỗi là `char*` trong C chứ? Xem thử có dùng cái này viết một
biến thể `strlen()` để tính độ dài chuỗi bằng phép trừ con trỏ được
không.

Ý tưởng là nếu có con trỏ tới đầu chuỗi, ta có thể tìm con trỏ tới
cuối chuỗi bằng cách quét tới khi gặp ký tự `NUL`.

Và nếu có con trỏ tới đầu chuỗi, và tính được con trỏ tới cuối chuỗi,
ta chỉ cần trừ hai con trỏ là ra độ dài!

``` {.c .numberLines}
#include <stdio.h>

int my_strlen(char *s)
{
    // Start scanning from the beginning of the string
    char *p = s;

    // Scan until we find the NUL character
    while (*p != '\0')
        p++;

    // Return the difference in pointers
    return p - s;
}

int main(void)
{
    printf("%d\n", my_strlen("Hello, world!"));  // Prints "13"
}
```

Nhớ rằng bạn chỉ được trừ con trỏ giữa hai con trỏ trỏ tới cùng một
mảng!
[i[Pointers-->subtracting]>]

## Array/Pointer Equivalence {#arraypointerequiv}

[i[Pointers-->array equivalence]<]
Cuối cùng thì cũng đến lúc nói chuyện này! Ta đã thấy kha khá ví dụ
chỗ nào đó trộn lẫn ký hiệu mảng, nhưng giờ xin đưa ra _công thức
căn bản của array/pointer equivalence_:

``` {.c}
a[b] == *(a + b)
```

Nghiền đi! Chúng tương đương và dùng thay cho nhau được!

Tôi đã đơn giản hoá một chút, vì trong ví dụ trên `a` và `b` đều có
thể là biểu thức, và có khi ta cần thêm ngoặc để ép thứ tự toán tử
nếu biểu thức phức tạp.

Spec thì luôn cụ thể, tuyên bố (trong C11 §6.5.2.1¶2):

> `E1[E2]` is identical to `(*((E1)+(E2)))`

nhưng cái đó hơi khó hình dung. Chỉ cần đảm bảo dùng ngoặc nếu biểu
thức phức tạp để phép toán diễn ra đúng thứ tự.

Nghĩa là ta có thể _quyết định_ dùng ký hiệu mảng hay ký hiệu con trỏ
cho bất kỳ mảng hay con trỏ nào (giả định nó trỏ tới một phần tử của
một mảng).

Dùng cả mảng và con trỏ với cả hai ký hiệu:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int a[] = {11, 22, 33, 44, 55};

    int *p = a;  // p points to the first element of a, 11

    // Print all elements of the array a variety of ways:

    for (int i = 0; i < 5; i++)
        printf("%d\n", a[i]);      // Array notation with a

    for (int i = 0; i < 5; i++)
        printf("%d\n", p[i]);      // Array notation with p

    for (int i = 0; i < 5; i++)
        printf("%d\n", *(a + i));  // Pointer notation with a

    for (int i = 0; i < 5; i++)
        printf("%d\n", *(p + i));  // Pointer notation with p

    for (int i = 0; i < 5; i++)
        printf("%d\n", *(p++));    // Moving pointer p
        //printf("%d\n", *(a++));    // Moving array variable a--ERROR!
}
```

Có thể thấy nhìn chung, nếu bạn có biến mảng, có thể dùng ký hiệu con
trỏ hay ký hiệu mảng để truy cập phần tử. Tương tự với biến con trỏ.

Khác biệt lớn duy nhất là bạn có thể _sửa_ một con trỏ để trỏ sang
địa chỉ khác, nhưng không làm thế được với biến mảng. <!-- 6.3.2.1p2
--> Nói cách khác, bạn không gán được vào biến mảng, chỉ gán vào từng
phần tử của mảng đó thôi.

Nếu thực sự muốn copy mảng này sang mảng khác, bạn phải dùng hàm như
`memcpy()` (hay một vòng lặp).

### Array/Pointer Equivalence trong lời gọi hàm

Đây chắc chắn là chỗ bạn gặp khái niệm này nhiều nhất.

Nếu bạn có hàm nhận đối số là con trỏ, ví dụ:

``` {.c}
int my_strlen(char *s)
```

nghĩa là bạn có thể truyền hoặc mảng, hoặc con trỏ vào hàm này và nó
vẫn chạy!

``` {.c}
char s[] = "Antelopes";
char *t = "Wombats";

printf("%d\n", my_strlen(s));  // Works!
printf("%d\n", my_strlen(t));  // Works, too!
```

Và đó cũng là lý do hai signature hàm này tương đương:

``` {.c}
int my_strlen(char *s)    // Works!
int my_strlen(char s[])   // Works, too!
```
[i[Pointers-->array equivalence]>]

## Con trỏ `void`

[i[`void*` void pointer]<]

Bạn đã thấy từ khoá `void` dùng với hàm để chỉ không có tham số hay
không có giá trị trả về, nhưng cái này là một con thú hoàn toàn tách
biệt, không liên quan.

Một `void*` chắc chắn là con trỏ tới một *thứ* đang tồn tại. Nhưng
phần `void` chỉ ra rằng ta không biết _kiểu_ của thứ đó. Và đôi khi,
tin hay không tuỳ bạn, cái đó thực sự hữu ích. Nó cho phép viết code
kiểu-bất-khả-tri hơn một chút, một sự linh hoạt rất nice trong một
ngôn ngữ có kiểu như C.

Về cơ bản có hai trường hợp sử dụng, xem và giải hoặc một chút bí ẩn.

[i[`memcpy()` function]<]

1. Hàm sẽ xử lý một thứ gì đó theo từng byte. Ví dụ, `memcpy()` chép
   byte bộ nhớ từ con trỏ này sang con trỏ kia, nhưng các con trỏ đó
   có thể trỏ tới kiểu bất kỳ. `memcpy()` tận dụng chuyện nếu bạn
   duyệt qua các `char*`, bạn đang duyệt qua các byte của một đối
   tượng bất kể đối tượng là kiểu gì. Sẽ nói thêm ở tiểu mục
   [Multibyte Values](#multibyte-values).

2. Một hàm khác gọi một hàm bạn truyền vào cho nó (callback), và nó
   truyền dữ liệu cho bạn. Bạn biết kiểu của dữ liệu, nhưng hàm gọi
   bạn thì không. Nên nó truyền `void*` cho bạn, vì nó không biết
   kiểu, rồi bạn chuyển cái đó về kiểu mình cần.
   [fl[`qsort()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-qsort]]
   và
   [fl[`bsearch()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-bsearch]]
   có sẵn đều dùng kỹ thuật này.

Xem ví dụ, hàm `memcpy()` có sẵn:

``` {.c}
void *memcpy(void *s1, void *s2, size_t n);
```

Hàm này chép `n` byte bộ nhớ bắt đầu từ địa chỉ `s2` vào bộ nhớ bắt
đầu từ địa chỉ `s1`.

Nhưng nhìn! `s1` và `s2` là `void*`! Vì sao? Nghĩa là gì? Thử thêm ví
dụ.

Chẳng hạn, ta có thể chép một chuỗi bằng `memcpy()` (dù `strcpy()`
phù hợp hơn cho chuỗi):

``` {.c .numberLines}
#include <stdio.h>
#include <string.h>

int main(void)
{
    char s[] = "Goats!";
    char t[100];

    memcpy(t, s, 7);  // Copy 7 bytes--including the NUL terminator!

    printf("%s\n", t);  // "Goats!"
}
```

Hoặc chép vài `int`:

``` {.c .numberLines}
#include <stdio.h>
#include <string.h>

int main(void)
{
    int a[] = {11, 22, 33};
    int b[3];

    memcpy(b, a, 3 * sizeof(int));  // Copy 3 ints of data

    printf("%d\n", b[1]);  // 22
}
```

Cái này hơi hoang đấy, thấy ta vừa làm gì với `memcpy()` chứ? Ta chép
dữ liệu từ `a` sang `b`, nhưng phải chỉ rõ chép bao nhiêu _byte_, và
một `int` thì lớn hơn một byte.

Vậy, một `int` chiếm bao nhiêu byte? Trả lời: tuỳ hệ thống. Nhưng ta
có thể biết bao nhiêu byte một kiểu chiếm bằng toán tử `sizeof`.

Vậy đây rồi: một `int` cần `sizeof(int)` byte bộ nhớ để lưu.

Và nếu có 3 cái trong mảng, như ví dụ đó, tổng dung lượng dùng cho 3
`int` phải là `3 * sizeof(int)`.

(Ở ví dụ chuỗi trước, chặt chẽ kỹ thuật hơn thì phải chép
`7 * sizeof(char)` byte. Nhưng `char` theo định nghĩa luôn dài một
byte, nên cái đó thoái hoá thành `7 * 1`.)

Ta thậm chí có thể chép một `float` hay `struct` bằng `memcpy()`!
(Dù đây là lạm dụng, ta nên dùng `=` cho chuyện đó):

``` {.c}
struct antelope my_antelope;
struct antelope my_clone_antelope;

// ...

memcpy(&my_clone_antelope, &my_antelope, sizeof my_antelope);
```

Nhìn `memcpy()` đa năng chưa! Nếu có con trỏ tới nguồn và con trỏ tới
đích, và biết số byte muốn chép, bạn có thể chép _bất kỳ kiểu dữ liệu
nào_.

Tưởng tượng nếu không có `void*`. Ta sẽ phải viết các hàm `memcpy()`
chuyên biệt cho mỗi kiểu:

[i[`memcpy()` function]>]

``` {.c}
memcpy_int(int *a, int *b, int count);
memcpy_float(float *a, float *b, int count);
memcpy_double(double *a, double *b, int count);
memcpy_char(char *a, char *b, int count);
memcpy_unsigned_char(unsigned char *a, unsigned char *b, int count);

// etc... blech!
```

Tốt hơn nhiều là cứ dùng `void*` và có một hàm lo hết.

Đó là sức mạnh của `void*`. Bạn có thể viết hàm không quan tâm kiểu
biến mà vẫn làm được việc với nó.

Nhưng sức mạnh lớn đi kèm trách nhiệm lớn. Có thể trách nhiệm không
_đến mức_ đó trong trường hợp này, nhưng có những giới hạn.

[i[`void*` void pointer-->caveats]<]

1. Không làm được số học con trỏ trên `void*`.
2. Không dereference được `void*`.
3. Không dùng được toán tử mũi tên trên `void*`, vì đó cũng là
   dereference.
4. Không dùng được ký hiệu mảng trên `void*`, vì đó cũng là
   dereference^[Vì nhớ rằng ký hiệu mảng chỉ là một dereference cộng
   chút toán tử con trỏ, mà bạn không dereference được `void*`!].

Và nếu nghĩ kỹ, các quy tắc này hợp lý. Tất cả thao tác đó dựa vào
việc biết `sizeof` của kiểu dữ liệu được trỏ tới, mà với `void*` ta
không biết kích cỡ của dữ liệu được trỏ tới, có thể là bất cứ gì!

[i[`void*` void pointer-->caveats]>]

Nhưng khoan, nếu không dereference được `void*` thì nó có ích gì cho
bạn?

Giống như với `memcpy()`, nó giúp bạn viết các hàm tổng quát xử lý
được nhiều kiểu dữ liệu. Nhưng bí mật là, ở cốt lõi, _bạn chuyển
`void*` sang kiểu khác trước khi dùng_!

Và chuyển thì dễ: chỉ cần gán vào biến có kiểu mong muốn^[Bạn cũng có
thể _cast_ `void*` sang kiểu khác, nhưng ta chưa tới cast.].

``` {.c}
char a = 'X';  // A single char

void *p = &a;  // p points to the 'X'
char *q = p;   // q also points to the 'X'

printf("%c\n", *p);  // ERROR--cannot dereference void*!
printf("%c\n", *q);  // Prints "X"
```

[i[`memcpy()` function]<]
Viết `memcpy()` của riêng mình để thử. Ta có thể chép byte (`char`),
và biết số byte vì nó được truyền vào.

``` {.c}
void *my_memcpy(void *dest, void *src, int byte_count)
{
    // Convert void*s to char*s
    char *s = src, *d = dest;

    // Now that we have char*s, we can dereference and copy them
    while (byte_count--) {
        *d++ = *s++;
    }

    // Most of these functions return the destination, just in case
    // that's useful to the caller.
    return dest;
}
```

Ngay đầu, ta chép `void*` vào `char*` để có thể dùng chúng như
`char*`. Đơn giản vậy thôi.

Rồi vui vẻ trong một vòng while, nơi ta giảm `byte_count` đến khi
thành false (`0`). Nhớ rằng với post-decrement, giá trị của biểu thức
được tính (cho `while` dùng) _rồi_ biến mới được giảm.

Và vui vẻ trong phần copy, nơi ta gán `*d = *s` để chép byte, nhưng
làm với post-increment để cả `d` và `s` chuyển sang byte kế sau khi
gán xong.

Cuối cùng, hầu hết các hàm về bộ nhớ và chuỗi trả về một bản sao của
con trỏ tới đích phòng khi caller cần dùng.

Xong rồi, tôi chỉ muốn nhanh chóng chỉ ra rằng ta có thể dùng kỹ
thuật này để duyệt qua các byte của _bất kỳ_ đối tượng nào trong C,
`float`, `struct`, hay gì cũng được!
[i[`memcpy()` function]>]

[i[`qsort()` function]<]
[Làm]{#qsort-example} thêm một ví dụ thực tế với routine có sẵn
`qsort()`, có thể sắp xếp _bất cứ gì_ nhờ phép màu của `void*`.

(Trong ví dụ dưới, có thể bỏ qua từ `const`, ta chưa nói tới.)

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

// The type of structure we're going to sort
struct animal {
    char *name;
    int leg_count;
};

// This is a comparison function called by qsort() to help it determine
// what exactly to sort by. We'll use it to sort an array of struct
// animals by leg_count.
int compar(const void *elem1, const void *elem2)
{
    // We know we're sorting struct animals, so let's make both
    // arguments pointers to struct animals
    const struct animal *animal1 = elem1;
    const struct animal *animal2 = elem2;

    // Return <0 =0 or >0 depending on whatever we want to sort by.

    // Let's sort ascending by leg_count, so we'll return the difference
    // in the leg_counts
    if (animal1->leg_count > animal2->leg_count)
        return 1;
    
    if (animal1->leg_count < animal2->leg_count)
        return -1;

    return 0;
}

int main(void)
{
    // Let's build an array of 4 struct animals with different
    // characteristics. This array is out of order by leg_count, but
    // we'll sort it in a second.
    struct animal a[4] = {
        {.name="Dog", .leg_count=4},
        {.name="Monkey", .leg_count=2},
        {.name="Antelope", .leg_count=4},
        {.name="Snake", .leg_count=0}
    };

    // Call qsort() to sort the array. qsort() needs to be told exactly
    // what to sort this data by, and we'll do that inside the compar()
    // function.
    //
    // This call is saying: qsort array a, which has 4 elements, and
    // each element is sizeof(struct animal) bytes big, and this is the
    // function that will compare any two elements.
    qsort(a, 4, sizeof(struct animal), compar);

    // Print them all out
    for (int i = 0; i < 4; i++) {
        printf("%d: %s\n", a[i].leg_count, a[i].name);
    }
}
```

Chỉ cần bạn đưa cho `qsort()` một hàm có thể so sánh hai item trong
mảng cần sort, nó sắp được mọi thứ. Và làm vậy mà không cần phải
hard-code kiểu của item ở đâu cả. `qsort()` chỉ sắp xếp lại các khối
byte dựa vào kết quả của hàm `compar()` bạn truyền vào.
[i[`qsort()` function]>]
[i[`void*` void pointer]>]
[i[Pointers-->arithmetic]>]

<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Types IV: Qualifiers và Specifiers

Giờ ta đã có thêm vài kiểu dưới tay rồi, hóa ra ta có thể gắn cho
chúng thêm vài thuộc tính để điều khiển cách chúng cư xử. Đó chính
là _type qualifier_ (bổ từ kiểu) và _storage-class specifier_
(specifier lớp lưu trữ).

## Type Qualifier

[i[Type qualifiers]<]

Mấy thứ này sẽ cho phép bạn khai báo giá trị hằng, và cũng cho
compiler thêm gợi ý tối ưu hóa mà nó có thể dùng.

### `const`

[i[`const` type qualifier]<]

Đây là type qualifier phổ biến nhất bạn sẽ gặp. Nó nghĩa là biến đó
là hằng, và bất kỳ nỗ lực nào hòng sửa nó sẽ khiến compiler nổi đóa.

``` {.c}
const int x = 2;

x = 4;  // COMPILER PUKING SOUNDS, can't assign to a constant
```

Bạn không thể đổi giá trị `const`.

Bạn cũng hay thấy `const` trong danh sách tham số của hàm:

``` {.c}
void foo(const int x)
{
    printf("%d\n", x + 30);  // OK, doesn't modify "x"
}
```

#### `const` và con trỏ

[i[`const` type qualifier-->and pointers]<]

Chỗ này hơi lạ đời, vì có hai cách dùng mang hai ý nghĩa khác nhau
khi dính tới con trỏ.

Một là, ta có thể làm sao cho bạn không đổi được thứ mà con trỏ trỏ
đến. Bạn làm thế bằng cách đặt `const` ra phía trước cùng với tên
kiểu (trước dấu sao) trong khai báo kiểu.

``` {.c}
int x[] = {10, 20};
const int *p = x; 

p++;  // We can modify p, no problem

*p = 30; // Compiler error! Can't change what it points to
```

Hơi khó hiểu tí, nhưng hai thứ sau là tương đương:

``` {.c}
const int *p;  // Can't modify what p points to
int const *p;  // Can't modify what p points to, just like the previous line
```

Hay rồi, vậy là ta không đổi được thứ con trỏ trỏ đến, nhưng vẫn đổi
được bản thân con trỏ. Thế nếu muốn ngược lại thì sao? Ta muốn đổi
được thứ con trỏ trỏ đến nhưng _không_ đổi được bản thân con trỏ?

Chỉ cần chuyển `const` ra sau dấu sao trong khai báo:

``` {.c}
int *const p;   // We can't modify "p" with pointer arithmetic

p++;  // Compiler error!
```

Nhưng ta vẫn đổi được thứ nó trỏ đến:

``` {.c}
int x = 10;
int *const p = &x;

*p = 20;   // Set "x" to 20, no problem
```

Bạn cũng có thể làm cả hai thứ đều `const`:

``` {.c}
const int *const p;  // Can't modify p or *p!
```

Cuối cùng, nếu bạn có nhiều mức gián tiếp, bạn nên đặt `const` ở
đúng mức. Một con trỏ `const` không có nghĩa là con trỏ mà nó trỏ
tới cũng phải thế. Bạn có thể đặt tường minh như mấy ví dụ sau:

``` {.c}
char **p;
p++;     // OK!
(*p)++;  // OK!

char **const p;
p++;     // Error!
(*p)++;  // OK!

char *const *p;
p++;     // OK!
(*p)++;  // Error!

char *const *const p;
p++;     // Error!
(*p)++;  // Error!
```

[i[`const` type qualifier-->and pointers]>]

#### `const` Correctness

[i[`const` type qualifier-->correctness]<]

Còn một chuyện nữa tôi phải nhắc, là compiler sẽ cảnh báo với thứ
kiểu thế này:

``` {.c}
const int x = 20;
int *p = &x;
```

nói đại loại như:

``` {.default}
initialization discards 'const' qualifier from pointer type target
```

Có chuyện gì ở đây?

Ta cần nhìn kiểu ở hai bên dấu gán:

``` {.c}
    const int x = 20;
    int *p = &x;
//    ^       ^
//    |       |
//  int*    const int*
```

Compiler đang cảnh báo rằng giá trị bên phải dấu gán là `const`,
còn bên trái thì không. Và compiler đang báo cho ta biết rằng nó
đang vứt đi tính "const" của biểu thức bên phải.

Tức là ta _vẫn_ có thể thử làm như dưới đây, nhưng nó sai. Compiler
sẽ cảnh báo, và đó là hành vi không xác định:

``` {.c}
const int x = 20;
int *p = &x;

*p = 40;  // Undefined behavior--maybe it modifies "x", maybe not!

printf("%d\n", x);  // 40, if you're lucky
```

[i[`const` type qualifier-->correctness]>]
[i[`const` type qualifier]>]

### `restrict`

[i[`restrict` type qualifier]<]

TLDR: bạn không bao giờ phải dùng cái này và có thể lờ nó đi mỗi khi
thấy. Nếu bạn dùng đúng, rất có thể bạn sẽ được chút hiệu năng. Nếu
dùng sai, bạn sẽ được undefined behavior.

`restrict` là gợi ý cho compiler rằng một vùng bộ nhớ cụ thể sẽ chỉ
được truy cập qua đúng một con trỏ chứ không phải qua cái khác. (Tức
là sẽ không có aliasing với đối tượng mà con trỏ `restrict` trỏ
tới.) Nếu dev khai báo một con trỏ là `restrict` rồi truy cập đối
tượng đó theo cách khác (ví dụ qua con trỏ khác), hành vi là không
xác định.

Về cơ bản bạn đang nói với C: "Này, tôi đảm bảo rằng cái con trỏ
duy nhất này là đường duy nhất tôi truy cập bộ nhớ đó, và nếu tôi
nói dối thì anh cứ quẳng undefined behavior vào mặt tôi."

Và C dùng thông tin đó để thực hiện một số tối ưu. Ví dụ, nếu bạn
dereference con trỏ `restrict` lặp đi lặp lại trong vòng lặp, C có
thể quyết định cache kết quả trong một thanh ghi và chỉ lưu kết quả
cuối cùng khi vòng lặp xong. Nếu có con trỏ khác cùng trỏ đến vùng
nhớ đó và truy cập trong vòng lặp, kết quả sẽ không chính xác.

(Lưu ý là `restrict` không có tác dụng nếu đối tượng được trỏ tới
không bao giờ được ghi. Tất cả là về tối ưu quanh chuyện ghi vào bộ
nhớ.)

Ta viết thử một hàm hoán đổi hai biến, và sẽ dùng từ khóa `restrict`
để cam kết với C rằng ta sẽ không bao giờ truyền vào hai con trỏ
cùng trỏ tới một chỗ. Rồi ta làm hỏng cam kết đó và thử truyền hai
con trỏ cùng trỏ tới một chỗ.

``` {.c .numberLines}
void swap(int *restrict a, int *restrict b)
{
    int t;

    t = *a;
    *a = *b;
    *b = t;
}

int main(void)
{
    int x = 10, y = 20;

    swap(&x, &y);  // OK! "a" and "b", above, point to different things

    swap(&x, &x);  // Undefined behavior! "a" and "b" point to the same thing
}
```

Nếu ta bỏ các từ khóa `restrict` ra, cả hai lời gọi trên đều an
toàn. Nhưng khi đó compiler có thể không tối ưu được.

`restrict` có block scope, tức là hạn chế chỉ kéo dài trong scope nó
được dùng. Nếu nó ở danh sách tham số của một hàm, nó ở trong block
scope của hàm đó.

Nếu con trỏ bị restrict trỏ vào một mảng, nó chỉ áp dụng cho từng
đối tượng trong mảng. Các con trỏ khác vẫn có thể đọc và ghi mảng
đó miễn là không đọc hay ghi cùng phần tử mà con trỏ restrict đã
chạm vào.

Nếu nó ở ngoài mọi hàm, tức ở file scope, hạn chế áp dụng cho toàn
bộ chương trình.

Bạn rất có thể sẽ thấy cái này trong các hàm thư viện như
`printf()`:

``` {.c}
int printf(const char * restrict format, ...);
```

Vẫn thế, nó chỉ báo với compiler rằng bên trong hàm `printf()`, chỉ
có đúng một con trỏ nào đó trỏ đến phần bất kỳ của chuỗi `format`.

Một chú ý cuối: nếu vì lý do nào đó bạn dùng ký pháp mảng cho tham
số hàm thay vì ký pháp con trỏ, bạn có thể dùng `restrict` như sau:

``` {.c}
void foo(int p[restrict])     // With no size

void foo(int p[restrict 10])  // Or with a size
```

Nhưng ký pháp con trỏ thì phổ biến hơn.

[i[`restrict` type qualifier]>]

### `volatile`

[i[`volatile` type qualifier]<]

Bạn ít khi gặp hay cần cái này trừ khi đang trực tiếp làm việc với
phần cứng.

`volatile` báo với compiler rằng một giá trị có thể thay đổi sau
lưng nó và phải được tra cứu lại mỗi lần.

Ví dụ: compiler đang nhìn vào bộ nhớ ở một địa chỉ mà liên tục được
cập nhật trong hậu trường, ví dụ như một bộ đếm thời gian phần cứng.

Nếu compiler quyết định tối ưu chuyện đó bằng cách lưu giá trị trong
một thanh ghi suốt một quãng dài, giá trị trong bộ nhớ sẽ đổi còn
thanh ghi thì không phản ánh.

Khi khai báo một thứ là `volatile`, bạn đang nói với compiler: "Này,
thứ con trỏ này trỏ đến có thể đổi bất cứ lúc nào vì lý do ngoài mã
của chương trình này."

``` {.c}
volatile int *p;
```

### `_Atomic`

Đây là một tính năng C tùy chọn mà ta sẽ bàn ở [chương
Atomics](#chapter-atomics).

[i[`volatile` type qualifier]>]
[i[Type qualifiers]>]

## Storage-Class Specifiers

[i[Storage-Class Specifiers]<]

Storage-class specifier tương tự như type qualifier. Chúng cho
compiler thêm thông tin về kiểu của một biến.

### `auto`

[i[`auto` storage class]<]

Bạn gần như không bao giờ thấy từ khóa này, vì `auto` là mặc định
cho biến ở block scope. Nó được ngầm định.

Hai đoạn sau là như nhau:

``` {.c}
{
    int a;         // auto is the default...
    auto int a;    // So this is redundant
}
```

Từ khóa `auto` cho biết đối tượng này có _automatic storage
duration_ (thời gian tồn tại tự động). Tức là nó tồn tại trong scope
mà nó được định nghĩa, và tự động được giải phóng khi thoát scope.

Một cái bẫy với biến automatic là giá trị của chúng là không xác
định cho đến khi bạn khởi tạo tường minh. Ta nói chúng đầy dữ liệu
"random" hoặc "rác", dù tôi không ưa cả hai cách gọi đó lắm. Dù sao,
bạn sẽ không biết trong đó có gì trừ khi bạn khởi tạo nó.

[i[`auto` storage class]>]

Luôn khởi tạo mọi biến automatic trước khi dùng!

### `static` {#static}

[i[`static` storage class]<]

Từ khóa này có hai nghĩa, tùy thuộc biến ở file scope hay block
scope.

Bắt đầu với block scope.

#### `static` ở block scope

[i[`static` storage class-->in block scope]<]

Ở đây, về cơ bản ta đang nói: "Tôi chỉ muốn một phiên bản duy nhất
của biến này tồn tại, dùng chung giữa các lần gọi."

Tức là giá trị của nó sẽ giữ nguyên giữa các lời gọi.

`static` ở block scope kèm initializer sẽ chỉ được khởi tạo đúng một
lần lúc khởi động chương trình, chứ không phải mỗi lần hàm được gọi.

Làm ví dụ cái xem:

``` {.c .numberLines}
#include <stdio.h>

void counter(void)
{
    static int count = 1;  // This is initialized one time

    printf("This has been called %d time(s)\n", count);

    count++;
}

int main(void)
{
    counter();  // "This has been called 1 time(s)"
    counter();  // "This has been called 2 time(s)"
    counter();  // "This has been called 3 time(s)"
    counter();  // "This has been called 4 time(s)"
}
```

Thấy cách giá trị của `count` còn giữ giữa các lời gọi không?

Một chuyện đáng lưu ý là biến `static` ở block scope mặc định được
khởi tạo bằng `0`.

``` {.c}
static int foo;      // Default starting value is `0`...
static int foo = 0;  // So the `0` assignment is redundant
```

Cuối cùng, hãy nhớ rằng nếu bạn viết chương trình đa luồng, bạn phải
chắc chắn không để nhiều luồng cùng dẫm lên một biến.

[i[`static` storage class-->in block scope]>]

#### `static` ở file scope

[i[`static` storage class-->in file scope]<]

Khi ra tới file scope, ngoài mọi block, nghĩa thay đổi kha khá.

Biến ở file scope vốn đã tồn tại giữa các lời gọi hàm, nên chuyện đó
đã sẵn ở đó rồi.

Thay vào đó, `static` trong ngữ cảnh này nghĩa là biến này không
nhìn thấy được bên ngoài file mã nguồn này. Hơi giống "global",
nhưng chỉ trong file này.

Sẽ nói thêm ở phần build từ nhiều file mã nguồn.

[i[`static` storage class-->in file scope]>]
[i[`static` storage class]>]

### `extern` {#extern}

[i[`extern` storage class]<]

Storage-class specifier `extern` cho ta cách tham chiếu đến các đối
tượng trong file mã nguồn khác.

Ví dụ, giả sử file `bar.c` chỉ có đúng đoạn sau:

``` {.c .numberLines}
// bar.c

int a = 37;
```

Chỉ vậy thôi. Khai báo một `int a` mới ở file scope.

Nhưng nếu ta có file mã nguồn khác là `foo.c`, và muốn tham chiếu
đến `a` ở trong `bar.c` thì sao?

Dễ với từ khóa `extern`:

``` {.c .numberLines}
// foo.c

extern int a;

int main(void)
{
    printf("%d\n", a);  // 37, from bar.c!

    a = 99;

    printf("%d\n", a);  // Same "a" from bar.c, but it's now 99
}
```

Ta cũng có thể đặt `extern int a` trong block scope, nó vẫn tham
chiếu tới `a` trong `bar.c`:

``` {.c .numberLines}
// foo.c

int main(void)
{
    extern int a;

    printf("%d\n", a);  // 37, from bar.c!

    a = 99;

    printf("%d\n", a);  // Same "a" from bar.c, but it's now 99
}
```

Bây giờ, nếu `a` trong `bar.c` đã được đánh dấu `static`, chuyện này
sẽ không hoạt động. Biến `static` ở file scope không nhìn thấy được
bên ngoài file đó.

Một ghi chú cuối về `extern` với hàm. Với hàm, `extern` là mặc định,
nên nó thừa. Bạn có thể khai báo hàm là `static` nếu chỉ muốn nó
nhìn thấy được trong một file mã nguồn duy nhất.

[i[`extern` storage class]>]

### `register`

[i[`register` storage class]<]

Đây là từ khóa để gợi ý cho compiler rằng biến này được dùng thường
xuyên, và nên được làm cho truy cập nhanh nhất có thể. Compiler
không có nghĩa vụ phải đồng ý.

Giờ thì, bộ tối ưu của compiler C hiện đại khá giỏi trong việc tự
tìm ra chuyện này, nên hiếm khi thấy từ khóa này ngày nay.

Nhưng nếu bạn phải dùng:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    register int a;   // Make "a" as fast to use as possible.

    for (a = 0; a < 10; a++)
        printf("%d\n", a);
}
```

Có cái giá đi kèm. Bạn không thể lấy địa chỉ của một register:

``` {.c}
register int a;
int *p = &a;    // COMPILER ERROR! Can't take address of a register
```

Điều tương tự áp dụng cho bất kỳ phần nào của một mảng:

``` {.c}
register int a[] = {11, 22, 33, 44, 55};
int *p = a;  // COMPILER ERROR! Can't take address of a[0]
```

Hoặc dereference phần nào đó của mảng:

``` {.c}
register int a[] = {11, 22, 33, 44, 55};

int a = *(a + 2);  // COMPILER ERROR! Address of a[0] taken
```

Thú vị là, với phiên bản tương đương dùng ký pháp mảng, gcc chỉ cảnh
báo:

``` {.c}
register int a[] = {11, 22, 33, 44, 55};

int a = a[2];  // COMPILER WARNING!
```

với:

``` {.default}
warning: ISO C forbids subscripting ‘register’ array
```

Việc không lấy được địa chỉ biến register giải phóng compiler để nó
có thể tối ưu quanh giả định đó, nếu nó chưa tự tìm ra rồi. Thêm
`register` vào biến `const` còn ngăn ta vô tình truyền con trỏ của
nó cho hàm khác sẵn sàng ngó lơ tính
const^[https://gustedt.wordpress.com/2010/08/17/a-common-misconsception-the-register-keyword/].

Chút gốc gác lịch sử cho vui: sâu trong CPU có mấy "biến" nhỏ chuyên
dụng gọi là [flw[_thanh ghi_|Processor_register]]. Chúng truy cập
siêu nhanh so với RAM, nên dùng chúng thì lên được ít tốc độ. Nhưng
chúng không ở trong RAM, nên không có địa chỉ bộ nhớ gắn kèm (đó là
lý do bạn không thể lấy địa chỉ của hay lấy con trỏ tới chúng).

Nhưng, như tôi nói, compiler hiện đại rất giỏi sinh mã tối ưu, dùng
thanh ghi bất cứ khi nào có thể bất kể bạn có ghi từ khóa `register`
hay không. Không những thế, spec còn cho phép chúng coi như bạn gõ
`auto` nếu nó muốn. Nên không có đảm bảo gì.

[i[`register` storage class]>]

### `_Thread_local`

[i[`_Thread_local` storage class]<]

Khi bạn dùng nhiều luồng và bạn có vài biến ở global scope hoặc ở
`static` block scope, đây là cách để mỗi luồng có bản sao riêng của
biến. Nó sẽ giúp bạn tránh race condition và việc các luồng dẫm lên
chân nhau.

Nếu bạn đang ở block scope, bạn phải dùng cái này cùng với `extern`
hoặc `static`.

Ngoài ra, nếu bạn include `<threads.h>`, bạn có thể dùng
`thread_local` dễ nuốt hơn, làm alias cho cái `_Thread_local` xấu
xí.

Thông tin thêm có ở [phần Threads](#thread-local).

[i[`_Thread_local` storage class]>]
[i[Storage-Class Specifiers]>]

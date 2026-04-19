<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Types Phần V: Compound Literals và Generic Selections

Đây là chương cuối về types! Ta sẽ nói hai chuyện:

* Làm sao có object "ẩn danh" không tên và lợi ích của nó.
* Làm sao tạo code phụ thuộc kiểu.

Chúng không liên quan lắm, nhưng cũng không đáng mỗi cái một chương.
Nên tôi nhét chúng vào đây như một kẻ nổi loạn!

## Compound Literals

[i[Compound literals]<]

Đây là một tính năng hay của ngôn ngữ cho phép bạn tạo một object
thuộc kiểu nào đó trên đường đi mà không cần gán nó vào biến. Bạn có
thể làm kiểu đơn giản, mảng, `struct`, gì cũng được.

Một trong những cách dùng chính của nó là truyền đối số phức tạp cho
hàm khi bạn không muốn tạo biến tạm để giữ giá trị.

Cách bạn tạo compound literal là đặt tên kiểu trong ngoặc đơn, rồi
đặt một initializer list phía sau. Ví dụ, một mảng `int` không tên
có thể trông như vầy:

``` {.c}
(int []){1,2,3,4}
```

Giờ, dòng code đó tự nó không làm gì cả. Nó tạo một mảng không tên
gồm 4 `int`, rồi vứt đi mà không dùng.

Ta có thể dùng một con trỏ để lưu tham chiếu tới mảng...

``` {.c}
int *p = (int []){1 ,2 ,3 ,4};

printf("%d\n", p[1]);  // 2
```

Nhưng cái đó có vẻ như kiểu vòng vo để có mảng. Ý là, ta cũng có thể
đã làm vầy^[Cũng không hoàn toàn giống, vì nó là mảng chứ không phải
con trỏ tới `int`.]:

``` {.c}
int p[] = {1, 2, 3, 4};

printf("%d\n", p[1]);  // 2
```

Vậy hãy xem ví dụ hữu ích hơn.

### Truyền object không tên cho hàm

[i[Compound literals-->passing to functions]<]

Giả sử ta có một hàm tính tổng một mảng `int`:

``` {.c}
int sum(int p[], int count)
{
    int total = 0;

    for (int i = 0; i < count; i++)
        total += p[i];

    return total;
}
```

Nếu ta muốn gọi nó, thường ta phải làm kiểu này, khai báo mảng và
lưu giá trị vào nó để truyền cho hàm:

``` {.c}
int a[] = {1, 2, 3, 4};

int s = sum(a, 4);
```

Nhưng object không tên cho ta cách bỏ qua biến bằng cách truyền
thẳng nó vào (tên tham số liệt kê phía trên). Xem này, ta sẽ thay
biến `a` bằng một mảng không tên truyền làm đối số đầu:

``` {.c}
//                   p[]         count
//           |-----------------|  |
int s = sum((int []){1, 2, 3, 4}, 4);
```

Khá gọn!

[i[Compound literals-->passing to functions]>]

### `struct` không tên

[i[Compound literals-->with `struct`]<]
[i[`struct` keyword-->compound literals]<]

Ta có thể làm điều tương tự với `struct`.

Trước, hãy làm không dùng object không tên. Ta sẽ định nghĩa một
`struct` để giữ toạ độ `x`/`y`. Rồi ta định nghĩa một cái, truyền
giá trị vào initializer của nó. Cuối cùng, truyền nó cho một hàm để
in giá trị ra:

``` {.c .numberLines}
#include <stdio.h>

struct coord {
    int x, y;
};

void print_coord(struct coord c)
{
    printf("%d, %d\n", c.x, c.y);
}

int main(void)
{
    struct coord t = {.x=10, .y=20};

    print_coord(t);   // prints "10, 20"
}
```

Đủ thẳng thắn?

Chỉnh nó để dùng object không tên thay cho biến `t` mà ta đang
truyền cho `print_coord()`.

Ta chỉ cần rút `t` ra và thay bằng một `struct` không tên:

``` {.c .numberLines startFrom="7"}
    //struct coord t = {.x=10, .y=20};

    print_coord((struct coord){.x=10, .y=20});   // prints "10, 20"
```

Vẫn chạy!

[i[`struct` keyword-->compound literals]>]
[i[Compound literals-->with `struct`]>]

### Con trỏ tới object không tên

[i[Compound literals-->pointers to]<]

Bạn có thể để ý trong ví dụ cuối rằng dù ta đang dùng `struct`, ta
truyền một bản sao của `struct` cho `print_coord()` chứ không phải
truyền con trỏ tới `struct`.

Hoá ra, ta có thể lấy địa chỉ của một object không tên bằng `&` như
thường.

Đó là vì, nhìn chung, nếu một toán tử chạy được với biến thuộc kiểu
đó, bạn có thể dùng toán tử đó trên object không tên thuộc cùng
kiểu.

Chỉnh code trên để ta truyền con trỏ tới object không tên

``` {.c .numberLines}
#include <stdio.h>

struct coord {
    int x, y;
};

void print_coord(struct coord *c)
{
    printf("%d, %d\n", c->x, c->y);
}

int main(void)
{
    //     Note the &
    //          |
    print_coord(&(struct coord){.x=10, .y=20});   // prints "10, 20"
}
```

Thêm nữa, đây có thể là cách hay ngay cả để truyền con trỏ tới
object đơn giản:

``` {.c}
// Pass a pointer to an int with value 3490
foo(&(int){3490});
```

Dễ vậy thôi.

[i[Compound literals-->pointers to]>]

### Object không tên và scope

[i[Compound literals-->scope]<]

Vòng đời của object không tên kết thúc ở cuối scope của nó. Cách
lớn nhất mà chuyện này có thể cắn bạn là nếu bạn tạo một object
không tên mới, lấy con trỏ tới nó, rồi rời khỏi scope của object.
Trong trường hợp đó, con trỏ sẽ tham chiếu tới một object đã chết.

Nên cái này là hành vi không xác định:

``` {.c}
int *p;

{
    p = &(int){10};
}

printf("%d\n", *p);  // INVALID: The (int){10} fell out of scope
```

Tương tự, bạn không thể trả về một con trỏ tới object không tên từ
một hàm. Object được giải phóng khi nó rơi khỏi scope:

``` {.c .numberLines}
#include <stdio.h>

int *get3490(void)
{
    // Don't do this
    return &(int){3490};
}

int main(void)
{
    printf("%d\n", *get3490());  // INVALID: (int){3490} fell out of scope
}
```

Cứ nghĩ scope của chúng giống như biến cục bộ thông thường. Bạn cũng
không thể trả về con trỏ tới biến cục bộ.

[i[Compound literals-->scope]>]

### Ví dụ object không tên hơi ngớ

Bạn có thể đặt kiểu nào vào đó và tạo object không tên cũng được.

Ví dụ, những cái này thực tế tương đương:

``` {.c}
int x = 3490;

printf("%d\n", x);               // 3490 (variable)
printf("%d\n", 3490);            // 3490 (constant)
printf("%d\n", (int){3490});     // 3490 (unnamed object)
```

Cái cuối là không tên, nhưng ngớ ngẩn. Thà làm cái đơn giản ở dòng
trước.

Nhưng hy vọng nó cho thêm chút rõ ràng về cú pháp.

[i[Compound literals]>]

## Generic Selections {#type-generics}

[i[Generic selections]<]

Đây là một biểu thức cho phép bạn chọn các đoạn code khác nhau tuỳ
vào _type_ của đối số đầu của biểu thức.

Ta sẽ xem ví dụ trong tích tắc, nhưng quan trọng là biết rằng cái
này được xử lý tại compile time, _không phải runtime_. Không có
phân tích runtime nào xảy ra ở đây.

[i[`_Generic` keyword]<]

Biểu thức bắt đầu bằng `_Generic`, chạy kiểu như `switch`, và nhận
ít nhất hai đối số.

Đối số đầu là một biểu thức (hay biến^[Biến dùng ở đây _là_ một biểu
thức.]) có một _type_. Mọi biểu thức đều có type. Các đối số còn lại
cho `_Generic` là các case về việc thay gì vào cho kết quả của biểu
thức nếu đối số đầu có type đó.

Cái gì cơ?

Thử coi sao.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i;
    float f;
    char c;

    char *s = _Generic(i,
                    int: "that variable is an int",
                    float: "that variable is a float",
                    default: "that variable is some type"
                );

    printf("%s\n", s);
}
```

Xem biểu thức `_Generic` bắt đầu ở dòng 9.

Khi compiler thấy nó, nó nhìn vào type của đối số đầu. (Trong ví dụ
này, type của biến `i`.) Rồi nó nhìn qua các case để tìm cái nào
thuộc type đó. Và rồi thay đối số vào chỗ toàn bộ biểu thức
`_Generic`.

Trong trường hợp này, `i` là `int`, nên nó khớp case đó. Rồi chuỗi
được thay vào chỗ biểu thức. Nên dòng trở thành vầy khi compiler
thấy:

``` {.c}
    char *s = "that variable is an int";
```

Nếu compiler không tìm thấy type khớp trong `_Generic`, nó tìm case
`default` tuỳ chọn và dùng nó.

Nếu không tìm được type khớp và không có `default`, bạn sẽ bị lỗi
compile. Biểu thức đầu **phải** khớp một trong các type hoặc
`default`.

Vì viết `_Generic` đi viết lại bất tiện, nó thường được dùng để làm
thân của một macro có thể tái dùng dễ dàng.

Hãy làm một macro `TYPESTR(x)` nhận một đối số và trả về chuỗi với
type của đối số.

Nên `TYPESTR(1)` sẽ trả về chuỗi `"int"`, chẳng hạn.

Nào:

``` {.c}
#include <stdio.h>

#define TYPESTR(x) _Generic((x), \
                        int: "int", \
                        long: "long", \
                        float: "float", \
                        double: "double", \
                        default: "something else")

int main(void)
{
    int i;
    long l;
    float f;
    double d;
    char c;

    printf("i is type %s\n", TYPESTR(i));
    printf("l is type %s\n", TYPESTR(l));
    printf("f is type %s\n", TYPESTR(f));
    printf("d is type %s\n", TYPESTR(d));
    printf("c is type %s\n", TYPESTR(c));
}
```

Cái này xuất ra:

``` {.default}
i is type int
l is type long
f is type float
d is type double
c is type something else
```

Không có gì bất ngờ, vì như ta đã nói, code trong `main()` được thay
bằng cái sau khi compile:

``` {.c}
    printf("i is type %s\n", "int");
    printf("l is type %s\n", "long");
    printf("f is type %s\n", "float");
    printf("d is type %s\n", "double");
    printf("c is type %s\n", "something else");
```

Và đó đúng là output ta thấy.

Làm thêm cái nữa. Tôi đã kèm vài macro ở đây để khi bạn chạy:

``` {.c}
int i = 10;
char *s = "Foo!";

PRINT_VAL(i);
PRINT_VAL(s);
```

bạn được output:

``` {.default}
i = 10
s = Foo!
```

Ta sẽ phải dùng chút phép thuật macro để làm được chuyện đó.

``` {.c .numberLines}
#include <stdio.h>
#include <string.h>

// Macro that gives back a format specifier for a type
#define FMTSPEC(x) _Generic((x), \
                        int: "%d", \
                        long: "%ld", \
                        float: "%f", \
                        double: "%f", \
                        char *: "%s")
                        // TODO: add more types
                        
// Macro that prints a variable in the form "name = value"
#define PRINT_VAL(x) do { \
    char fmt[512]; \
    snprintf(fmt, sizeof fmt, #x " = %s\n", FMTSPEC(x)); \
    printf(fmt, (x)); \
} while(0)

int main(void)
{
    int i = 10;
    float f = 3.14159;
    char *s = "Hello, world!";

    PRINT_VAL(i);
    PRINT_VAL(f);
    PRINT_VAL(s);
}
```

[i[`_Generic` keyword]>]

cho output:

``` {.default}
i = 10
f = 3.141590
s = Hello, world!
```

Ta có thể nhét hết vào một macro to, nhưng tôi chẻ ra hai để tránh
chảy máu mắt.

[i[Generic selections]>]

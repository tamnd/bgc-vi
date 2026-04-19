<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Cấp phát bộ nhớ thủ công

[i[Manual memory management]<]
Đây là một trong những mảng lớn C có khả năng khác với các ngôn ngữ
bạn đã biết: _cấp phát bộ nhớ thủ công_.

Các ngôn ngữ khác dùng reference counting, garbage collection, hay các
cách khác để quyết định khi nào cấp phát bộ nhớ mới cho dữ liệu, và khi
nào giải phóng khi không còn biến nào tham chiếu tới nó nữa.

Và chuyện đó nice. Nice khi không cần lo nghĩ, cứ thả hết tham chiếu
tới một item và tin rằng ở lúc nào đó bộ nhớ gắn với nó sẽ được giải
phóng.

Nhưng C không hoàn toàn như vậy.

[i[Automatic variables]<]
Dĩ nhiên trong C, một số biến được cấp phát và giải phóng tự động khi
chúng vào scope và ra scope. Ta gọi chúng là biến automatic. Đó là các
biến "local" block-scope bình thường. Không vấn đề gì.
[i[Automatic variables]>]

Nhưng nếu bạn muốn cái gì đó tồn tại lâu hơn một block cụ thể thì sao?
Đây là lúc quản lý bộ nhớ thủ công vào cuộc.

Bạn có thể bảo C cấp phát tường minh cho bạn một số byte nhất định để
dùng theo ý muốn. Các byte này sẽ vẫn được cấp phát cho đến khi bạn
tường minh giải phóng bộ nhớ đó^[Hoặc cho đến khi chương trình thoát,
lúc đó mọi bộ nhớ được cấp phát sẽ được giải phóng. Dấu sao: một số
hệ thống cho phép cấp phát bộ nhớ tồn tại cả sau khi chương trình
thoát, nhưng chuyện đó phụ thuộc hệ thống, ngoài phạm vi của sách
này, và chắc chắn bạn sẽ không vô tình làm thế.].

Quan trọng là giải phóng bộ nhớ khi xong việc với nó! Nếu không, ta
gọi đó là _memory leak_ (rò rỉ bộ nhớ) và process của bạn sẽ tiếp tục
chiếm bộ nhớ đó cho đến khi thoát.

_Bạn đã cấp phát thủ công, thì bạn phải giải phóng thủ công khi xong
việc._

Vậy làm sao? Ta sẽ học thêm vài hàm, và dùng toán tử `sizeof` để giúp
biết cần cấp phát bao nhiêu byte.

[i[The stack]<]
[i[The heap]<]
Nói kiểu phổ thông trong C, dev hay nói biến automatic local được cấp
phát "trên stack", còn bộ nhớ cấp phát thủ công thì "trên heap". Spec
không nói tới hai thứ đó, nhưng mọi dev C đều hiểu nếu bạn nhắc tới.
[i[The heap]>]
[i[The stack]>]

Mọi hàm ta sẽ học trong chương này đều nằm trong `<stdlib.h>`.

## Cấp phát và giải phóng, `malloc()` và `free()`

[i[`malloc()` function]<]
Hàm `malloc()` nhận số byte cần cấp phát, và trả về con trỏ void tới
khối bộ nhớ vừa cấp phát.

Vì nó là `void*`, bạn có thể gán vào bất kỳ kiểu con trỏ nào tuỳ ý,
thường con trỏ đó sẽ ứng theo cách nào đó với số byte bạn đang cấp
phát.

[i[`sizeof` operator-->with `malloc()`]<]
Vậy nên cấp phát bao nhiêu byte? Ta có thể dùng `sizeof` giúp cho
chuyện đó. Nếu muốn cấp phát đủ chỗ cho một `int`, ta có thể dùng
`sizeof(int)` và truyền vào `malloc()`.
[i[`sizeof` operator-->with `malloc()`]>]

[i[`free()` function]<]
Sau khi xong việc với đoạn bộ nhớ đã cấp phát, ta gọi `free()` để báo
đã xong với bộ nhớ đó và có thể dùng cho việc khác. Đối số là cùng
con trỏ bạn nhận từ `malloc()` (hoặc bản sao của nó). Dùng vùng bộ
nhớ sau khi `free()` là undefined behavior.

Thử xem. Ta sẽ cấp phát đủ chỗ cho một `int`, rồi lưu cái gì đó vào
đó, rồi in ra.

``` {.c}
// Allocate space for a single int (sizeof(int) bytes-worth):

int *p = malloc(sizeof(int));

*p = 12;  // Store something there

printf("%d\n", *p);  // Print it: 12

free(p);  // All done with that memory

//*p = 3490;  // ERROR: undefined behavior! Use after free()!
```
[i[`free()` function]>]

Trong ví dụ bịa đặt đó, thực sự cũng chẳng lợi gì. Ta có thể dùng một
`int` automatic và nó vẫn chạy. Nhưng rồi sẽ thấy khả năng cấp phát
bộ nhớ cách này có những lợi thế, đặc biệt với các cấu trúc dữ liệu
phức tạp hơn.

[i[`sizeof` operator-->with `malloc()`]<]
Một điều khác hay gặp tận dụng chuyện `sizeof` có thể cho biết kích
cỡ của kiểu kết quả của bất kỳ biểu thức hằng nào. Nên bạn cũng có
thể đặt tên biến vào đó dùng. Đây là ví dụ, y như cái trước:

``` {.c}
int *p = malloc(sizeof *p);  // *p is an int, so same as sizeof(int)
```
[i[`sizeof` operator-->with `malloc()`]>]
[i[`malloc()` function]>]

## Kiểm lỗi

[i[`malloc()` function-->error checking]<]
Mọi hàm cấp phát đều trả về con trỏ tới vùng bộ nhớ vừa cấp phát, hoặc
`NULL` nếu vì lý do nào đó bộ nhớ không cấp phát được.

Một số OS như Linux có thể được cấu hình sao cho `malloc()` không bao
giờ trả `NULL`, kể cả khi hết bộ nhớ. Nhưng dù vậy, bạn vẫn luôn nên
viết code có phòng bị.

``` {.c}
int *x;

x = malloc(sizeof(int) * 10);

if (x == NULL) {
    printf("Error allocating 10 ints\n");
    // do something here to handle it
}
```

Đây là mẫu phổ biến bạn sẽ thấy, gán và kiểm tra trên cùng một dòng:

``` {.c}
int *x;

if ((x = malloc(sizeof(int) * 10)) == NULL) {
    printf("Error allocating 10 ints\n");
    // do something here to handle it
}
```
[i[`malloc()` function-->error checking]>]

## Cấp phát cho mảng

[i[`malloc()` function-->and arrays]<]
Ta đã xem cách cấp phát cho một thứ, giờ thế nào với một đống chúng
trong mảng?

Trong C, mảng là một đống những thứ giống nhau xếp sát nhau trong một
vùng bộ nhớ liên tục.

Ta có thể cấp phát một vùng bộ nhớ liên tục, đã thấy cách rồi. Nếu
muốn 3490 byte bộ nhớ, cứ đòi:

``` {.c}
char *p = malloc(3490);  // Voila
```

Và, đúng thế!, đó là mảng 3490 `char` (cũng là một chuỗi!) vì mỗi
`char` là 1 byte. Nói cách khác, `sizeof(char)` là `1`.

Chú ý: bộ nhớ vừa cấp phát không được khởi tạo gì, đầy rác. Dọn sạch
bằng `memset()` nếu cần, hoặc xem `calloc()` ở dưới.

Nhưng ta chỉ cần nhân kích cỡ thứ ta muốn với số phần tử muốn, rồi
truy cập bằng ký hiệu con trỏ hay ký hiệu mảng. Ví dụ!

[i[`sizeof` operator-->with `malloc()`]<]
``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    // Allocate space for 10 ints
    int *p = malloc(sizeof(int) * 10);

    // Assign them values 0-45:
    for (int i = 0; i < 10; i++)
        p[i] = i * 5;

    // Print all values 0, 5, 10, 15, ..., 40, 45
    for (int i = 0; i < 10; i++)
        printf("%d\n", p[i]);

    // Free the space
    free(p);
}
```

Chìa khoá ở dòng `malloc()`. Nếu biết mỗi `int` chiếm `sizeof(int)`
byte, và muốn 10 cái, ta cấp phát đúng chừng đó byte với:

``` {.c}
sizeof(int) * 10
```

Và mánh này chạy cho mọi kiểu. Cứ truyền vào `sizeof` rồi nhân với
kích cỡ mảng.
[i[`sizeof` operator-->with `malloc()`]>]
[i[`malloc()` function-->and arrays]>]

## Phương án khác: `calloc()`

[i[`calloc()` function]<]
Đây là một hàm cấp phát nữa, hoạt động tương tự `malloc()`, với hai
khác biệt then chốt:

* Thay vì một đối số, bạn truyền kích cỡ của một phần tử, và số phần
  tử muốn cấp phát. Cứ như sinh ra để cấp phát mảng.
* Nó xoá bộ nhớ về không.

Bạn vẫn dùng `free()` để giải phóng bộ nhớ lấy qua `calloc()`.

Đây là so sánh giữa `calloc()` và `malloc()`.

``` {.c}
// Allocate space for 10 ints with calloc(), initialized to 0:
int *p = calloc(10, sizeof(int));

// Allocate space for 10 ints with malloc(), initialized to 0:
int *q = malloc(10 * sizeof(int));
memset(q, 0, 10 * sizeof(int));   // set to 0
```

Lần nữa, kết quả giống nhau cả hai, chỉ khác `malloc()` không xoá bộ
nhớ về 0 mặc định.
[i[`calloc()` function]>]

## Đổi kích cỡ đã cấp phát với `realloc()`

[i[`realloc()` function]<]
Nếu bạn đã cấp phát 10 `int`, nhưng sau lại quyết định cần 20, làm
sao?

Một lựa chọn là cấp phát chỗ mới rồi `memcpy()` sang... nhưng hoá ra
đôi khi không cần dịch chuyển gì. Và có một hàm vừa đủ thông minh để
làm điều đúng trong mọi trường hợp đúng: `realloc()`.

Nó nhận con trỏ tới vùng bộ nhớ đã cấp phát trước (qua `malloc()`
hoặc `calloc()`) và kích cỡ mới cho vùng bộ nhớ đó.

Nó sẽ nới rộng hoặc thu nhỏ bộ nhớ đó, rồi trả về con trỏ tới nó. Đôi
khi nó có thể trả về cùng con trỏ (nếu dữ liệu không cần chuyển đi
đâu), hoặc con trỏ khác (nếu dữ liệu phải bị chép đi).

Chắc chắn khi gọi `realloc()`, bạn phải chỉ định số _byte_ cần cấp
phát, không phải số phần tử mảng! Tức là:

``` {.c}
num_floats *= 2;

np = realloc(p, num_floats);  // WRONG: need bytes, not number of elements!

np = realloc(p, num_floats * sizeof(float));  // Better!
```

Cấp phát một mảng 20 `float`, rồi đổi ý thành 40 cái.

Ta sẽ gán giá trị trả về của realloc() vào một con trỏ khác để kiểm
tra xem nó có phải NULL không. Nếu không phải, ta có thể gán lại vào
con trỏ gốc. (Nếu gán thẳng giá trị trả về vào con trỏ gốc, ta sẽ mất
con trỏ đó nếu hàm trả về `NULL` mà không có cách nào lấy lại.)

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    // Allocate space for 20 floats
    float *p = malloc(sizeof *p * 20);  // sizeof *p same as sizeof(float)

    // Assign them fractional values 0.0-1.0:
    for (int i = 0; i < 20; i++)
        p[i] = i / 20.0;

    // But wait! Let's actually make this an array of 40 elements
    float *new_p = realloc(p, sizeof *p * 40);

    // Check to see if we successfully reallocated
    if (new_p == NULL) {
        printf("Error reallocing\n");
        return 1;
    }

    // If we did, we can just reassign p
    p = new_p;

    // And assign the new elements values in the range 1.0-2.0
    for (int i = 20; i < 40; i++)
        p[i] = 1.0 + (i - 20) / 20.0;

    // Print all values 0.0-2.0 in the 40 elements:
    for (int i = 0; i < 40; i++)
        printf("%f\n", p[i]);

    // Free the space
    free(p);
}
```

Chú ý chỗ ta lấy giá trị trả về từ `realloc()` rồi gán lại vào cùng
biến con trỏ `p` đã truyền vào. Chuyện này khá phổ biến.

Cũng vậy, nếu dòng 7 trông lạ, với `sizeof *p` ở đó, nhớ rằng `sizeof`
hoạt động trên kích cỡ của kiểu của biểu thức. Và kiểu của `*p` là
`float`, nên dòng đó tương đương với `sizeof(float)`.

Cuối cùng, có thể hơi lạ là tôi không có `free(new_p)` ở đâu cả, dù
đó là con trỏ trả về từ `realloc()`. Lý do là ta chép `new_p` vào `p`
ở dòng 23, nên cả hai cùng giá trị; cả hai trỏ tới cùng một khối bộ
nhớ, và chỉ có một khối. Nên khi `free()`, thực ra tôi free cái nào
cũng được kết quả như nhau.

[i[`realloc()` function]>]


### Đọc dòng có độ dài bất kỳ

Tôi muốn minh hoạ hai chuyện bằng ví dụ đầy đủ này.

1. Dùng `realloc()` để nới buffer khi đọc thêm dữ liệu.
2. Dùng `realloc()` để co buffer về kích cỡ hoàn hảo sau khi đã đọc
   xong.

Thứ ta thấy đây là một vòng lặp gọi `fgetc()` lặp đi lặp lại để nối
vào buffer đến khi thấy ký tự cuối là newline.

Khi tìm thấy newline, nó thu buffer về đúng kích cỡ rồi trả về.

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

// Read a line of arbitrary size from a file
//
// Returns a pointer to the line.
// Returns NULL on EOF or error.
//
// It's up to the caller to free() this pointer when done with it.
//
// Note that this strips the newline from the result. If you need
// it in there, probably best to switch this to a do-while.

char *readline(FILE *fp)
{
    int offset = 0;   // Index next char goes in the buffer
    int bufsize = 4;  // Preferably power of 2 initial size
    char *buf;        // The buffer
    int c;            // The character we've read in

    buf = malloc(bufsize);  // Allocate initial buffer

    if (buf == NULL)   // Error check
        return NULL;

    // Main loop--read until newline or EOF
    while (c = fgetc(fp), c != '\n' && c != EOF) {

        // Check if we're out of room in the buffer accounting
        // for the extra byte for the NUL terminator
        if (offset == bufsize - 1) {  // -1 for the NUL terminator
            bufsize *= 2;  // 2x the space

            char *new_buf = realloc(buf, bufsize);

            if (new_buf == NULL) {
                free(buf);   // On error, free and bail
                return NULL;
            }

            buf = new_buf;  // Successful realloc
        }

        buf[offset++] = c;  // Add the byte onto the buffer
    }

    // We hit newline or EOF...

    // If at EOF and we read no bytes, free the buffer and
    // return NULL to indicate we're at EOF:
    if (c == EOF && offset == 0) {
        free(buf);
        return NULL;
    }

    // Shrink to fit
    if (offset < bufsize - 1) {  // If we're short of the end
        char *new_buf = realloc(buf, offset + 1); // +1 for NUL terminator

        // If successful, point buf to new_buf;
        // otherwise we'll just leave buf where it is
        if (new_buf != NULL)
            buf = new_buf;
    }

    // Add the NUL terminator
    buf[offset] = '\0';

    return buf;
}

int main(void)
{
    FILE *fp = fopen("foo.txt", "r");

    char *line;

    while ((line = readline(fp)) != NULL) {
        printf("%s\n", line);
        free(line);
    }

    fclose(fp);
}
```

Khi nới bộ nhớ kiểu này, thường (dù chẳng phải luật) là nhân đôi
không gian cần thiết ở mỗi bước để giảm số lần `realloc()`.

Cuối cùng bạn có thể để ý rằng `readline()` trả về con trỏ tới buffer
được `malloc()`. Vì vậy, caller có trách nhiệm `free()` tường minh bộ
nhớ đó khi xong việc.

### `realloc()` với `NULL`

[i[`realloc()` function-->with `NULL` argument]<]
Giờ tới Trivia! Hai dòng này tương đương:

``` {.c}
char *p = malloc(3490);
char *p = realloc(NULL, 3490);
```

Có thể tiện khi bạn có vòng lặp cấp phát và không muốn xử lý riêng
cho `malloc()` lần đầu.

``` {.c}
int *p = NULL;
int length = 0;

while (!done) {
    // Allocate 10 more ints:
    length += 10;
    p = realloc(p, sizeof *p * length);

    // Do amazing things
    // ...
}
```

Trong ví dụ đó, ta không cần `malloc()` khởi tạo vì `p` ban đầu là
`NULL`.
[i[`realloc()` function-->with `NULL` argument]>]

## Cấp phát có canh lề

[i[Memory alignment]<]
Chắc bạn sẽ không phải dùng cái này.

Và tôi không muốn đi quá xa chỗ cỏ dại bàn về nó ngay bây giờ, nhưng
có một thứ gọi là _memory alignment_ (canh lề bộ nhớ), liên quan tới
chuyện địa chỉ bộ nhớ (giá trị con trỏ) là bội của một số cụ thể.

Ví dụ, hệ thống có thể yêu cầu các giá trị 16-bit phải bắt đầu ở địa
chỉ bộ nhớ là bội của 2. Hoặc giá trị 64-bit phải bắt đầu ở địa chỉ
là bội của 2, 4, hay 8, chẳng hạn. Tuỳ CPU.

Vài hệ thống yêu cầu kiểu canh lề này để truy cập bộ nhớ nhanh, hoặc
một số hệ thì để truy cập được bộ nhớ tí nào.

Nếu bạn dùng `malloc()`, `calloc()`, hay `realloc()`, C sẽ đưa bạn
một khối bộ nhớ được canh lề ổn cho bất kỳ giá trị nào, kể cả
`struct`. Chạy trong mọi trường hợp.

Nhưng có thể có lúc bạn biết một số dữ liệu canh được theo biên nhỏ
hơn, hoặc vì lý do nào đó phải canh theo biên lớn hơn. Tôi đoán
chuyện này phổ biến hơn với lập trình hệ nhúng.

[i[`aligned_alloc()` function]<]
Trong những trường hợp đó, bạn có thể chỉ định một alignment với
`aligned_alloc()`.

Alignment là số nguyên luỹ thừa của hai lớn hơn không, nên là `2`,
`4`, `8`, `16`, v.v. và bạn đưa cái đó cho `aligned_alloc()` trước
số byte bạn quan tâm.

Ràng buộc kia là số byte bạn cấp phát phải là bội của alignment.
Nhưng chuyện này có thể đang thay đổi. Xem [fl[C Defect Report
460|http://www.open-std.org/jtc1/sc22/wg14/www/docs/summary.htm#dr_460]]

Ví dụ, cấp phát trên biên 64-byte:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Allocate 256 bytes aligned on a 64-byte boundary
    char *p = aligned_alloc(64, 256);  // 256 == 64 * 4

    // Copy a string in there and print it
    strcpy(p, "Hello, world!");
    printf("%s\n", p);

    // Free the space
    free(p);
}
```

Tôi muốn ghi chú ở đây về `realloc()` và `aligned_alloc()`. `realloc()`
không có bảo đảm gì về alignment, nên nếu bạn cần lấy được vùng cấp
phát lại đã canh lề, bạn sẽ phải làm kiểu khó với `memcpy()`.
[i[`aligned_alloc()` function]>]

Đây là một hàm `aligned_realloc()` không chuẩn, nếu bạn cần:

``` {.c}
void *aligned_realloc(void *ptr, size_t old_size, size_t alignment, size_t size)
{
    char *new_ptr = aligned_alloc(alignment, size);

    if (new_ptr == NULL)
        return NULL;

    size_t copy_size = old_size < size? old_size: size;  // get min

    if (ptr != NULL)
        memcpy(new_ptr, ptr, copy_size);

    free(ptr);

    return new_ptr;
}
```

Chú ý rằng nó _luôn luôn_ chép dữ liệu, tốn thời gian, trong khi
`realloc()` thật sẽ tránh chép nếu có thể. Nên nó kém hiệu quả. Tránh
phải cấp phát lại dữ liệu canh lề tuỳ chỉnh.
[i[Memory alignment]>]
[i[Manual memory management]>]

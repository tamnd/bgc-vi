<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Dự án nhiều file

[i[Multifile projects]<]

Từ đầu đến giờ ta chỉ xem mấy chương trình đồ chơi mà phần lớn đều
nhét vừa trong một file. Nhưng chương trình C phức tạp được tạo từ
nhiều file, tất cả được biên dịch và link lại thành một file thực
thi.

Chương này ta sẽ xem vài mẫu và lối làm thường gặp khi ghép các dự
án lớn hơn lại với nhau.

## Include và function prototype  {#includes-func-protos}

[i[Multifile projects-->includes]<]
[i[Multifile projects-->function prototypes]<]

Một tình huống rất phổ biến là vài hàm của bạn được định nghĩa trong
một file, và bạn muốn gọi chúng từ file khác.

Chuyện này thực ra chạy được ngay với một cảnh báo, cứ thử trước rồi
ta xem cách đúng để dẹp cảnh báo đó.

Để biên dịch, bạn cần chỉ định mọi file nguồn trên dòng lệnh:

``` {.zsh}
# output file   source files
#     v            v
#   |----| |---------|
gcc -o foo foo.c bar.c
```

Trong ví dụ đó, `foo.c` và `bar.c` được build thành file thực thi
tên `foo`.

Với mấy ví dụ này, ta để tên file như comment đầu tiên trong nguồn.
Xem file nguồn `bar.c`:

``` {.c .numberLines}
// File bar.c

int add(int x, int y)
{
    return x + y;
}
```

Và file `foo.c` có main trong đó:

``` {.c .numberLines}
// File foo.c

#include <stdio.h>

int main(void)
{
    printf("%d\n", add(2, 3));  // 5!
}
```

Thấy cách từ `main()` ta gọi `add()` chứ, mà `add()` lại nằm trong
một file nguồn hoàn toàn khác! Nó ở `bar.c`, còn lời gọi tới nó nằm
trong `foo.c`!

Nếu build cái này bằng:

``` {.zsh}
gcc -o foo foo.c bar.c
```

ta sẽ nhận được lỗi này:

``` {.default}
error: implicit declaration of function 'add' is invalid in C99
```

(Hoặc bạn có thể nhận được cảnh báo. Mà thứ bạn không nên bỏ qua.
Đừng bao giờ bỏ qua cảnh báo trong C, xử lý hết đi.)

Nếu bạn còn nhớ từ [phần về prototype](#prototypes), khai báo ngầm
bị cấm trong C hiện đại và không có lý do chính đáng nào để đưa
chúng vào code mới. Ta nên sửa nó.

`implicit declaration` nghĩa là ta đang dùng một hàm, ở đây là
`add()`, mà không cho C biết trước cái gì về nó cả. C muốn biết nó
trả về gì, nhận kiểu gì làm đối số, và các thứ kiểu vậy.

Ta đã thấy cách sửa chuyện đó từ trước với _function prototype_.
Đúng thế, nếu ta thêm một cái vào `foo.c` trước khi gọi, mọi thứ sẽ
ổn:

``` {.c .numberLines}
// File foo.c

#include <stdio.h>

int add(int, int);  // Add the prototype

int main(void)
{
    printf("%d\n", add(2, 3));  // 5!
}
```

Hết lỗi!

Nhưng chuyện đó cũng nhọc, phải gõ prototype mỗi khi muốn dùng một
hàm. Ủa kìa, ta vừa dùng `printf()` ngay đó mà đâu cần gõ prototype,
vậy là sao?

_Thật ra ta đã include prototype cho `printf()` rồi_! Nó ở trong file
`stdio.h`! Và ta đã include file đó bằng `#include`!

Ta làm tương tự với hàm `add()` của mình được không? Làm prototype
cho nó và nhét vào một file header?

Dĩ nhiên được!

Header file trong C theo quy ước có phần mở rộng `.h`. Và chúng
thường, dù không phải luôn luôn, có cùng tên với file `.c` tương
ứng. Vậy ta tạo file `bar.h` cho file `bar.c`, và nhét prototype vào
đó:

``` {.c .numberLines}
// File bar.h

int add(int, int);
```

Giờ sửa `foo.c` để include file đó. Giả sử nó ở cùng thư mục, ta
include nó bên trong dấu nháy kép (thay vì dấu ngoặc nhọn):

``` {.c .numberLines}
// File foo.c

#include <stdio.h>

#include "bar.h"  // Include from current directory

int main(void)
{
    printf("%d\n", add(2, 3));  // 5!
}
```

Chú ý ta không còn prototype trong `foo.c` nữa, ta include nó từ
`bar.h`. Giờ _bất cứ_ file nào muốn dùng chức năng `add()` chỉ cần
`#include "bar.h"` là có, không cần lo chuyện gõ prototype của hàm.

Như bạn có thể đoán, `#include` theo đúng nghĩa đen đưa file được
gọi tên _ngay vào đó_ trong mã nguồn của bạn, y như là bạn đã gõ
vào.

Rồi build và chạy:

``` {.zsh}
./foo
5
```

Đúng rồi, ta nhận được kết quả $2+3$! Hú hồn!

Nhưng đừng vội khui chai đồ uống yêu thích. Gần xong thôi! Còn một
mẩu boilerplate nữa phải thêm.

[i[Multifile projects-->function prototypes]>]

## Xử lý include bị lặp

Cũng không hiếm chuyện một file header lại `#include` các header
khác cần cho chức năng của các file C tương ứng. Kiểu, sao không?

Và có thể bạn có một header được `#include` nhiều lần từ nhiều chỗ
khác nhau. Có khi chẳng sao, có khi lại gây lỗi compiler. Và ta
không kiểm soát được có bao nhiêu chỗ `#include` nó!

Tệ hơn, có khi ta rơi vào tình huống điên rồ kiểu header `a.h`
include header `b.h`, và `b.h` lại include `a.h`! Đúng là chu kỳ
`#include` vô hạn!

Thử build một thứ như vậy sẽ báo lỗi:

``` {.default}
error: #include nested depth 200 exceeds maximum of 200
```

Biết đâu bước thứ 201 nó đã giải được chu kỳ...

Việc ta cần làm là nếu một file đã được include một lần rồi, các
`#include` sau cho cùng file đó sẽ bị lờ đi.

**Mấy thứ ta sắp làm phổ biến đến mức mà bạn cứ tự động làm mỗi lần
tạo file header!**

Và cách phổ biến để làm chuyện này là một biến preprocessor mà ta
đặt vào lần đầu tiên `#include` file. Rồi với các `#include` sau, ta
kiểm tra trước để chắc rằng biến đó chưa được định nghĩa.

Về tên biến, cực kỳ phổ biến việc lấy tên file header, như `bar.h`,
viết hoa lên, và thay dấu chấm bằng gạch dưới: `BAR_H`.

Vậy đặt một kiểm tra ở sát đầu file xem nó đã được include chưa, và
coi như comment cả file đi nếu rồi.

(Đừng đặt gạch dưới ở đầu (vì gạch dưới đầu theo sau là chữ hoa đã
được reserved) hay hai gạch dưới ở đầu (vì cái đó cũng được
reserved).)

``` {.c .numberLines}
#ifndef BAR_H   // If BAR_H isn't defined...
#define BAR_H   // Define it (with no particular value)

// File bar.h

int add(int, int);

#endif          // End of the #ifndef BAR_H
```

Cái này sẽ khiến file header chỉ được include đúng một lần, bất kể
bao nhiêu chỗ cố `#include` nó.

[i[Multifile projects-->includes]>]

## `static` và `extern`

[i[`static` storage class]<]
[i[`extern` storage class]<]
[i[Multifile projects-->`static` storage class]<]
[i[Multifile projects-->`extern` storage class]<]

Bạn có thể tham chiếu đến các đối tượng ở file khác bằng `extern`.

Bạn có thể đảm bảo biến và hàm ở file scope _không_ nhìn thấy được
từ các file nguồn khác (dù có `extern`) bằng từ khóa `static`.

Thêm thông tin, xem các phần trong sách về storage-class specifier
[`static`](#static) và [`extern`](#extern).

[i[`static` storage class]>]
[i[`extern` storage class]>]
[i[Multifile projects-->`static` storage class]>]
[i[Multifile projects-->`extern` storage class]>]

## Biên dịch với object file

[i[Object files]<]

Chuyện này không có trong spec, nhưng nó là 99.999% phổ biến trong
thế giới C.

Bạn có thể biên dịch file C thành dạng biểu diễn trung gian gọi là
_object file_. Chúng chứa mã máy (tức là các bit 1 và 0 của các lệnh
thực sự) nhưng chưa được ghép thành file thực thi. 

Object file trong Windows có phần mở rộng `.OBJ`; trong các hệ
Unix-like, chúng là `.o`.

[i[`gcc` compiler]<]

Trong gcc, ta có thể build mấy cái đó thế này, với cờ `-c` (chỉ
compile thôi!):

``` {.zsh}
gcc -c foo.c     # produces foo.o
gcc -c bar.c     # produces bar.o
```

Rồi ta có thể _link_ chúng lại thành một file thực thi duy nhất:

``` {.zsh}
gcc -o foo foo.o bar.o
```

_Voila_, ta đã tạo ra file thực thi `foo` từ hai object file.

Nhưng bạn nghĩ, tội gì cho khổ? Chẳng phải ta có thể:

``` {.zsh}
gcc -o foo foo.c bar.c
```

và [flw[hạ|Boids]] hai con chim bằng một viên đá?

[i[`gcc` compiler]>]

Với chương trình nhỏ thì ổn. Tôi vẫn làm vậy suốt.

Nhưng với chương trình lớn hơn, ta có thể tận dụng chuyện biên dịch
từ nguồn ra object file thì tương đối chậm, còn link một đống object
file lại thì tương đối nhanh.

Điều này thể hiện rõ nhất với công cụ `make`, thứ chỉ build lại
những nguồn mới hơn output của chúng.

Giả sử bạn có một nghìn file C. Ban đầu bạn compile tất cả chúng
thành object file (chậm) rồi gộp tất cả các object file đó thành
file thực thi (nhanh).

Giờ giả sử bạn sửa đúng một trong số các file nguồn C đó, đây mới là
phép màu: _bạn chỉ cần build lại đúng object file cho file nguồn
đó_! Rồi build lại file thực thi (nhanh). Mọi file C khác không cần
đụng tới.

Nói cách khác, nhờ chỉ build lại những object file cần, ta cắt giảm
thời gian compile dữ dội. (Dĩ nhiên trừ khi bạn làm build "clean",
khi đó tất cả object file đều phải được tạo lại.)

[i[Object files]>]
[i[Multifile projects]>]

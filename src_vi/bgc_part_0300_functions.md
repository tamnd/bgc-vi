<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Hàm {#functions}

> _"Sir, not in an environment such as this. That's why I've also been
> programmed for over thirty secondary functions that---"_
>
> ---C3PO[i[C3PO]], before being rudely interrupted, reporting a
> now-unimpressive number of additional functions, _Star Wars_ script

[i[Functions]<]
Rất giống với các ngôn ngữ khác bạn đã quen, C có khái niệm _function_
(hàm).

Hàm có thể nhận nhiều loại _argument_ (đối số)[i[Function arguments]]
và trả về một giá trị. Có một điều quan trọng: kiểu của đối số và giá
trị trả về phải được khai báo trước, vì C thích thế!

Hãy nhìn một hàm. Đây là hàm nhận một `int` làm đối số, và trả
về[i[`return` statement]] một `int`.

``` {.c .numberLines}
#include <stdio.h>

int plus_one(int n)  // The "definition"
{
    return n + 1;
}
 
```

Chữ `int` trước `plus_one` chỉ kiểu trả về.

`int n` chỉ rằng hàm nhận một đối số `int`, được lưu trong _parameter_
(tham số) `n`[i[Function parameters]]. Parameter là một loại biến cục
bộ đặc biệt mà đối số được sao chép vào.

Tôi sẽ nhấn mạnh rằng đối số được _sao chép_ vào parameter. Nhiều thứ
trong C dễ hiểu hơn nếu bạn biết parameter là một _bản sao_ của đối
số, chứ không phải bản thân đối số. Nói thêm sau một chút.

Đi tiếp xuống `main()`, ta có thể thấy lời gọi hàm, nơi ta gán giá trị
trả về vào biến cục bộ `j`:

``` {.c .numberLines startFrom="8"}
int main(void)
{
    int i = 10, j;
    
    j = plus_one(i);  // The "call"

    printf("i + 1 is %d\n", j);
}
```

> Trước khi quên, để ý rằng tôi đã định nghĩa hàm trước khi dùng nó.
> Nếu không làm vậy, trình biên dịch sẽ chưa biết gì về hàm khi biên
> dịch `main()` và sẽ văng lỗi gọi hàm không xác định. Có cách chuẩn
> chỉnh hơn để viết đoạn code trên bằng _function prototype_ (nguyên
> mẫu hàm), nhưng sẽ nói sau.

Để ý luôn rằng `main()`[i[`main()` function]] cũng là một hàm!

Nó trả về `int`.

Nhưng cái `void`[i[`void` type]] này là gì? Đây là một keyword dùng để
nói rằng hàm không nhận đối số nào.

Bạn cũng có thể trả về `void` để nói rằng bạn không trả về giá trị nào:

``` {.c .numberLines}
#include <stdio.h>

// This function takes no arguments and returns no value:

void hello(void)
{
    printf("Hello, world!\n");
}

int main(void)
{
    hello();  // Prints "Hello, world!"
}
```

## Truyền theo giá trị {#passvalue}

[i[Pass by value]()]Tôi đã nói trước đó rằng khi bạn truyền một đối số
cho hàm, một bản sao của đối số đó được tạo ra và lưu vào parameter
tương ứng.

Nếu đối số là một biến, một bản sao của giá trị biến đó được tạo ra và
lưu vào parameter.

Tổng quát hơn, toàn bộ biểu thức đối số được tính ra giá trị. Giá trị
đó được sao chép vào parameter.

Trong mọi trường hợp, giá trị trong parameter là của riêng nó. Nó độc
lập với bất kỳ giá trị hay biến nào bạn dùng làm đối số khi gọi hàm.

Hãy xem một ví dụ. Nghiên cứu và thử đoán output trước khi chạy:

``` {.c .numberLines}
#include <stdio.h>

void increment(int a)
{
    a++;
}

int main(void)
{
    int i = 10;

    increment(i);

    printf("i == %d\n", i);  // What does this print?
}
```

Thoạt nhìn, có vẻ `i` là `10`, và ta truyền nó vào hàm `increment()`.
Ở đó giá trị được tăng, nên khi in ra phải là `11` đúng không?

> _"Get used to disappointment."_
>
> ---Dread Pirate Roberts, _The Princess Bride_

Nhưng không phải `11`, nó in ra `10`! Sao thế?

Mọi chuyện nằm ở việc biểu thức bạn truyền vào hàm được _sao chép_ vào
parameter tương ứng. Parameter là bản sao, không phải bản gốc.

Vậy `i` là `10` ngoài `main()`. Và ta truyền nó vào `increment()`.
Parameter tương ứng có tên là `a` trong hàm đó.

Và phép sao chép xảy ra, như thể là một phép gán. Đại khái, `a = i`.
Nên tại thời điểm đó, `a` là `10`. Và ngoài `main()`, `i` cũng là
`10`.

Rồi ta tăng `a` lên `11`. Nhưng ta không chạm vào `i` chút nào! Nó vẫn
là `10`.

Cuối cùng, hàm kết thúc. Tất cả biến cục bộ bị bỏ đi (chào nhé, `a`!)
và ta quay lại `main()`, nơi `i` vẫn là `10`.

Rồi ta in ra, được `10`, và xong.

Đây là lý do trong ví dụ trước với hàm `plus_one()`, ta đã `return`
giá trị đã bị sửa cục bộ để có thể thấy nó lại trong `main()`.

Nghe hạn chế nhỉ? Kiểu như bạn chỉ lấy về được một mẩu dữ liệu từ hàm,
bạn đang nghĩ vậy đấy. Tuy nhiên, còn một cách khác để lấy dữ liệu về;
dân C gọi cách đó là _passing by reference_ (truyền theo tham chiếu)
và đó là câu chuyện để dành dịp khác.

Nhưng đừng để cái tên hoa lá cành đó đánh lừa bạn khỏi sự thật rằng
_MỌI THỨ_ bạn truyền vào hàm, _KHÔNG NGOẠI LỆ_, đều được sao chép vào
parameter tương ứng, và hàm thao tác trên bản sao cục bộ đó, _BẤT KỂ
THẾ NÀO_. Nhớ lấy, ngay cả khi ta đang nói về cái gọi là truyền theo
tham chiếu.
[i[Pass by value]>]

## Function Prototype {#prototypes}

[i[Function prototypes]<]Nếu bạn còn nhớ từ thời kỳ băng hà vài mục
trước, tôi có nói rằng bạn phải định nghĩa hàm trước khi dùng nó,
không thì trình biên dịch sẽ chưa biết gì về hàm, và sẽ văng lỗi.

Điều này không hoàn toàn nghiêm ngặt đúng. Bạn có thể báo trước cho
trình biên dịch rằng bạn sẽ dùng một hàm có kiểu nhất định với danh
sách parameter nhất định. Như thế, hàm có thể được định nghĩa ở đâu
cũng được (kể cả ở file khác), miễn là _function prototype_ đã được
khai báo trước khi bạn gọi hàm đó.

May thay, function prototype thực ra khá dễ. Nó chỉ là bản sao của
dòng đầu tiên trong định nghĩa hàm, kèm thêm dấu chấm phẩy ở cuối cho
chắc ăn. Ví dụ, đoạn code này gọi một hàm được định nghĩa ở phía sau,
vì prototype đã được khai báo trước:

``` {.c .numberLines}
#include <stdio.h>

int foo(void);  // This is the prototype!

int main(void)
{
    int i;
    
    // We can call foo() here before it's definition because the
    // prototype has already been declared, above!

    i = foo();
    
    printf("%d\n", i);  // 3490
}

int foo(void)  // This is the definition, just like the prototype!
{
    return 3490;
}
```

Nếu bạn không khai báo hàm trước khi dùng (bằng prototype hoặc bằng
định nghĩa), bạn đang làm một thứ gọi là _implicit declaration_ (khai
báo ngầm)[i[Implicit declaration]]. Chuyện này được cho phép trong
chuẩn C đầu tiên (C89), và chuẩn đó có quy định cho nó, nhưng ngày nay
không còn được phép nữa. Và không có lý do chính đáng nào để trông cậy
vào nó trong code mới.

Bạn có thể để ý một điều về các đoạn code mẫu ta đã dùng... Đó là ta
đã dùng hàm `printf()` cũ kỹ mà tốt lành mà không định nghĩa cũng
không khai báo prototype! Làm sao ta thoát được sự vô luật pháp này?
Thật ra ta không thoát đâu. Có prototype; nó nằm trong file header
`stdio.h` mà ta đã kèm vào bằng `#include`, nhớ không? Nên ta vẫn hợp
pháp đó, thưa ông cảnh sát![i[Function prototypes]>]

## Danh sách parameter rỗng

[i[Empty parameter lists]]
Bạn có thể thấy cái này đây đó trong code cũ, nhưng không bao giờ nên
viết nó trong code mới. Luôn dùng `void`[i[`void` type]] để chỉ rằng
hàm không nhận parameter nào. Không bao giờ^[Đừng bao giờ nói "không
bao giờ".] có lý do để bỏ qua chuyện này trong code hiện đại.

Nếu bạn giỏi nhớ việc bỏ `void` vào cho danh sách parameter rỗng trong
hàm và prototype, bạn có thể bỏ qua phần còn lại của mục này.

Có hai bối cảnh cho chuyện này:

* Bỏ hết parameter khi định nghĩa hàm
* Bỏ hết parameter trong prototype

Trước tiên xem định nghĩa hàm tiềm tàng:

``` {.c}
void foo()  // Should really have a `void` in there
{
    printf("Hello, world!\n");
}
```

Dù spec có nói hành vi trong trường hợp này _như thể_ bạn đã ghi
`void` (C11 §6.7.6.3¶14), kiểu `void` ở đó có lý do. Hãy dùng nó.

Nhưng trong trường hợp function prototype, có một khác biệt _đáng kể_
giữa dùng `void`[i[`void` type-->in function prototypes]] và không:

``` {.c}
void foo();
void foo(void);  // Not the same!
```

Bỏ `void` khỏi prototype báo cho trình biên dịch rằng không có thông
tin thêm về các parameter của hàm. Nó hiệu quả tắt hết mọi kiểm tra
kiểu.

Với prototype, **chắc chắn** dùng `void` khi bạn có danh sách
parameter rỗng.

[i[Functions]>]

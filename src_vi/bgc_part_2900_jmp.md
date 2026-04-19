<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Long Jump với `setjmp`, `longjmp` {#setjmp-longjmp}

[i[Long jumps]<]

Ta đã thấy `goto`, nhảy trong scope hàm. Nhưng `longjmp()` cho phép
bạn nhảy ngược về một điểm sớm hơn trong thực thi, về một hàm đã gọi
hàm này.

Có cả đống hạn chế và cảnh báo, nhưng đây có thể là hàm hữu ích để
thoát từ sâu trong call stack ngược lên trạng thái sớm hơn.

Theo kinh nghiệm của tôi, chức năng này rất hiếm khi được dùng.

## Dùng `setjmp` và `longjmp`

[i[`setjmp()` function]<]
[i[`longjmp()` function]<]

Vũ điệu ta sẽ làm ở đây là về cơ bản đặt một bookmark trong thực
thi với `setjmp()`. Sau đó, ta gọi `longjmp()` và nó sẽ nhảy về điểm
sớm hơn trong thực thi nơi ta đặt bookmark bằng `setjmp()`.

Và nó có thể làm chuyện này ngay cả khi bạn đã gọi các hàm con.

Đây là demo nhanh trong đó ta gọi vào các hàm sâu vài cấp rồi thoát
ra khỏi nó.

Ta sẽ dùng biến file scope `env` để giữ _state_ khi gọi `setjmp()`
để có thể khôi phục khi gọi `longjmp()` sau này. Đây là biến ta nhớ
"vị trí" của mình.

Biến `env` thuộc kiểu `jmp_buf`, một kiểu mờ được khai báo trong
`<setjmp.h>`.

``` {.c .numberLines}
#include <stdio.h>
#include <setjmp.h>

jmp_buf env;

void depth2(void)
{
    printf("Entering depth 2\n");
    longjmp(env, 3490);           // Bail out
    printf("Leaving depth 2\n");  // This won't happen
}

void depth1(void)
{
    printf("Entering depth 1\n");
    depth2();
    printf("Leaving depth 1\n");  // This won't happen
}

int main(void)
{
    switch (setjmp(env)) {
      case 0:
          printf("Calling into functions, setjmp() returned 0\n");
          depth1();
          printf("Returned from functions\n");  // This won't happen
          break;

      case 3490:
          printf("Bailed back to main, setjmp() returned 3490\n");
          break;
    }
}
```

Khi chạy, cái này xuất ra:

``` {.default}
Calling into functions, setjmp() returned 0
Entering depth 1
Entering depth 2
Bailed back to main, setjmp() returned 3490
```

Nếu bạn cố lấy output đó và khớp với code, rõ là có chuyện gì đó rất
_quái_ đang xảy ra.

Một trong những thứ đáng chú ý nhất là `setjmp()` return _hai lần_.
Cái quái gì thế? Phép thuật gì đây?!

Vậy đây là deal: nếu `setjmp()` trả `0`, tức là bạn đã đặt
"bookmark" thành công tại điểm đó.

Nếu nó trả khác 0, tức là bạn vừa trở về "bookmark" đã đặt trước đó.
(Và giá trị trả về là giá trị bạn truyền cho `longjmp()`.)

Kiểu này bạn có thể phân biệt giữa việc đặt bookmark và trở về nó
sau này.

Nên khi code trên gọi `setjmp()` lần đầu, `setjmp()` _lưu_ state vào
biến `env` và trả về `0`. Sau đó khi ta gọi `longjmp()` với cùng
`env` đó, nó khôi phục state và `setjmp()` trả về giá trị đã truyền
cho `longjmp()`.

[i[`setjmp()` function]>]
[i[`longjmp()` function]>]

## Bẫy

Dưới mui, cái này khá thẳng thắn. Thông thường _stack pointer_ theo
dõi vị trí trong bộ nhớ nơi biến cục bộ được lưu, và _program
counter_ theo dõi địa chỉ của lệnh hiện đang thực thi^[Cả "stack
pointer" và "program counter" đều liên quan tới kiến trúc nằm dưới
và cài đặt C, và không phải phần của spec.].

Nên nếu ta muốn nhảy về hàm sớm hơn, về cơ bản chỉ là chuyện khôi
phục stack pointer và program counter về giá trị giữ trong biến
[i[`jmp_buf` type]] `jmp_buf`, và đảm bảo giá trị trả về được set
đúng. Và rồi thực thi sẽ tiếp tục ở đó.

Nhưng đủ kiểu yếu tố làm rối cái này, tạo ra một số lượng đáng kể
các bẫy hành vi không xác định.

### Giá trị của biến cục bộ

[i[`setjmp()` function]<]

Nếu bạn muốn giá trị của biến cục bộ automatic (không `static` và
không `extern`) tồn tại trong hàm đã gọi `setjmp()` sau khi một
[i[`longjmp()`]] `longjmp()` xảy ra, bạn phải khai báo các biến đó
là [i[`volatile` type qualifier-->with `setjmp()`]<] `volatile`.

Về mặt kỹ thuật, chúng chỉ cần `volatile` nếu chúng thay đổi giữa
lúc `setjmp()` được gọi và lúc `longjmp()` được gọi^[Lý lẽ ở đây là
chương trình có thể lưu giá trị tạm thời trong _CPU register_ khi nó
đang làm việc với giá trị đó. Trong khoảng thời gian đó, register
giữ giá trị đúng, và giá trị trên stack có thể đã cũ. Rồi sau đó giá
trị register bị ghi đè và các thay đổi đối với biến bị mất.].

Ví dụ, nếu ta chạy code này:

``` {.c}
int x = 20;

if (setjmp(env) == 0) {
    x = 30;
}
```

và sau đó `longjmp()` quay lại, giá trị của `x` sẽ không xác định.

Nếu ta muốn sửa chuyện này, `x` phải là `volatile`:

``` {.c}
volatile int x = 20;

if (setjmp(env) == 0) {
    x = 30;
}
```

[i[`setjmp()` function]>]

[i[`volatile` type qualifier-->with `setjmp()`]>]

Giờ giá trị sẽ là đúng `30` sau khi một [i[`longjmp()`]] `longjmp()`
đưa ta về điểm này.

### Bao nhiêu state được lưu?

[i[`setjmp()` function]<]
[i[`longjmp()` function]<]

Khi bạn `longjmp()`, thực thi tiếp tục tại điểm của `setjmp()` tương
ứng. Và thế thôi.

[i[`setjmp()` function]>]

Spec chỉ rõ nó giống như bạn đã nhảy về hàm tại điểm đó với biến cục
bộ được set về bất cứ giá trị nào chúng có tại lúc gọi `longjmp()`.

[i[`longjmp()` function]>]

Những thứ không được khôi phục bao gồm, diễn giải lại spec:

* Cờ trạng thái dấu chấm động
* File đang mở
* Bất kỳ thành phần nào khác của máy trừu tượng

### Bạn không thể đặt tên gì là `setjmp`

Bạn không thể có định danh `extern` nào với tên `setjmp`. Hoặc, nếu
`setjmp` là macro, bạn không thể undefine nó.

Cả hai đều là hành vi không xác định.

### Bạn không thể `setjmp()` trong biểu thức lớn hơn

[i[`setjmp()`-->in an expression]<]

Tức là, bạn không thể làm kiểu này:

``` {.c}
if (x == 12 && setjmp(env) == 0) { ... }
```

Chuyện đó quá phức tạp để spec cho phép vì những cỗ máy cần chạy khi
tháo stack và tất cả mấy chuyện đó. Ta không thể `longjmp()` về vào
biểu thức phức tạp nào đó mà chỉ mới thực thi một phần.

Nên có giới hạn về độ phức tạp của biểu thức đó.

* Nó có thể là toàn bộ biểu thức điều khiển của điều kiện.

  ``` {.c}
  if (setjmp(env)) {...}
  ```

  ``` {.c}
  switch (setjmp(env)) {...}
  ```

* Nó có thể là một phần của biểu thức quan hệ hay đẳng thức, miễn là
  toán hạng kia là hằng số nguyên. Và toàn bộ là biểu thức điều
  khiển của điều kiện.

  ``` {.c}
  if (setjmp(env) == 0) {...}
  ```

* Toán hạng của phép NOT logic (`!`), là toàn bộ biểu thức điều
  khiển.

  ``` {.c}
  if (!setjmp(env)) {...}
  ```

* Biểu thức đứng một mình, có thể được cast thành `void`.

  ``` {.c}
  setjmp(env);
  ```
  ``` {.c}
  (void)setjmp(env);
  ```

[i[`setjmp()`-->in an expression]>]

### Khi nào bạn không thể `longjmp()`?

[i[`lonjmp()`]<]

Là hành vi không xác định nếu:

* Bạn không gọi `setjmp()` trước đó
* Bạn gọi `setjmp()` từ thread khác
* Bạn gọi `setjmp()` trong scope của một mảng độ dài biến đổi (VLA),
  và thực thi rời khỏi scope của VLA đó trước khi `longjmp()` được
  gọi.
* Hàm chứa `setjmp()` đã thoát trước khi `longjmp()` được gọi.

Ở cái cuối, "thoát" bao gồm return bình thường khỏi hàm, cũng như
trường hợp một `longjmp()` khác nhảy về "sớm hơn" trong call stack
so với hàm đang nói tới.

### Bạn không thể truyền `0` cho `longjmp()`

Nếu bạn thử truyền giá trị `0` cho `longjmp()`, nó sẽ âm thầm đổi
giá trị đó thành `1`.

Vì `setjmp()` rốt cuộc trả giá trị này, và việc `setjmp()` trả `0`
có nghĩa đặc biệt, nên trả `0` bị cấm.

### `longjmp()` và mảng độ dài biến đổi

Nếu bạn đang trong scope của một VLA và `longjmp()` ra ngoài, bộ nhớ
cấp cho VLA có thể bị leak^[Tức là, vẫn được cấp phát tới khi chương
trình kết thúc mà không có cách giải phóng.].

Chuyện tương tự xảy ra nếu bạn `longjmp()` về qua bất kỳ hàm sớm hơn
nào vẫn còn VLA trong scope.

Đây là một thứ thực sự làm tôi thấy phiền về VLA, rằng bạn có thể
viết code C hoàn toàn hợp lệ mà phí bộ nhớ. Nhưng thôi, tôi không
phải người quyết spec.

[i[`lonjmp()`]>]
[i[Long jumps]>]

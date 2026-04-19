<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Ký tự và chuỗi II

Ta đã nói về chuyện kiểu `char` thực ra chỉ là kiểu số nguyên nhỏ,
và ký tự nằm trong dấu nháy đơn cũng vậy.

Nhưng chuỗi trong dấu nháy kép thì có kiểu `const char *`.

Hóa ra còn vài kiểu chuỗi và ký tự nữa, và nó dẫn tới một trong
những hang thỏ khét tiếng nhất của ngôn ngữ này: cả cái mớ
multibyte/wide/Unicode/localization.

Ta sẽ ghé nhìn xuống hang thỏ đó, nhưng chưa chui vào. Chưa đâu!

## Escape sequence

[i[Escape sequences]<]

Ta quen với chuỗi và ký tự gồm chữ cái, dấu câu và số thông thường:

``` {.c}
char *s = "Hello!";
char t = 'c';
```

Nhưng lỡ ta muốn nhét mấy ký tự đặc biệt mà bàn phím không gõ được
vì nó không có ở đó (ví dụ "€"), hay kể cả khi ta muốn một ký tự là
dấu nháy đơn, thì sao? Rõ ràng ta không thể viết:

``` {.c}
char t = ''';
```

[i[`\` backslash escape]<]

Để làm mấy chuyện này, ta dùng thứ gọi là _escape sequence_ (chuỗi
thoát). Nó là ký tự backslash (`\`) theo sau là một ký tự khác. Hai
(hoặc nhiều) ký tự đi với nhau mang nghĩa đặc biệt.

Với ví dụ ký tự nháy đơn, ta có thể đặt một escape (tức là `\`)
trước dấu nháy đơn ở giữa để giải quyết:

[i[`\'` single quote]<]

``` {.c}
char t = '\'';
```

Giờ C biết `\'` nghĩa là dấu nháy thật mà ta muốn in ra, chứ không
phải điểm kết thúc chuỗi ký tự.

[i[`\'` single quote]>]

Bạn có thể nói "backslash" hoặc "escape" trong ngữ cảnh này ("escape
cái nháy đó đi") và dân C sẽ hiểu bạn đang nói gì. Lưu ý "escape" ở
đây khác với phím `Esc` hay mã ASCII `ESC`.

### Mấy escape hay dùng

Theo ý tôi, mấy escape dưới đây chiếm 99.2%^[Tôi bịa con số đó,
nhưng chắc không sai lệch bao xa] của mọi escape.

[i[`\n` newline]]
[i[`\'` single quote]]
[i[`\"` double quote]]
[i[`\\` backslash]]

|Code|Mô tả|
|--|------------|
|`\n`|Ký tự newline, khi in ra, phần sau tiếp tục ở dòng kế|
|`\'`|Nháy đơn, dùng cho hằng ký tự là dấu nháy đơn|
|`\"`|Nháy kép, dùng cho dấu nháy kép trong string literal|
|`\\`|Backslash, dùng cho ký tự `\` theo đúng nghĩa trong chuỗi hay ký tự|

Vài ví dụ về escape và cái chúng in ra:

``` {.c}
printf("Use \\n for newline\n");  // Use \n for newline
printf("Say \"hello\"!\n");       // Say "hello"!
printf("%c\n", '\'');             // '
```

### Mấy escape ít dùng

Còn nhiều escape khác nữa! Chỉ là bạn ít gặp chúng hơn.

[i[`\a` alert]]
[i[`\b` backspace]]
[i[`\f` formfeed]]
[i[`\r` carriage return]]
[i[`\t` tab]]
[i[`\v` vertical tab]]
[i[`\?` question mark]]

|Code|Mô tả|
|--|--------------------|
|`\a`|Alert. Khiến terminal kêu hoặc chớp sáng, hoặc cả hai!|
|`\b`|Backspace. Lùi con trỏ về một ký tự. Không xóa ký tự đó.|
|`\f`|Formfeed. Nhảy sang "trang" tiếp theo, nhưng chuyện đó chẳng còn mấy ý nghĩa ở thời nay. Trên máy tôi, nó hành xử như `\v`.|
|`\r`|Return. Về đầu cùng dòng hiện tại.|
|`\t`|Tab ngang. Nhảy tới tab stop ngang kế tiếp. Trên máy tôi, nó dóng vào các cột là bội số của 8, nhưng YMMV.|
|`\v`|Tab dọc. Nhảy tới tab stop dọc kế tiếp. Trên máy tôi, nó nhảy sang cùng cột ở dòng kế.|
|`\?`|Dấu hỏi theo đúng nghĩa. Đôi khi bạn cần cái này để tránh trigraph, sẽ nói bên dưới.|

#### Cập nhật trạng thái trên một dòng

[i[`\b` backspace]<]
[i[`\r` carriage return]<]

Một ca dùng của `\b` hay `\r` là hiển thị cập nhật trạng thái trên
cùng một dòng màn hình mà không làm nội dung cuộn. Đây là ví dụ đếm
ngược từ 10. (Nếu compiler của bạn không hỗ trợ threading, bạn có
thể dùng hàm POSIX không chuẩn `sleep()` từ `<unistd.h>`, nếu không
ở hệ Unix-like, tìm nền tảng của bạn cộng với `sleep` để có cái
tương đương.)

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

int main(void)
{
    for (int i = 10; i >= 0; i--) {
        printf("\rT minus %d second%s... \b", i, i != 1? "s": "");

        fflush(stdout);  // Force output to update

        // Sleep for 1 second
        thrd_sleep(&(struct timespec){.tv_sec=1}, NULL);
    }

    printf("\rLiftoff!             \n");
}
```

Có kha khá chuyện xảy ra ở dòng 7. Đầu tiên, ta mở đầu bằng `\r` để
về đầu dòng hiện tại, rồi ghi đè lên bất cứ thứ gì đang ở đó bằng
đoạn đếm ngược hiện tại. (Có toán tử ternary ở đó để đảm bảo ta in
`1 second` chứ không phải `1 seconds`.)

Cũng có một khoảng trắng sau `...` Đó là để ta ghi đè đúng dấu `.`
cuối cùng khi `i` tụt từ `10` xuống `9` và ta bị hụt đi một cột. Thử
bỏ khoảng trắng đó đi để thấy tôi muốn nói gì.

Và ta kết bằng `\b` để lùi qua khoảng trắng đó cho con trỏ nằm đúng
cuối dòng, cho đẹp.

[i[`\b` backspace]>]

Chú ý dòng 15 cũng có nhiều khoảng trắng ở cuối để ghi đè các ký tự
còn sót lại từ đoạn đếm ngược.

Cuối cùng, có một dòng `fflush(stdout)` lạ lạ, mà không hiểu nghĩa
là gì. Ngắn gọn là phần lớn terminal mặc định _line buffered_, nghĩa
là chúng không thật sự in ra gì cho tới khi gặp ký tự newline. Vì ta
không có newline (chỉ có `\r`), nếu không có dòng đó, chương trình
sẽ ngồi im cho tới lúc `Liftoff!` rồi in tất cả trong một nháy.
`fflush()` ghi đè hành vi này và ép output diễn ra _ngay bây giờ_.

[i[`\r` carriage return]>]

#### Escape cho dấu hỏi

[i[`\?` question mark]<]

Sao phải bận tâm chuyện này? Cái này chạy tốt mà:

``` {.c}
printf("Doesn't it?\n");
```

Và dùng escape cũng chạy tốt:

``` {.c}
printf("Doesn't it\?\n");   // Note \?
```

Vậy thì để làm gì??!

[i[Trigraphs]<]

Ta nhấn mạnh hơn chút với thêm một dấu hỏi và một dấu chấm than:

``` {.c}
printf("Doesn't it??!\n");
```

Khi tôi compile cái này, tôi nhận được cảnh báo:

``` {.zsh}
foo.c: In function ‘main’:
foo.c:5:23: warning: trigraph ??! converted to | [-Wtrigraphs]
    5 |     printf("Doesn't it??!\n");
      |    
```

Và chạy nó thì cho kết quả khó tin:

``` {.default}
Doesn't it|
```

Vậy _trigraph_? Cái quái gì đây??!

Tôi chắc ta sẽ quay lại cái góc bụi bặm này của ngôn ngữ sau, nhưng
ngắn gọn là compiler tìm một số bộ ba ký tự nhất định bắt đầu bằng
`??` rồi thay chúng bằng ký tự khác. Vậy nếu bạn đang ngồi trước một
terminal cổ lỗ sĩ không có ký hiệu pipe (`|`) trên bàn phím, bạn có
thể gõ `??!` thay thế.

Bạn có thể sửa bằng cách escape dấu hỏi thứ hai, kiểu vầy:

``` {.c}
printf("Doesn't it?\?!\n");
```

Và rồi nó compile và chạy như mong đợi.

Tất nhiên, ngày nay chẳng ai dùng trigraph nữa. Nhưng cái `??!` đó
đôi khi vẫn xuất hiện nếu bạn quyết định dùng nó trong một chuỗi để
nhấn mạnh.

[i[Trigraphs]>]
[i[`\?` question mark]>]

### Escape dạng số

Ngoài ra, còn có các cách để chỉ định hằng số hay giá trị ký tự khác
bên trong chuỗi hay hằng ký tự.

Nếu bạn biết biểu diễn octal hay hexadecimal của một byte, bạn có
thể đưa nó vào một chuỗi hay hằng ký tự.

Bảng dưới có các con số ví dụ, nhưng bất kỳ số hex hay octal nào
cũng dùng được. Pad thêm số 0 đầu nếu cần để đủ số chữ số.

[i[`\123` octal value]]
[i[`\x12` hexadecimal value]]
[i[`\u` Unicode escape]]
[i[`\U` Unicode escape]]

|Code|Mô tả|
|--|------------|
|`\123`|Nhúng byte có giá trị octal `123`, đúng 3 chữ số.|
|`\x4D`|Nhúng byte có giá trị hex `4D`, 2 chữ số.|
|`\u2620`|Nhúng ký tự Unicode tại code point có giá trị hex `2620`, 4 chữ số.|
|`\U0001243F`|Nhúng ký tự Unicode tại code point có giá trị hex `1243F`, 8 chữ số.|

Đây là ví dụ dùng ký pháp octal ít gặp để biểu diễn chữ `B` nằm giữa
`A` và `C`. Thường cách này được dùng cho ký tự đặc biệt không in
được, nhưng ta có cách khác để làm thế bên dưới, đây chỉ là demo
octal thôi:

[i[`\123` octal value]<]

``` {.c}
printf("A\102C\n");  // 102 is `B` in ASCII/UTF-8
```

Chú ý không có số 0 đầu ở số octal khi bạn viết theo kiểu này. Nhưng
nó cần đúng ba ký tự, nên hãy pad thêm số 0 đầu nếu cần.

[i[`\123` octal value]>]

[i[`\x12` hexadecimal value]<]

Nhưng phổ biến hơn nhiều ngày nay là dùng hằng hex. Đây là một demo
mà bạn không nên dùng, nhưng nó minh họa việc nhúng các byte UTF-8
0xE2, 0x80, và 0xA2 vào trong một chuỗi, ứng với ký tự Unicode
"bullet" (•).

``` {.c}
printf("\xE2\x80\xA2 Bullet 1\n");
printf("\xE2\x80\xA2 Bullet 2\n");
printf("\xE2\x80\xA2 Bullet 3\n");
```

Sinh ra output sau nếu bạn đang ở console UTF-8 (hoặc có khi là rác
nếu không):

``` {.default}
• Bullet 1
• Bullet 2
• Bullet 3
```

[i[`\x12` hexadecimal value]>]


[i[`\u` Unicode escape]<]
[i[`\U` Unicode escape]<]

Nhưng đó là cách lởm khởm để làm Unicode. Bạn có thể dùng escape
`\u` (16-bit) hoặc `\U` (32-bit) để tham chiếu Unicode bằng số code
point thẳng luôn. Bullet là `2022` (hex) trong Unicode, nên bạn có
thể làm vầy để có kết quả portable hơn:

``` {.c}
printf("\u2022 Bullet 1\n");
printf("\u2022 Bullet 2\n");
printf("\u2022 Bullet 3\n");
```

Nhớ pad `\u` đủ số 0 đầu cho đủ bốn ký tự, và `\U` đủ để ra tám.

[i[`\u` Unicode escape]>]

Ví dụ, cái bullet đó có thể làm bằng `\U` với bốn số 0 đầu:

``` {.c}
printf("\U00002022 Bullet 1\n");
```

[i[`\U` Unicode escape]>]

Ai rảnh mà dài dòng thế?

[i[`\` backslash escape]>]
[i[Escape sequences]>]

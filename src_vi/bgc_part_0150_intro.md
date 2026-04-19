<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Hello, World!

## Kỳ vọng gì từ C

> _"Mấy cái cầu thang này dẫn đi đâu?"_ \
> _"Nó dẫn đi lên."_
>
> ---Ray Stantz và Peter Venkman, Ghostbusters

C là một ngôn ngữ cấp thấp.

Nó đâu từng như vậy. Thời xa xưa khi người ta còn đục thẻ bìa đục lỗ từ đá hoa cương, C là một cách tuyệt vời để thoát khỏi cực hình của các ngôn ngữ cấp thấp hơn như [flw[assembly|Assembly_language]].

Nhưng ở thời hiện đại này, các ngôn ngữ thế hệ mới cung cấp đủ thứ tính năng không tồn tại vào năm 1972 khi C được phát minh. Điều đó có nghĩa là C là một ngôn ngữ khá cơ bản với không nhiều tính năng. Nó có thể làm _mọi thứ_, nhưng sẽ bắt bạn đổ mồ hôi cho chúng.

Vậy tại sao ta vẫn còn dùng C đến bây giờ?

* Như một công cụ học tập: C không chỉ là một mảnh lịch sử đáng kính của ngành máy tính, mà còn kết nối với [flw[phần cứng thô|Bare_machine]] (bare metal) theo cách mà các ngôn ngữ hiện thời không có. Khi học C, bạn học về việc phần mềm tương tác với bộ nhớ máy tính ở cấp độ thấp như thế nào. Không có dây an toàn. Bạn sẽ viết ra các phần mềm bị crash, tôi bảo đảm với bạn. Và đó là một phần của cuộc vui!

* Như một công cụ hữu ích: C vẫn còn được dùng cho một số ứng dụng nhất định, chẳng hạn như xây dựng [flw[hệ điều hành|Operating_system]] hay trong [flw[hệ thống nhúng|Embedded_system]]. (Mặc dù ngôn ngữ [flw[Rust|Rust_(programming_language)]] đang dòm ngó cả hai lĩnh vực đó!)

Nếu bạn đã quen với một ngôn ngữ khác, nhiều thứ trong C sẽ dễ. C đã truyền cảm hứng cho rất nhiều ngôn ngữ khác, và bạn sẽ thấy mảng mảng của nó trong Go, Rust, Swift, Python, JavaScript, Java, và đủ loại ngôn ngữ khác. Những phần đó sẽ quen thuộc.

Thứ duy nhất trong C làm người ta khựng lại là _con trỏ_ (pointers). Gần như mọi thứ khác đều quen thuộc, nhưng con trỏ là đứa con lạ. Khái niệm đằng sau con trỏ có lẽ bạn đã biết rồi, nhưng C buộc bạn phải tường minh về nó, bằng các toán tử mà có thể bạn chưa từng thấy bao giờ.

Điều đặc biệt khó chịu là một khi bạn đã [flw[_nắm được_|Grok]] con trỏ, nó bỗng nhiên trở nên dễ. Còn trước thời điểm đó, chúng cứ trơn tuột như lươn.

Mọi thứ khác trong C chỉ đơn giản là ghi nhớ một cách khác (đôi khi chính là _cùng_ một cách!) để làm một việc bạn đã làm rồi. Con trỏ là phần lạ lẫm. Và, nếu xét kỹ, ngay cả con trỏ cũng chỉ là biến tấu trên một chủ đề có lẽ bạn đã quen.

Vậy chuẩn bị tinh thần cho một chuyến phiêu lưu náo nhiệt gần nhất với lõi của máy tính mà bạn có thể đến được mà không cần đụng đến assembly, bằng ngôn ngữ có ảnh hưởng nhất mọi thời đại^[Tôi biết sẽ có người cãi tôi về điểm này, nhưng chắc ít nhất cũng phải nằm trong top ba, đúng không?]. Giữ chặt!


## Hello, World!

[i[Hello, world]()]Đây là ví dụ chuẩn mực của một chương trình C. Ai cũng dùng nó. (Lưu ý là các con số ở bên trái chỉ để người đọc tham khảo, chúng không phải là một phần của mã nguồn.)

``` {.c .numberLines}
/* Hello world program */

#include <stdio.h>

int main(void)
{
    printf("Hello, World!\n");  // Actually do the work here
}
```

Chúng ta sẽ đeo găng tay cao su loại dày tay áo dài, vớ lấy con dao mổ, và rạch thẳng vào thứ này để xem cái gì làm nó hoạt động. Nào, rửa tay đi, vì ta bắt đầu. Cắt nhẹ thôi...

[i[Comments]<]

Ta giải quyết cái dễ trước: mọi thứ nằm giữa hai cặp ký hiệu `/*` và `*/` là chú thích (comment) và sẽ bị trình biên dịch bỏ qua hoàn toàn. Mọi thứ nằm trên một dòng sau `//` cũng vậy. Nó cho phép bạn để lại thông điệp cho chính mình và cho người khác, để khi bạn quay lại đọc code của mình ở tương lai xa, bạn biết cái quái gì mình đang định làm. Tin tôi đi, bạn sẽ quên; chuyện đó xảy ra.

[i[Comments]>]

[i[C Preprocessor]<]
[i[`#include` directive]<]

Giờ, cái `#include` này là gì? KINH QUÁ! Được rồi, nó báo cho C Preprocessor kéo nội dung của một file khác và chèn vào code ngay _chỗ đó_.

Khoan, C Preprocessor là cái gì? Câu hỏi hay. Việc biên dịch có hai giai đoạn^[Ờ thì, về mặt kỹ thuật thì nhiều hơn hai, nhưng thôi, ta cứ giả vờ là có hai, càng ít biết càng vui, nhỉ?]: preprocessor và compiler. [i[Octothorpe]]Bất cứ thứ gì bắt đầu bằng dấu pound, dấu thăng, hay "octothorpe", (`#`) là thứ mà preprocessor[i[Preprocessor]] xử lý trước khi compiler thậm chí còn bắt đầu. Các _chỉ thị preprocessor_ (preprocessor directives) thường gặp, như người ta hay gọi, là `#include` và `#define`.[i[`#define` directive]] Bàn thêm về mấy cái đó sau.[i[`#include` directive]>][i[C Preprocessor]>]

Trước khi đi tiếp, tại sao tôi lại dày công chỉ ra rằng dấu pound được gọi là octothorpe? Câu trả lời đơn giản: tôi thấy từ octothorpe nó buồn cười xuất sắc, nên phải rải bừa cái tên đó ra mọi khi có dịp. Octothorpe. Octothorpe, octothorpe, octothorpe.

Nên _dù sao đi nữa_. Sau khi C preprocessor xử lý xong mọi thứ, kết quả được trao cho compiler để nó sản xuất ra [flw[mã assembly|Assembly_language]], [flw[mã máy|Machine_code]], hay bất cứ cái gì nó định làm. Mã máy là "ngôn ngữ" mà CPU hiểu, và nó có thể hiểu _rất nhanh_. Đây là một trong những lý do chương trình C thường chạy nhanh.

Đừng lo về các chi tiết kỹ thuật của quá trình biên dịch lúc này; cứ biết rằng code của bạn chạy qua preprocessor, rồi output của nó chạy qua compiler, rồi cái đó tạo ra một file thực thi để bạn chạy.

Còn phần còn lại của dòng thì sao? [i[`stdio.h` header file]<]Cái `<stdio.h>` là gì? Đó là thứ người ta gọi là _header file_. Chính cái chấm-h ở cuối đã tiết lộ điều đó. Thực ra nó là header file "Standard I/O" (`stdio`) mà bạn sẽ dần dần quen và yêu mến. Nó cho ta quyền truy cập vào một loạt chức năng I/O^[Về mặt kỹ thuật, nó chứa các chỉ thị preprocessor và nguyên mẫu hàm (function prototypes, bàn thêm sau) cho các nhu cầu input/output thông dụng.]. Với chương trình demo của ta, ta đang xuất chuỗi "Hello, World!", nên cụ thể là ta cần truy cập đến hàm [i[`printf()` function]<] `printf()` để làm việc đó. File `<stdio.h>` cho ta quyền truy cập đó. Nói cơ bản, nếu ta cố dùng `printf()` mà không có `#include <stdio.h>`, compiler sẽ rền rĩ phàn nàn với ta về chuyện đó.

Sao tôi biết phải `#include <stdio.h>` cho `printf()`?[i[`printf()` function]>] Câu trả lời: nó nằm trong tài liệu. Nếu bạn đang trên hệ Unix, gõ `man 3 printf` và nó sẽ cho bạn biết ngay ở đầu trang man cần những file header nào. Hoặc xem phần tham khảo trong cuốn sách này. `:-)` [i[`stdio.h` header file]>]

Trời đất ơi. Ngần ấy chỉ để cover dòng đầu tiên! Nhưng, nói thẳng ra, nó đã bị mổ xẻ hoàn toàn. Không còn bí ẩn nào sót lại!

Vậy thở một hơi đi... nhìn lại mã mẫu. Chỉ còn vài dòng dễ nữa thôi.

Chào mừng quay lại sau kỳ nghỉ! Tôi biết bạn chẳng nghỉ thực sự đâu; tôi chỉ chiều lòng bạn thôi.

[i[`main()` function]<]Dòng tiếp theo là `main()`. Đây là định nghĩa của hàm `main()`; mọi thứ giữa cặp dấu ngoặc nhọn ngoằn ngoèo (`{` và `}`) là một phần của định nghĩa hàm.

(Vậy thì _gọi_ một hàm khác như thế nào nhỉ? Câu trả lời nằm ở dòng `printf()`, ta sẽ đến đó trong một phút nữa.)

Giờ, hàm main là đặc biệt theo nhiều nghĩa, nhưng có một nghĩa nổi bật hơn cả: nó là hàm sẽ được gọi tự động khi chương trình của bạn bắt đầu chạy. Không có gì của bạn được gọi trước `main()`. Trong ví dụ của ta, điều này ổn vì tất cả những gì ta muốn làm là in một dòng rồi thoát.

À, còn chuyện này: một khi chương trình chạy qua khỏi cuối `main()`, chỗ dấu ngoặc nhọn đóng ở dưới đó, chương trình sẽ thoát, và bạn sẽ trở lại với dấu nhắc dòng lệnh.

Vậy giờ ta biết rằng chương trình đó đã kéo vào một header file, `stdio.h`[i[stdio.h]T], và khai báo một hàm `main()` sẽ chạy khi chương trình được khởi động. Bên trong `main()`[i[`main()` function]>] có những món ngon gì?

Tôi rất vui là bạn đã hỏi. Thật đấy! Ta chỉ có đúng một món ngon thôi: lời gọi đến hàm [i[`printf()` function]<]`printf()`. Bạn có thể nhận ra đây là một lời gọi hàm chứ không phải định nghĩa hàm qua nhiều cách, nhưng một dấu hiệu là không có cặp dấu ngoặc nhọn ngoằn ngoèo đi sau nó. Và bạn kết thúc lời gọi hàm bằng một dấu chấm phẩy để compiler biết đây là điểm kết của biểu thức. Bạn sẽ đặt dấu chấm phẩy sau gần như mọi thứ, bạn sẽ thấy.

Bạn đang truyền một đối số (argument) cho hàm `printf()`[i[`printf()` function]>]: một chuỗi sẽ được in ra khi bạn gọi nó. À đúng rồi, ta đang _gọi_ một hàm! Ta giỏi thế! Khoan, khoan, đừng vội tự mãn. [i[`\n` newline]<]Cái `\n` khùng khùng ở cuối chuỗi là gì? Ờ, phần lớn các ký tự trong chuỗi sẽ được in ra đúng như cách chúng được lưu. Nhưng có một số ký tự không thể in trên màn hình một cách đẹp đẽ nên được nhúng dưới dạng mã hai ký tự bắt đầu bằng dấu chéo ngược. Một trong những cái phổ biến nhất là `\n` (đọc là "backslash-N" hoặc đơn giản "newline") tương ứng với ký tự _xuống dòng_. Đây là ký tự làm cho việc in tiếp theo bắt đầu ở đầu dòng tiếp chứ không ở dòng hiện tại. Giống như bạn nhấn return ở cuối dòng.[i[`\n` newline]>]

Vậy chép đoạn code đó vào một file tên là `hello.c` và build nó. Trên nền tảng kiểu Unix (ví dụ Linux, BSD, Mac, hay WSL), từ dòng lệnh bạn sẽ build bằng lệnh kiểu thế này:

[i[`gcc` compiler]]
``` {.zsh}
gcc -o hello hello.c
```

(Có nghĩa là "biên dịch `hello.c`, và xuất ra file thực thi tên là `hello`".)

Sau khi xong, bạn sẽ có một file tên là `hello` mà bạn có thể chạy bằng lệnh này:

``` {.default}
./hello
```

(Phần `./` ở đầu bảo shell "chạy file từ thư mục hiện tại".)

Và xem thử nó ra cái gì:

``` {.default}
Hello, World! 
```

Xong và đã test! Ship it![i[Hello, world]>]

## Chi tiết về biên dịch

[i[Compilation]<]Nói thêm một chút về cách build chương trình C, và chuyện gì xảy ra hậu trường.

Giống các ngôn ngữ khác, C có _mã nguồn_ (source code). Nhưng, tùy vào ngôn ngữ bạn đến từ đâu, có thể bạn chưa bao giờ phải _biên dịch_ mã nguồn của mình thành một _file thực thi_ (executable).

Biên dịch là quá trình lấy mã nguồn C của bạn và biến nó thành một chương trình mà hệ điều hành có thể thực thi.

Dân JavaScript và Python không hề quen với một bước biên dịch tách biệt, dù rằng hậu trường nó vẫn đang diễn ra! Python biên dịch mã nguồn của bạn thành thứ gọi là _bytecode_ mà máy ảo Python có thể chạy. Dân Java thì quen với việc biên dịch, nhưng cái đó sinh ra bytecode cho Java Virtual Machine.

Khi biên dịch C, _mã máy_ được sinh ra. Đây là các số 1 và 0 mà CPU có thể chạy trực tiếp và nhanh chóng.

> Các ngôn ngữ thường không biên dịch được gọi là ngôn ngữ _thông dịch_ (interpreted). Nhưng như ta đã nói với Java và Python, chúng cũng có một bước biên dịch. Và không có luật nào nói C không thể được thông dịch. (Ngoài kia có cả interpreter cho C đấy!) Nói ngắn gọn, nó là một mớ ranh giới mờ. Biên dịch nói chung chỉ là việc lấy mã nguồn và biến nó thành một dạng khác, dễ thực thi hơn.

Trình biên dịch C (C compiler) là chương trình làm việc biên dịch đó.

Như đã nói, `gcc` là một trình biên dịch được cài sẵn trên rất nhiều [flw[hệ điều hành kiểu Unix|Unix]]. Và thường được chạy từ dòng lệnh trong terminal, nhưng không phải luôn luôn. Bạn cũng có thể chạy nó từ IDE.

Vậy ta build từ dòng lệnh kiểu gì?

## Build với `gcc`

[i[`gcc` compiler]<]Nếu bạn có một file nguồn tên là `hello.c` trong thư mục hiện tại, bạn có thể build nó thành một chương trình tên là `hello` bằng lệnh gõ trong terminal sau:

``` {.zsh}
gcc -o hello hello.c
```

Cờ `-o` có nghĩa là "xuất ra file này"^[Nếu bạn không chỉ định tên file xuất, nó sẽ xuất ra một file tên là `a.out` theo mặc định, cái tên này có gốc rễ sâu trong lịch sử Unix.]. Và ở cuối là `hello.c`, tên của file ta muốn biên dịch.

Nếu mã nguồn được tách làm nhiều file, bạn có thể biên dịch tất cả cùng nhau (gần như thể chúng là một file, dù các quy tắc thực sự có phần phức tạp hơn) bằng cách đưa tất cả các file `.c` lên dòng lệnh:

``` {.zsh}
gcc -o awesomegame ui.c characters.c npc.c items.c
```
[i[`gcc` compiler]>]

và tất cả chúng sẽ được build cùng nhau thành một file thực thi to.

Thế là đủ để bắt đầu, sau này ta sẽ bàn chi tiết về nhiều file nguồn, object files, và đủ thứ vui khác.[i[Compilation]>]

## Build với `clang`

Trên máy Mac, trình biên dịch mặc định không phải `gcc`, mà là `clang`[i[`clang` compiler]]. Nhưng cũng có một wrapper được cài sẵn để bạn vẫn chạy `gcc` được.

Bạn cũng có thể cài đúng trình `gcc`[i[`gcc` compiler]] qua [fl[Homebrew|https://formulae.brew.sh/formula/gcc]] hay cách khác.

## Build từ IDE

[i[Integrated Development Environment]<]Nếu bạn đang dùng _Môi trường phát triển tích hợp_ (Integrated Development Environment, IDE), có lẽ bạn không phải build từ dòng lệnh. 

Với Visual Studio, `CTRL-F7` sẽ build, và `CTRL-F5` sẽ chạy.

Với VS Code, bạn có thể nhấn `F5` để chạy qua debugger. (Bạn sẽ phải cài C/C++ Extension.)

Với XCode, bạn có thể build bằng `COMMAND-B` và chạy bằng `COMMAND-R`. Để có bộ command line tools, Google "XCode command line tools" và bạn sẽ tìm được hướng dẫn cài đặt.

Để bắt đầu, tôi khuyến khích bạn cũng thử build từ dòng lệnh, nó là lịch sử mà![i[Integrated Development Environment]>]

## Các phiên bản C

[i[Language versions]<]C đã đi một chặng đường dài qua nhiều năm, và nó có nhiều số hiệu phiên bản được đặt tên để chỉ ra phương ngữ của ngôn ngữ mà bạn đang dùng.

Chúng thường tham chiếu đến năm của bản đặc tả.

Nổi tiếng nhất là C89, C99, C11, và C23. Ta sẽ tập trung vào cái cuối cùng trong cuốn sách này.

Nhưng đây là một bảng đầy đủ hơn:

|Phiên bản|Mô tả|
|-----|--------------|
|K&R C|1978, bản gốc. Đặt tên theo Brian Kernighan và Dennis Ritchie. Ritchie thiết kế và viết ngôn ngữ, còn Kernighan đồng tác giả cuốn sách về nó. Ngày nay ít khi thấy code K&R gốc. Nếu có thấy, nó sẽ trông lạ, giống như tiếng Anh Trung cổ trông lạ với người đọc tiếng Anh hiện đại.|
|**C89**, ANSI C, C90|Năm 1989, Viện Tiêu chuẩn Quốc gia Hoa Kỳ (ANSI) cho ra một bản đặc tả ngôn ngữ C đặt nền tảng cho C kéo dài đến tận hôm nay. Một năm sau, dây cương được trao cho Tổ chức Tiêu chuẩn hóa Quốc tế (ISO), cho ra C90 giống hệt.|
|C95|Một bản bổ sung ít được nhắc tới cho C89 có thêm hỗ trợ ký tự rộng.|
|**C99**|Đợt đại tu lớn đầu tiên với rất nhiều bổ sung về ngôn ngữ. Thứ mà hầu như ai cũng nhớ là thêm kiểu chú thích `//`. Đây là phiên bản C phổ biến nhất còn được dùng tính đến thời điểm viết cuốn sách này.|
|**C11**|Bản cập nhật lớn này gồm hỗ trợ Unicode và đa luồng. Lưu ý rằng nếu bạn bắt đầu dùng các tính năng ngôn ngữ này, có thể bạn đang đánh đổi tính dễ chuyển với những nơi còn mắc kẹt ở C99. Nhưng, nói thật, 1999 cũng đã khá lâu rồi.|
|C17, C18|Bản cập nhật sửa lỗi cho C11. C17 có vẻ là tên chính thức, nhưng việc xuất bản bị hoãn đến 2018. Theo tôi thấy, hai tên này có thể thay nhau, C17 được ưa chuộng hơn.|
|C23|Bản đặc tả mới nhất.||

[i[`gcc` compiler]<]Bạn có thể ép GCC dùng một trong các chuẩn này bằng tham số dòng lệnh `-std=`. Nếu muốn nó soi kỹ chuẩn, thêm `-pedantic`.

Ví dụ:

``` {.zsh}
gcc -std=c11 -pedantic foo.c
```

Với cuốn sách này, tôi biên dịch chương trình cho C23 với toàn bộ cảnh báo bật lên:

``` {.zsh}
gcc -Wall -Wextra -std=c23 -pedantic foo.c
```
[i[`gcc` compiler]>]

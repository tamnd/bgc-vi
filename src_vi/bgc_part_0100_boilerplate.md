# Lời nói đầu
<!-- Beej's guide to C
# vim: ts=4:sw=4:nosi:et:tw=72
-->

<!-- No hyphenation -->
[nh[scalbn]]
[nh[scalbnf]]
[nh[scalbnl]]
[nh[scalbln]]
[nh[scalblnf]]
[nh[scalblnl]]
<!-- Can't do things that aren't letters
[nh[atan2]]
[nh[atan2f]]
[nh[atan2l]]
-->
[nh[lrint]]
[nh[lrintf]]
[nh[lrintl]]
[nh[llrint]]
[nh[llrintf]]
[nh[llrintl]]
[nh[lround]]
[nh[lroundf]]
[nh[lroundl]]
[nh[llround]]
[nh[llroundf]]
[nh[llroundl]]

<!-- Index see alsos -->
[is[String==>see `char *`]]
[is[New line==>see `\n` newline]]
[is[Ternary operator==>see `?:` ternary operator]]
[is[Addition operator==>see `+` addition operator]]
[is[Subtraction operator==>see `-` subtraction operator]]
[is[Multiplication operator==>see `*` multiplication operator]]
[is[Division operator==>see `/` division operator]]
[is[Modulus operator==>see `%` modulus operator]]
[is[Boolean NOT==>see `!` operator]]
[is[Boolean AND==>see `&&` operator]]
[is[Boolean OR==>see `||` operator]]
[is[Bell==>see `\a` operator]]
[is[Tab (is better)==>see `\t` operator]]
[is[Carriage return==>see `\r` operator]]
[is[Hexadecimal==>see `0x` hexadecimal]]

> _C không phải là một ngôn ngữ lớn, và nó không hợp với một cuốn sách lớn._
>
> --Brian W. Kernighan, Dennis M. Ritchie

Không có lý do gì để phí lời ở đây nữa, các bạn, ta nhảy thẳng vào code C luôn:

``` {.c}
E((ck?main((z?(stat(M,&t)?P+=a+'{'?0:3:
execv(M,k),a=G,i=P,y=G&255,
sprintf(Q,y/'@'-3?A(*L(V(%d+%d)+%d,0)
```

Và họ sống hạnh phúc mãi mãi về sau. Hết.

Hử? Bạn bảo vẫn còn điều gì đó chưa rõ về cái ngôn ngữ lập trình C này?

Ừ thì, nói thật, chính tôi cũng không biết đoạn code trên làm gì. Nó là một mẩu trích từ một bài dự thi năm 2001 của [fl[International Obfuscated C Code Contest|https://www.ioccc.org/]][i[International Obfuscated C Code Contest]], một cuộc thi tuyệt vời mà người dự thi cố viết code C khó đọc nhất có thể, thường cho ra kết quả gây ngạc nhiên.

Tin xấu là nếu bạn mới bắt đầu với thứ này, mọi đoạn code C bạn thấy trông có lẽ đều giống như bị làm rối tung lên! Tin tốt là, cảm giác đó sẽ không kéo dài lâu đâu.

Điều tôi sẽ cố làm trong suốt hướng dẫn này là dẫn bạn từ trạng thái hoang mang toàn tập đến kiểu hạnh phúc tỉnh ngộ chỉ có thể đạt được qua lập trình C thuần túy. Cứ thế nhé.

Ngày xưa, C là một ngôn ngữ đơn giản hơn. Rất nhiều tính năng trong cuốn sách này cùng _một đống_ tính năng trong tập Library Reference còn chưa tồn tại khi K&R viết ấn bản thứ hai nổi tiếng vào năm 1988. Dù vậy, phần lõi của ngôn ngữ vẫn nhỏ, và tôi hy vọng mình đã trình bày ở đây theo cách bắt đầu từ cái lõi đơn giản đó rồi mở rộng dần ra.

Và đó là lý do tôi bào chữa cho việc viết một cuốn sách to đến buồn cười về một ngôn ngữ nhỏ gọn và cô đọng như vậy.

## Đối tượng đọc

Hướng dẫn này giả định rằng bạn đã có sẵn một chút kiến thức lập trình từ một ngôn ngữ khác, kiểu như [flw[Python|Python_(programming_language)]], [flw[JavaScript|JavaScript]], [flw[Java|Java_(programming_language)]], [flw[Rust|Rust_(programming_language)]], [flw[Go|Go_(programming_language)]], [flw[Swift|Swift_(programming_language)]], v.v. (Dân [flw[Objective-C|Objective-C]] sẽ cực kỳ dễ thở!)

Chúng ta sẽ giả định là bạn biết biến là gì, vòng lặp làm gì, hàm hoạt động ra sao, và đại loại thế.

Nếu điều đó không đúng với bạn vì lý do nào đi nữa, thì điều tốt nhất tôi có thể hy vọng cung cấp là một chút giải trí chân thành cho niềm vui đọc sách của bạn. Điều duy nhất tôi có thể hứa một cách hợp lý là hướng dẫn này sẽ không kết thúc ở một nút thắt hồi hộp... hay _là_ sẽ kết thúc như thế?

## Cách đọc cuốn sách này

Hướng dẫn chia làm hai tập, và đây là tập đầu: tập hướng dẫn!

Tập thứ hai là [fl[library reference|https://beej.us/guide/bgclr/]], và nó mang tính tham khảo hơn là hướng dẫn nhiều.

Nếu bạn là người mới, hãy đi qua phần hướng dẫn theo thứ tự, nói chung là vậy. Càng lên cao trong các chương thì thứ tự càng bớt quan trọng.

Và dù trình độ của bạn đến đâu, phần tham khảo luôn sẵn ở đó với các ví dụ đầy đủ về những hàm trong thư viện chuẩn, giúp bạn làm mới trí nhớ bất cứ khi nào cần. Hợp để đọc khi đang ăn một tô ngũ cốc hoặc trong những lúc rảnh khác.

Cuối cùng, liếc qua phần mục lục (nếu bạn đang đọc bản in), các mục thuộc phần tham khảo được in nghiêng.

## Nền tảng và trình biên dịch

Tôi sẽ cố bám vào [flw[C chuẩn ISO kiểu cũ|ANSI_C]]. Ờ, phần lớn thôi. Đôi khi tôi có thể nổi hứng mà nói về [flw[POSIX|POSIX]] hay gì đó, nhưng để xem đã.

Người dùng **Unix** (ví dụ Linux, BSD, v.v.) thử chạy `cc` hoặc `gcc` từ dòng lệnh, biết đâu bạn đã có sẵn một trình biên dịch cài rồi. Nếu chưa, tìm trong bản phân phối của bạn cách cài `gcc` hoặc `clang`.

Người dùng **Windows** nên xem qua [fl[Visual Studio Community|https://visualstudio.microsoft.com/vs/community/]]. Hoặc, nếu bạn muốn trải nghiệm kiểu Unix hơn (rất khuyến khích!), cài [fl[WSL|https://docs.microsoft.com/en-us/windows/wsl/install-win10]] và `gcc`.

Người dùng **Mac** sẽ muốn cài [fl[XCode|https://developer.apple.com/xcode/]], và đặc biệt là bộ command line tools.

Có cả tá trình biên dịch ngoài kia, và hầu như tất cả đều dùng được cho cuốn sách này. Một trình biên dịch C++ cũng sẽ biên dịch được phần lớn (nhưng không phải tất cả!) code C. Tốt nhất là dùng một trình biên dịch C đúng nghĩa nếu được.

## Trang chủ chính thức

Vị trí chính thức của tài liệu này là [fl[https://beej.us/guide/bgc/|https://beej.us/guide/bgc/]]. Có thể điều này sẽ thay đổi trong tương lai, nhưng khả năng cao hơn là mọi hướng dẫn khác sẽ được dời khỏi máy tính ở Chico State.

## Chính sách email

Tôi thường có mặt để giúp trả lời các câu hỏi qua email, nên cứ viết cho tôi, nhưng tôi không thể bảo đảm sẽ trả lời. Tôi có một cuộc sống khá bận rộn và có những lúc đơn giản là không thể trả lời câu hỏi của bạn. Khi đó, thường là tôi xóa tin nhắn đi luôn. Không có gì cá nhân cả; chỉ là tôi sẽ không bao giờ có đủ thời gian để đưa ra câu trả lời chi tiết mà bạn cần.

Theo nguyên tắc chung, câu hỏi càng phức tạp thì khả năng tôi trả lời càng thấp. Nếu bạn thu hẹp được câu hỏi trước khi gửi và nhớ đính kèm mọi thông tin liên quan (như nền tảng, trình biên dịch, thông báo lỗi bạn đang nhận được, và bất cứ thứ gì bạn nghĩ có thể giúp tôi tìm ra vấn đề), khả năng có hồi âm sẽ cao hơn nhiều.

Nếu bạn không nhận được hồi âm, cứ tiếp tục mò mẫm, cố tự tìm ra câu trả lời, và nếu vẫn không ra, viết lại cho tôi với thông tin đã tìm được, hy vọng khi đó sẽ đủ để tôi giúp đỡ.

Giờ mà tôi đã cằn nhằn xong về chuyện viết hay không viết email cho tôi, chỉ xin nói thêm rằng tôi _thực sự_ trân trọng mọi lời khen mà cuốn hướng dẫn này đã nhận được suốt những năm qua. Nó là một liều tinh thần thật sự, và tôi vui khi biết nó đang được dùng vào việc tốt! `:-)` Cảm ơn bạn!

## Sao lưu (mirror)

Bạn hoàn toàn được hoan nghênh sao lưu trang này, dù là công khai hay riêng tư. Nếu bạn mirror công khai và muốn tôi liên kết tới bản của bạn từ trang chính, cứ gửi cho tôi một dòng ở [`beej@beej.us`](mailto:beej@beej.us).

## Ghi chú cho người dịch

Nếu bạn muốn dịch hướng dẫn này sang một ngôn ngữ khác, hãy viết cho tôi tại [`beej@beej.us`](beej@beej.us) và tôi sẽ liên kết tới bản dịch của bạn từ trang chính. Cứ thoải mái thêm tên và thông tin liên hệ của bạn vào bản dịch.

Xin lưu ý các điều khoản giấy phép ở mục Bản quyền và Phân phối bên dưới.

## Bản quyền và Phân phối

Beej's Guide to C có Bản quyền © 2021 Brian "Beej Jorgensen" Hall.

Ngoại trừ một vài trường hợp cụ thể dành cho mã nguồn và bản dịch, nêu ở dưới, tác phẩm này được cấp phép theo giấy phép Creative Commons Attribution-Noncommercial-No Derivative Works 3.0. Để xem một bản của giấy phép này, ghé [`https://creativecommons.org/licenses/by-nc-nd/3.0/`](https://creativecommons.org/licenses/by-nc-nd/3.0/) hoặc gửi thư tới Creative Commons, 171 Second Street, Suite 300, San Francisco, California, 94105, USA.

Một ngoại lệ cụ thể cho phần "No Derivative Works" của giấy phép như sau: hướng dẫn này có thể được tự do dịch sang bất kỳ ngôn ngữ nào, miễn là bản dịch chính xác, và hướng dẫn được in lại đầy đủ. Các giới hạn giấy phép áp dụng cho bản dịch cũng giống như áp dụng cho bản gốc. Bản dịch cũng có thể kèm theo tên và thông tin liên hệ của người dịch.

Mã nguồn C trình bày trong tài liệu này được trao cho miền công cộng, hoàn toàn không có bất kỳ giới hạn giấy phép nào.

Các nhà giáo dục được khuyến khích giới thiệu hoặc cung cấp các bản của hướng dẫn này cho học viên của mình.

Liên hệ [`beej@beej.us`](beej@beej.us) để biết thêm thông tin.

## Lời tri ân

Những điều khó nhất khi viết các hướng dẫn này là:

* Học tài liệu đủ kỹ để có thể giảng lại
* Tìm ra cách giải thích rõ ràng nhất, một quá trình lặp đi lặp lại tưởng như không có hồi kết
* Tự đặt mình vào vai kẻ được gọi là _người có thẩm quyền_, trong khi thật ra tôi chỉ là một người bình thường đang cố hiểu mọi thứ, giống như mọi người khác thôi
* Kiên trì khi có biết bao thứ khác kéo sự chú ý của tôi đi chỗ khác

Rất nhiều người đã giúp tôi đi qua quá trình này, và tôi muốn ghi nhận những người đã khiến cuốn sách này thành sự thật.

* Mọi người trên Internet đã quyết định chia sẻ kiến thức của mình dưới hình thức này hay hình thức khác. Chính việc tự do chia sẻ các thông tin mang tính hướng dẫn đã khiến Internet trở thành nơi tuyệt vời như hiện nay.
* Những tình nguyện viên ở [fl[cppreference.com|https://en.cppreference.com/]], những người bắc chiếc cầu nối từ bản spec sang thế giới thực.
* Các cao nhân thân thiện trên [fl[comp.lang.c|https://groups.google.com/g/comp.lang.c]] và [fl[r/C_Programming|https://www.reddit.com/r/C_Programming/]], những người đã kéo tôi qua những phần khó nhằn của ngôn ngữ.
* Tất cả những ai đã gửi sửa lỗi và pull request cho mọi thứ, từ hướng dẫn gây hiểu lầm cho đến lỗi chính tả.

Cảm ơn bạn! ♥

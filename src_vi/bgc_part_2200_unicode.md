<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Unicode, wide character, và mấy thứ đó

[i[Unicode]<]

Trước khi bắt đầu, lưu ý đây là vùng ngôn ngữ C đang phát triển sôi
động khi nó cố vượt qua vài, ờm, _cơn đau trưởng thành_. Giờ C23 đã
ra mắt, cập nhật ở đây là khả năng cao.

Phần lớn mọi người về cơ bản quan tâm câu hỏi tưởng đơn giản nhưng
lừa gạt, "Làm sao dùng bộ ký tự này-nọ trong C?" Ta sẽ tới đó. Nhưng
như ta sẽ thấy, có khi nó đã chạy sẵn trên hệ của bạn rồi. Hoặc bạn
có thể phải đổ qua thư viện bên thứ ba.

Ta sẽ nói về khá nhiều thứ trong chương này, vài cái không phụ thuộc
nền tảng, vài cái riêng của C.

Hãy xem sơ đồ những gì ta sắp xem:

* Nền tảng Unicode
* Nền tảng encoding ký tự
* Bộ ký tự nguồn và bộ ký tự thực thi
* Dùng Unicode và UTF-8
* Dùng các kiểu ký tự khác như `wchar_t`, `char16_t`, và `char32_t`

Lao vào nào!

## Unicode là gì?

Ngày xưa, ở Mỹ và phần lớn thế giới, phổ biến dùng encoding 7-bit
hay 8-bit cho ký tự trong bộ nhớ. Điều này nghĩa là ta có thể có 128
hay 256 ký tự (kể cả ký tự không in được) tổng cộng. Chừng đó ổn
với một thế giới lấy Mỹ làm trung tâm, nhưng hóa ra ngoài kia còn
bảng chữ cái khác, ai mà biết được? Tiếng Trung có hơn 50.000 ký
tự, và ngần đó không nhét vừa một byte.

Thế là người ta đẻ ra đủ kiểu cách khác nhau để biểu diễn bộ ký tự
riêng của mình. Và vậy cũng ổn, nhưng biến thành cơn ác mộng tương
thích.

Để thoát khỏi đó, Unicode được phát minh. Một bộ ký tự để cai trị
tất cả. Nó trải ra tới vô hạn (về cơ bản) nên ta sẽ không bao giờ
hết chỗ cho ký tự mới. Nó có tiếng Trung, Latin, Hy Lạp, chữ hình
nêm, ký hiệu cờ vua, emoji... gần như mọi thứ, thật đấy! Và liên tục
có thêm cái mới!

## Code point

[i[Unicode-->code points]<]

Tôi muốn nói về hai khái niệm ở đây. Hơi rối vì cả hai đều là số,
các số khác nhau cho cùng một thứ. Nhưng ráng theo tôi nào.

Định nghĩa _code point_ một cách lỏng lẻo là một giá trị số đại diện
cho một ký tự. (Code point cũng có thể đại diện cho ký tự điều khiển
không in được, nhưng cứ giả định tôi muốn nói tới cái gì đó như chữ
"B" hay ký tự "π".)

Mỗi code point đại diện cho một ký tự duy nhất. Và mỗi ký tự có một
code point số duy nhất gắn với nó.

Ví dụ, trong Unicode, giá trị số 66 đại diện cho "B", và 960 đại
diện cho "π". Các ánh xạ ký tự khác không phải Unicode dùng giá trị
khác, có thể, nhưng hãy quên chúng và tập trung vào Unicode, tương
lai!

Vậy đó là một chuyện: có một con số đại diện cho từng ký tự. Trong
Unicode, các số này chạy từ 0 tới hơn 1 triệu.

[i[Unicode-->code points]>]

Hiểu rồi chứ?

Vì ta sắp lật bàn tí đây.

## Encoding

[i[Unicode-->encoding]<]

Nếu bạn còn nhớ, một byte 8-bit có thể giữ giá trị từ 0-255, gồm cả
hai đầu. Chừng đó ổn với "B" là 66, cái đó vừa vặn trong một byte.
Nhưng "π" là 960, cái đó không vừa một byte! Ta cần byte khác. Làm
sao ta lưu hết mớ đó trong bộ nhớ? Hay mấy số lớn hơn, như 195.024?
Cái đó sẽ cần một số byte để giữ.

Câu hỏi lớn: các con số này được biểu diễn ra sao trong bộ nhớ? Đây
là cái ta gọi là _encoding_ của các ký tự.

Vậy ta có hai thứ: một là code point cho ta biết về cơ bản số sê-ri
của một ký tự cụ thể. Và ta có encoding cho ta biết ta sẽ biểu diễn
con số đó ra sao trong bộ nhớ.

Có cả đống encoding. Bạn có thể tự nghĩ ra encoding của mình ngay
bây giờ, nếu bạn muốn^[Ví dụ, ta có thể lưu code point trong một số
nguyên 32-bit big-endian. Thẳng thớm! Ta vừa phát minh ra một
encoding! Thật ra thì không; đó là cái encoding UTF-32BE. Chao ôi,
quay lại với công việc thôi!]. Nhưng ta sẽ xem vài encoding thực sự
phổ biến đang được dùng với Unicode.

[i[Unicode-->UTF-8]<]
[i[Unicode-->UTF-16]<]
[i[Unicode-->UTF-32]<]

|Encoding|Mô tả|
|:-----------------:|:--------------------------------------------------------------|
|UTF-8|Encoding hướng byte, dùng số byte thay đổi trên mỗi ký tự. Đây là cái nên dùng.|
|UTF-16|Encoding 16-bit cho mỗi ký tự[^091d].|
|UTF-32|Encoding 32-bit cho mỗi ký tự.|

[^091d]: Kiểu kiểu vậy. Về kỹ thuật, nó có độ rộng thay đổi, có cách
biểu diễn code point lớn hơn $2^{16}$ bằng cách ghép hai ký tự UTF-16
lại.

Với UTF-16 và UTF-32, thứ tự byte có ý nghĩa, nên bạn có thể thấy
UTF-16BE cho big-endian và UTF-16LE cho little-endian. Y vậy cho
UTF-32. Về kỹ thuật, nếu không chỉ định, bạn nên giả định
big-endian. Nhưng vì Windows dùng UTF-16 nhiều và nó little-endian,
đôi khi điều đó được giả định^[Có một ký tự đặc biệt tên _Byte Order
Mark_ (BOM), code point 0xFEFF, có thể tuỳ chọn đi trước luồng dữ
liệu và cho biết endianness. Tuy nhiên nó không bắt buộc.].

Xem vài ví dụ. Tôi sẽ viết giá trị theo hex vì nó đúng hai chữ số
cho mỗi byte 8-bit, và làm vậy dễ thấy mọi thứ xếp ra sao trong bộ
nhớ hơn.

|Ký tự|Code Point|UTF-16BE|UTF-32BE|UTF-16LE|UTF-32LE|UTF-8|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|`A`|41|0041|00000041|4100|41000000|41|
|`B`|42|0042|00000042|4200|42000000|42|
|`~`|7E|007E|0000007E|7E00|7E000000|7E|
|`π`|3C0|03C0|000003C0|C003|C0030000|CF80|
|`€`|20AC|20AC|000020AC|AC20|AC200000|E282AC|

[i[Unicode-->endianess]<]

Ngó trong đó tìm mẫu xem. Để ý UTF-16BE và UTF-32BE chỉ là code
point được biểu diễn thẳng dưới dạng giá trị 16 và 32-bit^[Lại, điều
này chỉ đúng trong UTF-16 cho các ký tự vừa trong hai byte.].

[i[Unicode-->UTF-16]>]
[i[Unicode-->UTF-32]>]

Little-endian cũng y vậy, chỉ khác là các byte theo thứ tự
little-endian.

[i[Unicode-->endianess]>]

Rồi ta có UTF-8 ở cuối. Đầu tiên bạn có thể để ý code point
một-byte được biểu diễn dưới dạng một byte. Cái đó hay. Bạn cũng có
thể để ý code point khác nhau chiếm số byte khác nhau. Đây là
encoding độ rộng thay đổi.

Nên ngay khi vượt qua một giá trị nào đó, UTF-8 bắt đầu dùng thêm
byte để lưu giá trị. Và chúng có vẻ không tương quan với giá trị
code point.

[flw[Chi tiết encoding UTF-8|UTF-8]] nằm ngoài phạm vi sách này,
nhưng biết thế này là đủ: nó có số byte thay đổi cho mỗi code point,
và các giá trị byte đó không khớp với code point _trừ 128 code
point đầu tiên_. Nếu bạn thật sự muốn biết thêm, [fl[Computerphile
có video về UTF-8 rất hay với Tom
Scott|https://www.youtube.com/watch?v=MijmeoH9LT4]].

Cái cuối đó là điều thú vị về Unicode và UTF-8 từ góc nhìn Bắc Mỹ:
nó tương thích ngược với encoding ASCII 7-bit! Nên nếu bạn quen với
ASCII, UTF-8 giống y vậy! Mọi tài liệu được encode bằng ASCII cũng
được encode bằng UTF-8! (Dĩ nhiên không phải ngược lại.)

Có lẽ chính điểm cuối này, hơn bất cứ điểm nào khác, đang đẩy UTF-8
thống trị thế giới.

[i[Unicode-->UTF-8]>]
[i[Unicode-->encoding]>]

## Bộ ký tự nguồn và thực thi {#src-exec-charset}

[i[Character sets]<]

Khi lập trình C, có (ít nhất) ba bộ ký tự đang chơi:

* Cái code của bạn tồn tại trên đĩa dưới dạng.
* [i[Character sets-->source]]Cái compiler dịch sang ngay khi bắt
  đầu compile (_bộ ký tự nguồn_). Có thể giống cái trên đĩa, hoặc
  không.
* [i[Character sets-->execution]]Cái compiler dịch bộ ký tự nguồn
  sang để thực thi (_bộ ký tự thực thi_). Có thể giống bộ ký tự
  nguồn, hoặc không.

Compiler của bạn có lẽ có tùy chọn để chọn các bộ ký tự này lúc
build.

[i[Character sets-->basic]<]

Bộ ký tự cơ bản cho cả nguồn và thực thi sẽ chứa các ký tự sau:

``` {.default}
A B C D E F G H I J K L M
N O P Q R S T U V W X Y Z
a b c d e f g h i j k l m
n o p q r s t u v w x y z
0 1 2 3 4 5 6 7 8 9
! " # % & ' ( ) * + , - . / :
; < = > ? [ \ ] ^ _ { | } ~
space tab vertical-tab
form-feed end-of-line
```

Đó là các ký tự bạn có thể dùng trong nguồn và vẫn portable 100%.

Bộ ký tự thực thi sẽ có thêm ký tự cho alert (chuông/chớp),
backspace, carriage return, và newline.

Nhưng phần lớn mọi người không đi tới mức cực đoan đó và thoải mái
dùng bộ ký tự mở rộng trong nguồn và chương trình chạy, nhất là bây
giờ Unicode và UTF-8 đang phổ biến hơn. Ý tôi là, bộ ký tự cơ bản
thậm chí không cho phép `@`, `$`, hay `` ` ``!

Đáng chú ý, đau đầu (dù làm được bằng escape sequence) khi gõ ký tự
Unicode chỉ bằng bộ ký tự cơ bản.

[i[Character sets-->basic]>]
[i[Character sets]>]

## Unicode trong C {#unicode-in-c}

Trước khi đi vào encoding trong C, hãy nói về Unicode từ góc độ code
point. Có cách trong C để chỉ định ký tự Unicode và chúng sẽ được
compiler dịch sang bộ ký tự thực thi^[Có lẽ compiler cố hết sức dịch
code point sang encoding output nào đó, nhưng tôi không tìm thấy
đảm bảo nào trong spec.].

Vậy làm sao ta làm?

Thử ký hiệu euro, code point 0x20AC. (Tôi viết nó bằng hex vì cả hai
cách biểu diễn nó trong C đều dùng hex.) Làm sao ta đặt nó vào code
C?

[i[`\u` Unicode escape]<]
Dùng escape `\u` để đặt nó trong chuỗi, ví dụ `"\u20AC"` (viết hoa
thường hex không quan trọng). Bạn phải đặt **đúng bốn** chữ số hex
sau `\u`, pad bằng số 0 đầu nếu cần.

Đây là ví dụ:

``` {.c}
char *s = "\u20AC1.23";

printf("%s\n", s);  // €1.23
```

[i[`\U` Unicode escape]<]

Vậy `\u` chạy với code point Unicode 16-bit, còn mấy cái lớn hơn
16-bit thì sao? Cho cái đó, ta cần chữ hoa: `\U`.

Ví dụ:

``` {.c}
char *s = "\U0001D4D1";

printf("%s\n", s);  // Prints a mathematical letter "B"
```

Giống `\u`, chỉ là 32-bit thay vì 16. Hai cái này tương đương:

``` {.c}
\u03C0
\U000003C0
```

[i[`\u` Unicode escape]>]
[i[`\U` Unicode escape]>]

Lại, các cái này được dịch sang [i[Character
sets-->execution]]bộ ký tự thực thi lúc compile. Chúng đại diện cho
code point Unicode, không phải encoding cụ thể nào. Thêm nữa, nếu
một code point Unicode không biểu diễn được trong bộ ký tự thực thi,
compiler có thể làm gì với nó cũng được.

Giờ, bạn có thể thắc mắc sao không làm thế này:

``` {.c}
char *s = "€1.23";

printf("%s\n", s);  // €1.23
```

Và có lẽ bạn làm được, với compiler hiện đại. [i[Character
sets-->source]]Bộ ký tự nguồn sẽ được compiler dịch sang [i[Character
sets-->execution]]bộ ký tự thực thi cho bạn. Nhưng compiler có quyền
nôn ra nếu tìm thấy ký tự nào không có trong bộ ký tự mở rộng của
nó, và ký hiệu € chắc chắn không có trong [i[Character
sets-->basic]]bộ ký tự cơ bản.

[i[`\u` Unicode escape]<]
[i[`\U` Unicode escape]<]

Lưu ý từ spec: bạn không thể dùng `\u` hay `\U` để encode bất kỳ code
point nào dưới 0xA0 trừ 0x24 (`$`), 0x40 (`@`), và 0x60 (`` `
``), đúng rồi, đó là bộ ba dấu câu phổ biến bị thiếu khỏi bộ ký tự
cơ bản. Rõ ràng hạn chế này được nới lỏng trong phiên bản spec sắp
tới.

[i[`\u` Unicode escape]>]
[i[`\U` Unicode escape]>]

Cuối cùng, bạn cũng có thể dùng các cái này trong định danh trong
code của mình, với vài hạn chế. Nhưng tôi không muốn đi vào đó ở
đây. Chương này ta chỉ tập trung xử lý chuỗi.

Và đó gần như là toàn bộ về Unicode trong C (trừ encoding).

## Ghi chú nhanh về UTF-8 trước khi lao vào bụi rậm {#utf8-quick}

[i[Unicode-->UTF-8]<]

Có thể file nguồn của bạn trên đĩa, các ký tự nguồn mở rộng, và các
ký tự thực thi mở rộng đều ở định dạng UTF-8. Và các thư viện bạn
dùng mong đợi UTF-8. Đây là tương lai rực rỡ của UTF-8 ở mọi nơi.

Nếu đúng vậy, và bạn không ngại không portable sang các hệ không
như thế, cứ chạy. Nhét ký tự Unicode vào nguồn và dữ liệu thoải mái.
Dùng chuỗi C thường và vui vẻ.

Nhiều thứ sẽ chạy được (dù không portable) vì chuỗi UTF-8 có thể
kết thúc bằng NUL an toàn y như chuỗi C nào khác. Nhưng có thể đổi
tính portable để xử lý ký tự dễ hơn là cái đánh đổi đáng giá với
bạn.

Tuy nhiên, có vài lưu ý:

* Những thứ như `strlen()` báo số byte trong chuỗi, không phải số ký
  tự. ([i[`mbstowcs()` function-->with
  UTF-8]]`mbstowcs()` trả về số ký tự trong chuỗi khi bạn chuyển nó
  sang wide character. POSIX mở rộng cái này để bạn có thể truyền
  `NULL` làm đối số đầu nếu chỉ muốn đếm số ký tự.)

* Những cái sau sẽ không chạy đúng với ký tự hơn một byte:
  [i[`strtok()` function-->with UTF-8]]`strtok()`, [i[`strchr()`
  function-->with UTF-8]]`strchr()` (dùng [i[`strstr()` function-->with
  UTF-8]]`strstr()` thay thế), họ hàm `strspn()`, [i[`toupper()`
  function-->with UTF-8]]`toupper()`, [i[`tolower()` function-->with
  UTF-8]]`tolower()`, họ hàm [i[`isalpha()` function-->with
  UTF-8]]`isalpha()`, và chắc còn nữa. Cảnh giác với bất cứ gì hoạt
  động trên byte.

* [i[`printf()` function-->with UTF-8]]Các biến thể `printf()` cho
  phép chỉ in ra một số byte của chuỗi^[Với format specifier kiểu
  `"%.12s"` chẳng hạn.]. Bạn cần chắc chắn in đúng số byte để kết
  thúc ở ranh giới ký tự.

* [i[`malloc()` function-->with UTF-8]]Nếu bạn muốn `malloc()` chỗ
  cho chuỗi, hay khai báo mảng `char` cho một chuỗi, lưu ý kích
  thước tối đa có thể nhiều hơn bạn nghĩ. Mỗi ký tự có thể chiếm tới
  [i[`MB_LEN_MAX` macro]]`MB_LEN_MAX` byte (từ `<limits.h>`), trừ
  các ký tự trong bộ ký tự cơ bản đảm bảo là một byte.

Và chắc còn nữa mà tôi chưa khám phá ra. Cho tôi biết còn cái bẫy
nào ngoài kia nữa nhé...

[i[Unicode-->UTF-8]>]

## Các kiểu ký tự khác nhau

Tôi muốn giới thiệu thêm kiểu ký tự. Ta quen với `char`, đúng không?

Nhưng cái đó quá dễ. Hãy làm mọi thứ khó hơn nhiều! Hoan hô!

### Ký tự multibyte

[i[Multibyte characters]<]

Trước hết, tôi muốn có thể thay đổi cách bạn nghĩ về chuỗi (mảng
`char`) là gì. Chúng là _chuỗi multibyte_ được tạo từ _ký tự
multibyte_.

Đúng rồi, cái chuỗi ký tự bình thường của bạn là multibyte. Khi ai
đó nói "C string", họ đang nói "chuỗi multibyte C".

Kể cả khi một ký tự cụ thể trong chuỗi chỉ là một byte, hay nếu
chuỗi được tạo chỉ từ ký tự đơn, nó vẫn được gọi là chuỗi multibyte.

Ví dụ:

``` {.c}
char c[128] = "Hello, world!";  // Multibyte string
```

Cái ta đang nói ở đây là một ký tự cụ thể không thuộc bộ ký tự cơ
bản có thể được tạo từ nhiều byte. Tối đa
[i[`MB_LEN_MAX` macro]]`MB_LEN_MAX` byte (từ `<limits.h>`). Chắc,
trên màn hình nó chỉ trông như một ký tự, nhưng có thể là nhiều
byte.

Bạn cũng có thể ném giá trị Unicode vào đó, như ta thấy trước đó:

``` {.c}
char *s = "\u20AC1.23";

printf("%s\n", s);  // €1.23
```

Nhưng ở đây ta vào vùng kỳ lạ, vì xem này:

[i[`strlen()` function-->with UTF-8]<]

``` {.c}
char *s = "\u20AC1.23";  // €1.23

printf("%zu\n", strlen(s));  // 7!
```

Độ dài chuỗi của `"€1.23"` là `7`?! Đúng vậy! Ờ, trên hệ của tôi,
đúng! Nhớ `strlen()` trả về số byte trong chuỗi, không phải số ký
tự. (Khi ta tới "wide character", sắp tới, ta sẽ thấy cách lấy số
ký tự trong chuỗi.)

[i[`strlen()` function-->with UTF-8]>]

Lưu ý C cho phép hằng `char` multibyte riêng lẻ (khác với `char*`),
nhưng hành vi của chúng thay đổi theo implementation và compiler có
thể cảnh báo về nó.

GCC, chẳng hạn, cảnh báo về hằng ký tự multi-character cho hai dòng
sau (và, trên hệ của tôi, in ra encoding UTF-8):

``` {.c}
printf("%x\n", '€');
printf("%x\n", '\u20ac');
```

[i[Multibyte characters]>]

### Wide character {#wide-characters}

[i[Wide characters]<]

Nếu bạn không phải ký tự multibyte, thì bạn là _wide character_.

Wide character là một giá trị đơn có thể đại diện duy nhất cho bất
kỳ ký tự nào trong locale hiện tại. Nó tương đương với code point
Unicode. Nhưng có thể không. Hoặc có thể.

Về cơ bản, chuỗi ký tự multibyte là mảng các byte, thì chuỗi wide
character là mảng các _ký tự_. Nên bạn có thể bắt đầu suy nghĩ theo
kiểu từng-ký-tự thay vì từng-byte (cái sau rối mù khi ký tự bắt đầu
chiếm số byte thay đổi).

[i[`wchar_t` type]<]

Wide character có thể được biểu diễn bằng một số kiểu, nhưng cái nổi
bật nhất là `wchar_t`. Nó là cái chính. Giống `char`, chỉ là wide.

Bạn có thể thắc mắc nếu bạn không biết có phải Unicode hay không,
sao mà cho bạn nhiều linh hoạt trong việc viết code? `wchar_t` mở vài
cửa đó ra, vì có bộ hàm phong phú bạn có thể dùng để xử lý chuỗi
`wchar_t` (như lấy độ dài, v.v.) mà không quan tâm encoding.

## Dùng wide character và `wchar_t`

Đến lúc cho kiểu mới: `wchar_t`. Đây là kiểu wide character chính.
Nhớ cách `char` chỉ một byte chứ? Và một byte có thể không đủ đại
diện mọi ký tự? Ừ, cái này đủ.

Để dùng `wchar_t`, include `<wchar.h>`.

Nó to bao nhiêu byte? Không rõ lắm. Có thể 16 bit. Có thể 32 bit.

Nhưng khoan, bạn đang nói, nếu chỉ 16 bit, nó không đủ giữ hết code
point Unicode đúng không? Đúng, không đủ. Spec không yêu cầu nó phải
thế. Nó chỉ phải biểu diễn được mọi ký tự trong locale hiện tại.

Điều này có thể gây đau đầu với Unicode trên nền tảng `wchar_t`
16-bit (ờ hèm, Windows). Nhưng cái đó ngoài phạm vi sách này.

[i[`L` wide character prefix]<]

Bạn có thể khai báo chuỗi hay ký tự kiểu này với tiền tố `L`, và bạn
có thể in chúng với format specifier `%ls` ("ell ess"). Hoặc in một
`wchar_t` riêng lẻ với `%lc`.

``` {.c}
wchar_t *s = L"Hello, world!";
wchar_t c = L'B';

printf("%ls %lc\n", s, c);
```

[i[`L` wide character prefix]>]

Giờ, các ký tự đó có được lưu dưới dạng code point Unicode hay
không? Tùy implementation. Nhưng bạn có thể kiểm tra xem có phải
không bằng macro [i[`__STDC_ISO_10646__` macro]]
`__STDC_ISO_10646__`. Nếu cái này được định nghĩa, câu trả lời là,
"Nó là Unicode!"

Chi tiết hơn, giá trị trong macro đó là số nguyên dạng `yyyymm` cho
bạn biết bạn có thể dựa vào chuẩn Unicode nào, bất cứ cái nào đang
có hiệu lực vào ngày đó.

Nhưng dùng chúng thế nào?

### Chuyển multibyte sang `wchar_t`

Vậy làm sao từ chuỗi chuẩn hướng byte sang chuỗi wide hướng ký tự và
ngược lại?

Ta có thể dùng vài hàm chuyển chuỗi để làm chuyện này.

Đầu tiên, vài quy ước đặt tên bạn sẽ thấy trong các hàm này:

* `mb`: multibyte
* `wc`: wide character
* `mbs`: multibyte string
* `wcs`: wide character string

Vậy nếu ta muốn chuyển chuỗi multibyte thành chuỗi wide character,
ta có thể gọi `mbstowcs()`. Và chiều ngược lại: `wcstombs()`.

[i[`mbtowc()` function]]
[i[`wctomb()` function]]
[i[`mbstowcs()` function]]
[i[`wcstombs()` function]]

|Hàm chuyển|Mô tả|
|------------|---------------------------------------------------|
|`mbtowc()`|Chuyển ký tự multibyte sang wide character.|
|`wctomb()`|Chuyển wide character sang ký tự multibyte.|
|`mbstowcs()`|Chuyển chuỗi multibyte sang chuỗi wide.|
|`wcstombs()`|Chuyển chuỗi wide sang chuỗi multibyte.|

Làm demo nhanh ta chuyển chuỗi multibyte thành chuỗi wide character,
rồi so độ dài chuỗi của hai cái bằng các hàm tương ứng.

[i[`mbstowcs()` function]<]

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <wchar.h>
#include <string.h>
#include <locale.h>

int main(void)
{
    // Get out of the C locale to one that likely has the euro symbol
    setlocale(LC_ALL, "");

    // Original multibyte string with a euro symbol (Unicode point 20ac)
    char *mb_string = "The cost is \u20ac1.23";  // €1.23
    size_t mb_len = strlen(mb_string);

    // Wide character array that will hold the converted string
    wchar_t wc_string[128];  // Holds up to 128 wide characters

    // Convert the MB string to WC; this returns the number of wide chars
    size_t wc_len = mbstowcs(wc_string, mb_string, 128);

    // Print result--note the %ls for wide char strings
    printf("multibyte: \"%s\" (%zu bytes)\n", mb_string, mb_len);
    printf("wide char: \"%ls\" (%zu characters)\n", wc_string, wc_len);
}
```

[i[`wchar_t` type]>]

Trên hệ của tôi, cái này in ra:

``` {.default}
multibyte: "The cost is €1.23" (19 bytes)
wide char: "The cost is €1.23" (17 characters)
```

(Máy bạn có thể khác số byte tùy locale.)

Một điều thú vị cần lưu ý là `mbstowcs()`, ngoài việc chuyển chuỗi
multibyte sang wide, còn trả về độ dài (tính bằng ký tự) của chuỗi
wide character. Trên hệ tuân thủ POSIX, bạn có thể tận dụng chế độ
đặc biệt nơi nó _chỉ_ trả về độ dài tính bằng ký tự của một chuỗi
multibyte cho trước: bạn chỉ truyền `NULL` cho đích, và `0` cho số
ký tự tối đa cần chuyển (giá trị này bị bỏ qua).

(Trong code dưới, tôi đang dùng bộ ký tự nguồn mở rộng của mình, bạn
có thể phải thay bằng escape `\u`.)

``` {.c}
setlocale(LC_ALL, "");

// The following string has 7 characters
size_t len_in_chars = mbstowcs(NULL, "§¶°±π€•", 0);

printf("%zu", len_in_chars);  // 7
```

[i[`mbstowcs()` function]>]

Lại, đó là mở rộng POSIX không portable.

Và, dĩ nhiên, nếu bạn muốn chuyển chiều ngược lại, đó là
[i[`wcstombs()` function]] `wcstombs()`.

## Chức năng wide character

Một khi đã ở xứ wide character, ta có đủ loại chức năng trong tay.
Tôi sẽ chỉ tóm tắt một đống hàm ở đây, nhưng về cơ bản cái ta có ở
đây là phiên bản wide character của các hàm chuỗi multibyte ta quen
thuộc. (Ví dụ, ta biết `strlen()` cho chuỗi multibyte; có
[i[`wcslen()` function]] `wcslen()` cho chuỗi wide character.)

### `wint_t`

[i[`wint_t` type]<]

Nhiều hàm trong đám này dùng `wint_t` để giữ ký tự đơn, dù chúng
được truyền vào hay trả về.

Về bản chất nó có liên quan đến `wchar_t`. `wint_t` là số nguyên có
thể đại diện mọi giá trị trong bộ ký tự mở rộng, và cũng một ký tự
end-of-file đặc biệt, `WEOF`.

[i[`wint_t` type]>]

Cái này được dùng bởi một số hàm wide character hướng ký tự đơn.

### Hướng luồng I/O {#io-stream-orientation}

[i[I/O stream orientation]<]

Tóm gọn ở đây là đừng trộn lẫn hàm hướng byte (như `fprintf()`) với
hàm hướng wide (như `fwprintf()`). Quyết định xem luồng sẽ hướng
byte hay hướng wide và bám lấy kiểu hàm I/O đó.

Chi tiết hơn: luồng có thể hướng byte hoặc hướng wide. Khi luồng
mới được tạo, nó không có hướng, nhưng lần đọc hoặc ghi đầu sẽ đặt
hướng.

Nếu bạn dùng phép wide trước (như `fwprintf()`) nó sẽ đặt hướng
luồng sang wide.

Nếu bạn dùng phép byte trước (như `fprintf()`) nó sẽ đặt hướng
luồng theo byte.

Bạn có thể đặt thủ công một luồng chưa có hướng theo cách này hoặc
cách kia bằng lời gọi [i[`fwide()` function]] `fwide()`. Bạn có thể
dùng cùng hàm đó để lấy hướng của luồng.

Nếu cần đổi hướng giữa chừng, bạn có thể làm bằng `freopen()`.

[i[I/O stream orientation]>]

### Hàm I/O

Thường include `<stdio.h>` và `<wchar.h>` cho mấy cái này.

[i[`wprintf()` function]]
[i[`wscanf()` function]]
[i[`getwchar()` function]]
[i[`putwchar()` function]]
[i[`fwprintf()` function]]
[i[`fwscanf()` function]]
[i[`fgetwc()` function]]
[i[`fputwc()` function]]
[i[`fgetws()` function]]
[i[`fputws()` function]]
[i[`swprintf()` function]]
[i[`swscanf()` function]]
[i[`vfwprintf()` function]]
[i[`vfwscanf()` function]]
[i[`vswprintf()` function]]
[i[`vswscanf()` function]]
[i[`vwprintf()` function]]
[i[`vwscanf()` function]]
[i[`ungetwc()` function]]
[i[`fwide()` function]]

|Hàm I/O|Mô tả|
|------------|---------------------------------------------------|
|`wprintf()`|Output console có định dạng.|
|`wscanf()`|Input console có định dạng.|
|`getwchar()`|Input console hướng ký tự.|
|`putwchar()`|Output console hướng ký tự.|
|`fwprintf()`|Output file có định dạng.|
|`fwscanf()`|Input file có định dạng.|
|`fgetwc()`|Input file hướng ký tự.|
|`fputwc()`|Output file hướng ký tự.|
|`fgetws()`|Input file hướng chuỗi.|
|`fputws()`|Output file hướng chuỗi.|
|`swprintf()`|Output chuỗi có định dạng.|
|`swscanf()`|Input chuỗi có định dạng.|
|`vfwprintf()`|Output file có định dạng, variadic.|
|`vfwscanf()`|Input file có định dạng, variadic.|
|`vswprintf()`|Output chuỗi có định dạng, variadic.|
|`vswscanf()`|Input chuỗi có định dạng, variadic.|
|`vwprintf()`|Output console có định dạng, variadic.|
|`vwscanf()`|Input console có định dạng, variadic.|
|`ungetwc()`|Đẩy một wide character ngược lại luồng output.|
|`fwide()`|Lấy hoặc đặt hướng multibyte/wide của luồng.|

### Hàm chuyển kiểu

Thường include `<wchar.h>` cho mấy cái này.

[i[`wcstod()` function]]
[i[`wcstof()` function]]
[i[`wcstold()` function]]
[i[`wcstol()` function]]
[i[`wcstoll()` function]]
[i[`wcstoul()` function]]
[i[`wcstoull()` function]]

|Hàm chuyển|Mô tả|
|------------|---------------------------------------------------|
|`wcstod()`|Chuyển chuỗi sang `double`.|
|`wcstof()`|Chuyển chuỗi sang `float`.|
|`wcstold()`|Chuyển chuỗi sang `long double`.|
|`wcstol()`|Chuyển chuỗi sang `long`.|
|`wcstoll()`|Chuyển chuỗi sang `long long`.|
|`wcstoul()`|Chuyển chuỗi sang `unsigned long`.|
|`wcstoull()`|Chuyển chuỗi sang `unsigned long long`.|

### Hàm copy chuỗi và bộ nhớ

Thường include `<wchar.h>` cho mấy cái này.

[i[`wcscpy()` function]]
[i[`wcsncpy()` function]]
[i[`wmemcpy()` function]]
[i[`wmemmove()` function]]
[i[`wcscat()` function]]
[i[`wcsncat()` function]]

|Hàm copy|Mô tả|
|----|----------------------------------------------|
|`wcscpy()`|Copy chuỗi.|
|`wcsncpy()`|Copy chuỗi, giới hạn độ dài.|
|`wmemcpy()`|Copy bộ nhớ.|
|`wmemmove()`|Copy bộ nhớ có thể chồng lấn.|
|`wcscat()`|Nối chuỗi.|
|`wcsncat()`|Nối chuỗi, giới hạn độ dài.|

### Hàm so sánh chuỗi và bộ nhớ

Thường include `<wchar.h>` cho mấy cái này.

[i[`wcscmp()` function]]
[i[`wcsncmp()` function]]
[i[`wcscoll()` function]]
[i[`wmemcmp()` function]]
[i[`wcsxfrm()` function]]

|Hàm so sánh|Mô tả|
|-------------------|---------------------------------------------------------------|
|`wcscmp()`|So sánh chuỗi theo thứ tự từ điển.|
|`wcsncmp()`|So sánh chuỗi theo thứ tự từ điển, giới hạn độ dài.|
|`wcscoll()`|So sánh chuỗi theo thứ tự từ điển của locale.|
|`wmemcmp()`|So sánh bộ nhớ theo thứ tự từ điển.|
|`wcsxfrm()`|Biến chuỗi thành phiên bản khiến `wcscmp()` hành xử như `wcscoll()`[^97d0].|

[^97d0]: `wcscoll()` là `wcsxfrm()` rồi theo sau bởi `wcscmp()`.

### Hàm tìm kiếm chuỗi

Thường include `<wchar.h>` cho mấy cái này.

[i[`wcschr()` function]]
[i[`wcsrchr()` function]]
[i[`wmemchr()` function]]
[i[`wcsstr()` function]]
[i[`wcspbrk()` function]]
[i[`wcsspn()` function]]
[i[`wcscspn()` function]]
[i[`wcstok()` function]]

|Hàm tìm|Mô tả|
|-|-|
|`wcschr()`|Tìm một ký tự trong chuỗi.|
|`wcsrchr()`|Tìm một ký tự trong chuỗi từ phía sau.|
|`wmemchr()`|Tìm một ký tự trong bộ nhớ.|
|`wcsstr()`|Tìm chuỗi con trong chuỗi.|
|`wcspbrk()`|Tìm bất kỳ ký tự nào trong một tập ký tự trong chuỗi.|
|`wcsspn()`|Tìm độ dài chuỗi con gồm bất kỳ ký tự nào trong tập.|
|`wcscspn()`|Tìm độ dài chuỗi con trước bất kỳ ký tự nào trong tập.|
|`wcstok()`|Tìm token trong chuỗi.|

### Hàm về độ dài/linh tinh

Thường include `<wchar.h>` cho mấy cái này.

[i[`wcslen()` function]]
[i[`wmemset()` function]]
[i[`wcsftime()` function]]

|Hàm Length/Misc|Mô tả|
|-|-|
|`wcslen()`|Trả về độ dài chuỗi.|
|`wmemset()`|Đặt ký tự trong bộ nhớ.|
|`wcsftime()`|Output ngày và giờ có định dạng.|

### Hàm phân loại ký tự

Include `<wctype.h>` cho mấy cái này.

[i[`iswalnum()` function]]
[i[`iswalpha()` function]]
[i[`iswblank()` function]]
[i[`iswcntrl()` function]]
[i[`iswdigit()` function]]
[i[`iswgraph()` function]]
[i[`iswlower()` function]]
[i[`iswprint()` function]]
[i[`iswpunct()` function]]
[i[`iswspace()` function]]
[i[`iswupper()` function]]
[i[`iswxdigit()` function]]
[i[`towlower()` function]]
[i[`towupper()` function]]

|Hàm Length/Misc|Mô tả|
|------------|---------------------------------------------------|
|`iswalnum()`|True nếu ký tự là chữ và số.|
|`iswalpha()`|True nếu ký tự là chữ cái.|
|`iswblank()`|True nếu ký tự là khoảng trắng (giống space, nhưng không phải newline).|
|`iswcntrl()`|True nếu ký tự là ký tự điều khiển.|
|`iswdigit()`|True nếu ký tự là chữ số.|
|`iswgraph()`|True nếu ký tự in được (trừ space).|
|`iswlower()`|True nếu ký tự là chữ thường.|
|`iswprint()`|True nếu ký tự in được (kể cả space).|
|`iswpunct()`|True nếu ký tự là dấu câu.|
|`iswspace()`|True nếu ký tự là khoảng trắng.|
|`iswupper()`|True nếu ký tự là chữ hoa.|
|`iswxdigit()`|True nếu ký tự là chữ số hex.|
|`towlower()`|Chuyển ký tự thành chữ thường.|
|`towupper()`|Chuyển ký tự thành chữ hoa.|

## Parse state, hàm restartable

[i[Multibyte characters-->parse state]<]

Ta sẽ đi sâu chút vào ruột gan của chuyển multibyte, nhưng đây là
chuyện hay để hiểu, về khái niệm.

Tưởng tượng chương trình của bạn lấy một chuỗi ký tự multibyte và
biến chúng thành wide character, hoặc ngược lại. Có lúc, nó có thể
đang phân tích dở một ký tự, hoặc có thể phải đợi thêm byte trước
khi chốt giá trị cuối.

Parse state này được lưu trong một biến mờ kiểu `mbstate_t` và được
dùng mỗi khi chuyển được thực hiện. Đó là cách các hàm chuyển theo
dõi chúng đang ở đâu giữa chừng.

Và nếu bạn đổi sang chuỗi ký tự khác giữa chừng, hoặc cố seek sang
chỗ khác trong chuỗi input, nó có thể bị rối.

Giờ bạn có thể bắt bẻ tôi: ta vừa chuyển vài cái ở trên, mà tôi
chưa bao giờ nhắc tới `mbstate_t` nào.

Đó là vì các hàm chuyển như `mbstowcs()`, `wctomb()`, v.v. mỗi cái
có biến `mbstate_t` riêng mà chúng dùng. Tuy nhiên chỉ có một cho
mỗi hàm, nên nếu bạn đang viết code đa luồng, chúng không an toàn
để dùng.

May thay, C định nghĩa phiên bản _restartable_ của các hàm này,
trong đó bạn có thể truyền vào `mbstate_t` riêng trên cơ sở từng
luồng nếu cần. Nếu bạn làm đa luồng, dùng mấy cái này!

Ghi chú nhanh về khởi tạo biến `mbstate_t`: chỉ cần `memset()` nó về
0. Không có hàm sẵn nào để ép nó khởi tạo.

``` {.c}
mbstate_t mbs;

// Set the state to the initial state
memset(&mbs, 0, sizeof mbs);
```

Đây là danh sách các hàm chuyển restartable, để ý quy ước đặt tên
chèn thêm "`r`" sau kiểu "from":

* `mbrtowc()`, multibyte sang wide character
* `wcrtomb()`, wide character sang multibyte
* `mbsrtowcs()`, chuỗi multibyte sang chuỗi wide character
* `wcsrtombs()`, chuỗi wide character sang chuỗi multibyte

Chúng thực sự giống với đồng nghiệp không restartable, trừ việc
chúng yêu cầu bạn truyền vào pointer tới biến `mbstate_t` riêng của
bạn. Và chúng cũng sửa pointer chuỗi nguồn (để giúp bạn nếu phát
hiện byte không hợp lệ), nên có thể hữu ích khi lưu bản sao của bản
gốc.

Đây là ví dụ trước đó trong chương, sửa lại để truyền `mbstate_t`
riêng vào.

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>
#include <wchar.h>
#include <string.h>
#include <locale.h>

int main(void)
{
    // Get out of the C locale to one that likely has the euro symbol
    setlocale(LC_ALL, "");

    // Original multibyte string with a euro symbol (Unicode point 20ac)
    char *mb_string = "The cost is \u20ac1.23";  // €1.23
    size_t mb_len = strlen(mb_string);

    // Wide character array that will hold the converted string
    wchar_t wc_string[128];  // Holds up to 128 wide characters

    // Set up the conversion state
    mbstate_t mbs;
    memset(&mbs, 0, sizeof mbs);  // Initial state

    // mbsrtowcs() modifies the input pointer to point at the first
    // invalid character, or NULL if successful. Let's make a copy of
    // the pointer for mbsrtowcs() to mess with so our original is
    // unchanged.
    //
    // This example will probably be successful, but we check farther
    // down to see.
    const char *invalid = mb_string;

    // Convert the MB string to WC; this returns the number of wide chars
    size_t wc_len = mbsrtowcs(wc_string, &invalid, 128, &mbs);

    if (invalid == NULL) {
        printf("No invalid characters found\n");

        // Print result--note the %ls for wide char strings
        printf("multibyte: \"%s\" (%zu bytes)\n", mb_string, mb_len);
        printf("wide char: \"%ls\" (%zu characters)\n", wc_string, wc_len);
    } else {
        ptrdiff_t offset = invalid - mb_string;
        printf("Invalid character at offset %td\n", offset);
    }
}
```

Với các hàm chuyển tự quản state riêng, bạn có thể reset state nội
bộ của chúng về trạng thái ban đầu bằng cách truyền `NULL` cho các
đối số `char*` của chúng, ví dụ:

``` {.c}
mbstowcs(NULL, NULL, 0);   // Reset the parse state for mbstowcs()
mbstowcs(dest, src, 100);  // Parse some stuff
```

Với I/O, mỗi luồng wide tự quản `mbstate_t` của mình và dùng nó cho
chuyển input và output theo diễn biến.

Và một số hàm I/O hướng byte như `printf()` và `scanf()` giữ state
nội bộ riêng khi làm việc.

Cuối cùng, các hàm chuyển restartable này thật ra có state nội bộ
riêng nếu bạn truyền `NULL` cho tham số `mbstate_t`. Điều này làm
chúng hành xử giống với đồng nghiệp không restartable hơn.

[i[Multibyte characters-->parse state]>]
[i[Wide characters]>]

## Encoding Unicode và C

Trong phần này, ta sẽ xem C làm được (và không làm được) gì với ba
encoding Unicode cụ thể: UTF-8, UTF-16, và UTF-32.

### UTF-8

[i[Unicode-->UTF-8]<]

Để làm mới trước phần này, đọc lại [ghi chú nhanh UTF-8 ở
trên](#utf8-quick).

Ngoài ra, C có khả năng UTF-8 gì?

Ờ, không nhiều, tiếc thay.

[i[`u8` UTF-8 prefix]<]

Bạn có thể nói với C rằng bạn muốn cụ thể một string literal được
encode UTF-8, và nó sẽ làm cho bạn. Bạn có thể đặt tiền tố `u8` trước
chuỗi:

``` {.c}
char *s = u8"Hello, world!";

printf("%s\n", s);   // Hello, world!--if you can output UTF-8
```

Giờ, bạn có thể nhét ký tự Unicode vào đó không?

``` {.c}
char *s = u8"€123";
```

[i[`u8` UTF-8 prefix]>]

Được! Nếu bộ ký tự nguồn mở rộng hỗ trợ nó. (gcc hỗ trợ.)

Nếu không hỗ trợ thì sao? Bạn có thể chỉ định code point Unicode với
người bạn thân `\u` và `\U`, [như đã nói ở trên](#unicode-in-c).

Nhưng cái đó là hết. Không có cách portable nào trong thư viện chuẩn
để lấy input tùy ý và biến nó thành UTF-8 trừ khi locale của bạn là
UTF-8. Hoặc parse UTF-8 trừ khi locale của bạn là UTF-8.

Nên nếu bạn muốn làm, hoặc ở trong locale UTF-8 và:

[i[`setlocale()` function]<]

``` {.c}
setlocale(LC_ALL, "");
```

hoặc tìm ra tên locale UTF-8 trên máy cục bộ và đặt nó tường minh
kiểu:

``` {.c}
setlocale(LC_ALL, "en_US.UTF-8");  // Non-portable name
```

[i[`setlocale()` function]>]

Hoặc dùng [thư viện bên thứ ba](#utf-3rd-party).

[i[Unicode-->UTF-8]>]

### UTF-16, UTF-32, `char16_t`, và `char32_t`

[i[Unicode-->UTF-16]<]
[i[Unicode-->UTF-32]<]
[i[`char16_t` type]<]
[i[`char32_t` type]<]

`char16_t` và `char32_t` là vài kiểu ký tự có tiềm năng wide khác
với kích thước 16 bit và 32 bit, tương ứng. Không nhất thiết là
wide, vì nếu chúng không thể đại diện mọi ký tự trong locale hiện
tại, chúng mất đi tính wide character. Nhưng spec gọi chúng là kiểu
"wide character" khắp nơi, nên đành vậy.

Mấy cái này có ở đây để làm mọi thứ thân thiện với Unicode hơn một
chút, có tiềm năng.

Để dùng, include `<uchar.h>`. (Đó là "u", không phải "w".)

Header file này không có trên OS X, tiếc. Nếu bạn chỉ muốn kiểu, bạn
có thể:

``` {.c}
#include <stdint.h>

typedef int_least16_t char16_t;
typedef int_least32_t char32_t;
```

Nhưng nếu cũng muốn hàm, cái đó tùy bạn.

[i[`u` Unicode prefix]<]
[i[`U` Unicode prefix]<]

Giả sử bạn vẫn ổn để tiếp, bạn có thể khai báo chuỗi hay ký tự các
kiểu này với tiền tố `u` và `U`:

``` {.c}
char16_t *s = u"Hello, world!";
char16_t c = u'B';

char32_t *t = U"Hello, world!";
char32_t d = U'B';
```

[i[`char32_t` type]>]
[i[`u` Unicode prefix]>]
[i[`U` Unicode prefix]>]

Giờ, giá trị trong mấy cái này có được lưu theo UTF-16 hay UTF-32
không? Tùy implementation.

Nhưng bạn có thể kiểm tra xem có phải không. Nếu các macro
[i[`__STDC_UTF_16__` macro]] `__STDC_UTF_16__` hoặc
[i[`__STDC_UTF_32__ macro`]] `__STDC_UTF_32__` được định nghĩa (thành
`1`) nghĩa là các kiểu giữ UTF-16 hoặc UTF-32, tương ứng.

Nếu bạn tò mò, và tôi biết bạn tò mò, các giá trị, nếu là UTF-16 hay
UTF-32, được lưu theo endianness của máy. Tức là, bạn có thể so
chúng thẳng với giá trị code point Unicode:

``` {.c}
char16_t pi = u"\u03C0";  // pi symbol

#if __STDC_UTF_16__
pi == 0x3C0;  // Always true
#else
pi == 0x3C0;  // Probably not true
#endif
```

[i[`char16_t` type]>]
[i[Unicode-->UTF-16]>]
[i[Unicode-->UTF-32]>]

### Chuyển Multibyte

Bạn có thể chuyển từ encoding multibyte sang `char16_t` hay
`char32_t` bằng vài hàm hỗ trợ.

(Như tôi đã nói, kết quả có thể không phải UTF-16 hay UTF-32 trừ khi
macro tương ứng được đặt thành `1`.)

Mọi hàm này đều restartable (tức là bạn truyền vào `mbstate_t`
riêng), và mọi hàm đều thao tác theo từng ký
tự^[Kiểu kiểu vậy, mọi thứ trở nên lạ với encoding UTF-16 multi-
`char16_t`.].

[i[`mbrtoc16()` function]]
[i[`mbrtoc32()` function]]
[i[`c16rtomb()` function]]
[i[`c32rtomb()` function]]

|Hàm chuyển|Mô tả|
|-|-|
|`mbrtoc16()`|Chuyển ký tự multibyte sang ký tự `char16_t`.|
|`mbrtoc32()`|Chuyển ký tự multibyte sang ký tự `char32_t`.|
|`c16rtomb()`|Chuyển ký tự `char16_t` sang ký tự multibyte.|
|`c32rtomb()`|Chuyển ký tự `char32_t` sang ký tự multibyte.|

### Thư viện bên thứ ba {#utf-3rd-party}

Cho chuyển hạng nặng giữa các encoding cụ thể khác nhau, có vài thư
viện chín muồi đáng xem. Lưu ý tôi chưa dùng cái nào trong số này.

* [flw[iconv|Iconv]], Internationalization Conversion, một API chuẩn
  POSIX phổ biến có sẵn trên các nền tảng lớn.
* [fl[ICU|http://site.icu-project.org/]], International Components
  for Unicode. Ít nhất một blogger thấy cái này dễ dùng.

Nếu bạn biết thư viện đáng chú ý khác, cho tôi biết.

[i[Unicode]>]

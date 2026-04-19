<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Locale và quốc tế hoá

[i[Locale]<]

_Localization_ (bản địa hoá) là quá trình làm cho app của bạn sẵn
sàng hoạt động tốt ở các locale (hay quốc gia) khác nhau.

Như bạn có thể biết, không phải ai cũng dùng cùng ký tự cho dấu thập
phân hay cho dấu phân cách hàng nghìn, hay cho đơn vị tiền tệ.

Các locale này có tên, và bạn có thể chọn một cái để dùng. Ví dụ,
locale Mỹ có thể viết một con số như:

100,000.00

Còn ở Brazil, cùng số đó có thể được viết với dấu phẩy và dấu chấm
đổi chỗ:

100.000,00

Chuyện này dễ dàng hơn khi bạn viết code sao cho dễ chuyển sang các
quốc tịch khác!

Ừ, kiểu kiểu. Hoá ra C chỉ có đúng một locale sẵn, và nó bị giới
hạn. Spec chừa ra khá nhiều chỗ mập mờ ở đây; khó mà thật sự
portable hoàn toàn.

Nhưng ta sẽ cố gắng hết sức!

## Đặt localization, nhanh và bẩn

Với các lời gọi này, include [i[`locale.h` header file]] `<locale.h>`.

Về cơ bản chỉ có một việc bạn có thể làm portable ở đây khi khai báo
một locale cụ thể. Đây rất có thể là điều bạn muốn làm nếu định đụng
tới locale:

[i[`setlocale()` function]<]

``` {.c}
setlocale(LC_ALL, "");  // Use this environment's locale for everything
```

Bạn sẽ muốn gọi nó để chương trình được khởi tạo với locale hiện tại
của bạn.

Đi vào chi tiết hơn, có một chuyện nữa bạn làm được mà vẫn portable:

``` {.c}
setlocale(LC_ALL, "C");  // Use the default C locale
```

nhưng cái đó được gọi mặc định mỗi lần chương trình của bạn khởi
chạy, nên không mấy khi cần tự gọi.

Trong chuỗi thứ hai đó, bạn có thể chỉ định bất kỳ locale nào được
hệ thống của bạn hỗ trợ. Chuyện này hoàn toàn phụ thuộc hệ, nên sẽ
khác nhau. Trên hệ của tôi, tôi có thể chỉ định cái này:

``` {.c}
setlocale(LC_ALL, "en_US.UTF-8");  // Non-portable!
```

Và cái đó sẽ chạy. Nhưng nó chỉ portable sang các hệ có đúng cùng
tên đó cho đúng locale đó, và bạn không thể bảo đảm được.

Bằng cách truyền chuỗi rỗng (`""`) làm đối số thứ hai, bạn đang nói
với C, "Này, tự tìm xem locale hiện tại trên hệ này là gì để tôi
khỏi phải nói cho."

[i[`setlocale()` function]>]

## Lấy thiết lập locale cho tiền tệ

[i[Locale-->money]<]

Vì di chuyển mấy tờ giấy xanh hứa hẹn là chìa khoá tới hạnh phúc^["
Hành tinh này có, hay đúng hơn, đã có một vấn đề, đó là: phần lớn
những người sống trên nó không hạnh phúc gần như suốt cả thời gian.
Nhiều giải pháp được đề xuất cho vấn đề này, nhưng phần lớn đều liên
quan đến việc di chuyển các tờ giấy xanh nhỏ, kỳ cục là nhìn chung
chẳng phải mấy tờ giấy xanh nhỏ không hạnh phúc." The Hitchhiker's
Guide to the Galaxy, Douglas Adams], hãy nói về locale cho tiền tệ.
Khi bạn viết code portable, bạn phải biết phải gõ gì cho tiền mặt,
đúng không? Dù nó là "$", "€", "¥", hay "£".

[i[`localeconv()` function]<]

Làm sao viết code đó mà không phát điên? May thay, khi bạn gọi
`setlocale(LC_ALL, "")`, bạn có thể tra mấy cái này bằng một lời gọi
`localeconv()`:

``` {.c}
struct lconv *x = localeconv();
```

Hàm này trả về pointer tới một `struct lconv` được cấp phát tĩnh có
mọi thông tin ngon lành bạn đang tìm.

Đây là các field của `struct lconv` và nghĩa của chúng.

Trước hết, vài quy ước. `_p_` nghĩa là "positive" (dương), `_n_`
nghĩa là "negative" (âm), và `int_` nghĩa là "international" (quốc
tế). Dù nhiều cái có kiểu `char` hoặc `char*`, phần lớn (hoặc các
chuỗi chúng trỏ tới) thật ra được xem như số nguyên^[Nhớ là `char`
chỉ là số nguyên cỡ một byte.].

Trước khi đi tiếp, biết rằng `CHAR_MAX` (từ `<limits.h>`) là giá trị
tối đa lưu được trong một `char`. Và nhiều giá trị `char` dưới đây
dùng nó để cho biết giá trị không có ở locale đó.

|Field|Mô tả|
|-----|-----------|
|`char *mon_decimal_point`|Ký tự dấu thập phân cho tiền, ví dụ `"."`.|
|`char *mon_thousands_sep`|Ký tự phân cách hàng nghìn cho tiền, ví dụ `","`.|
|`char *mon_grouping`|Mô tả cách nhóm cho tiền (xem bên dưới).|
|`char *positive_sign`|Dấu dương cho tiền, ví dụ `"+"` hoặc `""`.|
|`char *negative_sign`|Dấu âm cho tiền, ví dụ `"-"`.|
|`char *currency_symbol`|Ký hiệu tiền tệ, ví dụ `"$"`.|
|`char frac_digits`|Khi in lượng tiền, in bao nhiêu chữ số sau dấu thập phân, ví dụ `2`.|
|`char p_cs_precedes`|`1` nếu `currency_symbol` đứng trước giá trị cho lượng tiền không âm, `0` nếu sau.|
|`char n_cs_precedes`|`1` nếu `currency_symbol` đứng trước giá trị cho lượng tiền âm, `0` nếu sau.|
|`char p_sep_by_space`|Quy định cách ngăn cách `currency symbol` khỏi giá trị cho lượng không âm (xem bên dưới).|
|`char n_sep_by_space`|Quy định cách ngăn cách `currency symbol` khỏi giá trị cho lượng âm (xem bên dưới).|
|`char p_sign_posn`|Quy định vị trí của `positive_sign` cho giá trị không âm.|
|`char n_sign_posn`|Quy định vị trí của `positive_sign` cho giá trị âm.|
|`char *int_curr_symbol`|Ký hiệu tiền tệ quốc tế, ví dụ `"USD "`.|
|`char int_frac_digits`|Giá trị quốc tế cho `frac_digits`.|
|`char int_p_cs_precedes`|Giá trị quốc tế cho `p_cs_precedes`.|
|`char int_n_cs_precedes`|Giá trị quốc tế cho `n_cs_precedes`.|
|`char int_p_sep_by_space`|Giá trị quốc tế cho `p_sep_by_space`.|
|`char int_n_sep_by_space`|Giá trị quốc tế cho `n_sep_by_space`.|
|`char int_p_sign_posn`|Giá trị quốc tế cho `p_sign_posn`.|
|`char int_n_sign_posn`|Giá trị quốc tế cho `n_sign_posn`.|

[i[`localeconv()` function]>]

### Nhóm chữ số cho tiền tệ {#monetary-digit-grouping}

[i[`localeconv()` function-->`mon_grouping`]<]

Được rồi, cái này hơi chập cheng. `mon_grouping` là `char*`, nên bạn
có thể nghĩ nó là chuỗi. Nhưng trong trường hợp này, không, nó thực
ra là mảng các `char`. Nó luôn phải kết thúc bằng `0` hoặc
`CHAR_MAX`.

Các giá trị này mô tả cách nhóm các tập số trong tiền tệ ở phía
_trái_ dấu thập phân (phần nguyên).

Ví dụ, ta có thể có:

``` {.default}
  2   1   0
 --- --- ---
$100,000,000.00
```

Đây là các nhóm ba chữ số. Nhóm 0 (ngay bên trái dấu thập phân) có 3
chữ số. Nhóm 1 (nhóm kế tiếp về trái) có 3 chữ số, và nhóm cuối cũng
có 3.

Vậy ta có thể mô tả các nhóm này, từ phải (dấu thập phân) sang trái
bằng một loạt số nguyên đại diện cho kích thước nhóm:

``` {.default}
3 3 3
```

Và chừng đó sẽ ổn với giá trị tới $100,000,000.

Nhưng lỡ ta có nhiều hơn? Ta có thể cứ thêm `3`...

``` {.default}
3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3
```

nhưng kiểu đó điên rồ. May thay, ta có thể chỉ định `0` để báo rằng
kích thước nhóm trước được lặp lại:

``` {.default}
3 0
```

Nghĩa là lặp mỗi 3. Cái đó sẽ xử được $100, $1,000, $10,000,
$10,000,000, $100,000,000,000, và cứ thế.

Bạn có thể chính thức phát khùng với mấy cái này để chỉ ra vài kiểu
nhóm kỳ cục.

Ví dụ:

``` {.default}
4 3 2 1 0
```

sẽ cho ra:

``` {.default}
$1,0,0,0,0,00,000,0000.00
```

Một giá trị nữa có thể xuất hiện là `CHAR_MAX`. Cái này báo rằng
không còn nhóm nào nữa, và có thể xuất hiện ở bất kỳ chỗ nào trong
mảng, kể cả giá trị đầu.

``` {.default}
3 2 CHAR_MAX
```

sẽ cho ra:

``` {.default}
100000000,00,000.00
```

chẳng hạn.

Và chỉ cần có `CHAR_MAX` ở vị trí đầu của mảng là báo cho bạn biết
không có nhóm nào hết.

[i[`localeconv()` function-->`mon_grouping`]>]

### Dấu phân cách và vị trí dấu

[i[`localeconv()` function-->`sep_by_space`]<]

Mọi biến thể `sep_by_space` xử lý khoảng trắng quanh ký hiệu tiền
tệ. Giá trị hợp lệ là:

|Giá trị|Mô tả|
|:--:|------------|
|`0`|Không có khoảng trắng giữa ký hiệu tiền tệ và giá trị.|
|`1`|Tách ký hiệu tiền tệ (và dấu, nếu có) khỏi giá trị bằng một khoảng trắng.|
|`2`|Tách ký hiệu dấu khỏi ký hiệu tiền tệ (nếu kề nhau) bằng khoảng trắng, ngược lại tách ký hiệu dấu khỏi giá trị bằng khoảng trắng.|

Các biến thể `sign_posn` được quyết định bởi các giá trị sau:

|Giá trị|Mô tả|
|:--:|------------|
|`0`|Bọc giá trị và ký hiệu tiền tệ bằng cặp ngoặc.|
|`1`|Đặt chuỗi dấu trước ký hiệu tiền tệ và giá trị.|
|`2`|Đặt chuỗi dấu sau ký hiệu tiền tệ và giá trị.|
|`3`|Đặt chuỗi dấu ngay trước ký hiệu tiền tệ.|
|`4`|Đặt chuỗi dấu ngay sau ký hiệu tiền tệ.|

[i[`localeconv()` function-->`sep_by_space`]>]
[i[Locale-->money]>]

### Ví dụ giá trị

Khi tôi lấy các giá trị trên hệ của mình, đây là thứ tôi thấy (chuỗi
grouping hiển thị dưới dạng các giá trị byte riêng):

``` {.c}
mon_decimal_point  = "."
mon_thousands_sep  = ","
mon_grouping       = 3 3 0
positive_sign      = ""
negative_sign      = "-"
currency_symbol    = "$"
frac_digits        = 2
p_cs_precedes      = 1
n_cs_precedes      = 1
p_sep_by_space     = 0
n_sep_by_space     = 0
p_sign_posn        = 1
n_sign_posn        = 1
int_curr_symbol    = "USD "
int_frac_digits    = 2
int_p_cs_precedes  = 1
int_n_cs_precedes  = 1
int_p_sep_by_space = 1
int_n_sep_by_space = 1
int_p_sign_posn    = 1
int_n_sign_posn    = 1
```

## Chi tiết localization

Để ý ta đã truyền macro `LC_ALL` cho `setlocale()` ở trên, chuyện
này gợi ý có thể có biến thể khác cho phép bạn chính xác hơn về
_phần nào_ của locale bạn đang đặt.

Hãy xem các giá trị bạn có thể thấy cho mấy cái này:

[i[`setlocale()` function-->`LC_ALL` macro]]
[i[`setlocale()` function-->`LC_COLLATE` macro]]
[i[`setlocale()` function-->`LC_CTYPE` macro]]
[i[`setlocale()` function-->`LC_MONETARY` macro]]
[i[`setlocale()` function-->`LC_NUMERIC` macro]]
[i[`setlocale()` function-->`LC_TIME` macro]]

|Macro|Mô tả|
|----|--------------|
|`LC_ALL`|Đặt tất cả những cái dưới đây về locale đã cho.|
|`LC_COLLATE`|Kiểm soát hành vi của hàm `strcoll()` và `strxfrm()`.|
|`LC_CTYPE`|Kiểm soát hành vi của các hàm xử lý ký tự^[Trừ `isdigit()` và `isxdigit()`.].|
|`LC_MONETARY`|Kiểm soát giá trị mà `localeconv()` trả về.|
|`LC_NUMERIC`|Kiểm soát dấu thập phân cho họ hàm `printf()`.|
|`LC_TIME`|Kiểm soát định dạng thời gian cho các hàm in thời gian và ngày `strftime()` và `wcsftime()`.|

Khá phổ biến thấy [i[`setlocale()` function-->`LC_ALL` macro]]
`LC_ALL` được đặt, nhưng, này, ít nhất bạn có lựa chọn.

Cũng nên nói [i[`setlocale()` function-->`LC_CTYPE`
macro]] `LC_CTYPE` là một trong những cái lớn vì nó gắn với wide
character, một mớ bùng nhùng ta sẽ nói sau.

[i[Locale]>]

<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->
# Con trỏ, khép nép mà run! {#pointers}

> _"How do you get to Carnegie Hall?"_ \
> _"Practice!"_
>
> ---20th-century joke of unknown origin

[i[Pointers]<]Pointer (con trỏ) là một trong những thứ bị sợ nhất
trong ngôn ngữ C. Thật ra, đó chính là thứ khiến ngôn ngữ này có chút
thử thách. Nhưng vì sao?

Vì nó, nói thật, có thể tạo ra dòng điện chạy ngược từ bàn phím lên
rồi _hàn_ tay bạn dính vĩnh viễn tại chỗ, đày bạn cả đời trước bàn
phím với ngôn ngữ từ những năm 70!

Thật à? Ừ, không thật đâu. Tôi chỉ đang dựng sẵn bối cảnh để bạn thành
công.

Tuỳ ngôn ngữ bạn đến từ đâu, có thể bạn đã hiểu khái niệm _reference_
(tham chiếu), nơi một biến tham chiếu tới một đối tượng nào đó.

Chuyện trong C cũng rất giống, chỉ là ta phải nói rõ ràng với C hơn về
việc đang nói tới tham chiếu hay nói tới thứ được tham chiếu.


## Bộ nhớ và biến {#ptmem}

Bộ nhớ máy tính chứa đủ loại dữ liệu, đúng không? Nó chứa `float`,
`int`, và đủ thứ khác. Để bộ nhớ dễ xử lý, mỗi byte trong bộ nhớ được
gán một số nguyên để nhận dạng. Các số này tăng dần khi bạn đi lên
trong bộ nhớ^[Thông thường. Tôi chắc chắn có ngoại lệ đâu đó trong
những hành lang tối tăm của lịch sử điện toán.]. Bạn có thể hình dung
như một đống hộp được đánh số, mỗi hộp chứa một byte^[Byte là một số
gồm không quá 8 chữ số nhị phân, gọi tắt là _bit_. Nghĩa là tính theo
chữ số thập phân, giống thứ bà của bạn từng dùng, nó có thể chứa một
số không dấu từ 0 đến 255, bao gồm cả hai đầu.] dữ liệu. Hoặc như một
mảng lớn mà mỗi phần tử chứa một byte, nếu bạn đến từ ngôn ngữ có
mảng. Con số đại diện cho mỗi hộp được gọi là _address_ (địa
chỉ)[i[Memory address]].

Và không phải kiểu dữ liệu nào cũng chỉ dùng một byte. Ví dụ, `int`
thường bốn byte, `float` cũng vậy, nhưng thật ra tuỳ hệ thống. Bạn có
thể dùng toán tử `sizeof` để xem một kiểu dùng bao nhiêu byte bộ nhớ.

``` {.c}
// %zu is the format specifier for type size_t

printf("an int uses %zu bytes of memory\n", sizeof(int));

// That prints "4" for me, but can vary by system.
```

> **Sự thật vui về bộ nhớ**: Khi bạn có một kiểu dữ liệu (như `int`
> điển hình) dùng nhiều hơn một byte, các byte tạo nên dữ liệu luôn
> nằm liền kề nhau trong bộ nhớ. Đôi khi chúng theo thứ tự bạn nghĩ,
> đôi khi không^[Thứ tự các byte sắp xếp được gọi là _endianness_ của
> số. Các ứng cử viên quen thuộc là _big-endian_ (byte quan trọng
> nhất ở đầu) và _little-endian_ (byte quan trọng nhất ở cuối), hoặc,
> hiếm gặp hơn bây giờ, _mixed-endian_ (byte quan trọng nhất ở đâu
> đó).]. Dù C không bảo đảm thứ tự bộ nhớ cụ thể (tuỳ nền tảng), vẫn
> hoàn toàn có thể viết code theo hướng không phụ thuộc nền tảng, nơi
> bạn thậm chí không phải nghĩ tới cái trật tự byte phiền phức đó.

Vậy _dù sao thì_, nếu ta có thể đi tiếp và làm một hồi trống cùng chút
nhạc dồn dập cho định nghĩa của con trỏ, _con trỏ là một biến chứa địa
chỉ_. Hãy tưởng tượng bản nhạc kinh điển của 2001: A Space Odyssey
ngay lúc này. Ba bum ba bum ba bum BAAAAH!

Được rồi, có lẽ hơi lên gân nhỉ? Con trỏ không có nhiều bí hiểm lắm
đâu. Nó là địa chỉ của dữ liệu. Cũng như biến `int` có thể chứa giá
trị `12`, biến con trỏ có thể chứa địa chỉ của dữ liệu.

Nghĩa là các thứ sau đây cùng mang một nghĩa, tức là một con số biểu
diễn một điểm trong bộ nhớ:

* Chỉ số vào bộ nhớ (nếu bạn nghĩ bộ nhớ như một mảng lớn)
* Địa chỉ (Address)
* Vị trí (Location)

Tôi sẽ dùng lẫn lộn. Và đúng, tôi đã quăng _location_ vào đó, vì không
bao giờ là đủ từ đồng nghĩa cả.

Và biến con trỏ giữ con số địa chỉ đó. Cũng như biến `float` có thể
chứa `3.14159`.

Hãy tưởng tượng bạn có một xấp giấy nhớ Post-it® được đánh số thứ tự
theo địa chỉ. (Cái đầu tiên ở chỉ số `0`, cái kế ở chỉ số `1`, và cứ
thế.)

Ngoài con số biểu diễn vị trí, bạn cũng có thể viết lên mỗi tờ một số
khác tuỳ thích. Có thể là số chó bạn có. Hoặc số mặt trăng quanh sao
Hoả...

...Hoặc, _đó có thể là chỉ số của một tờ Post-it khác!_

Nếu bạn đã viết số chó bạn có, đó chỉ là một biến bình thường. Nhưng
nếu bạn viết chỉ số của một tờ Post-it khác, _đó là một con trỏ_. Nó
trỏ tới tờ giấy kia!

Một phép tương tự khác có thể là với địa chỉ nhà. Bạn có thể có một
căn nhà với những đặc tính nhất định, sân vườn, mái kim loại, tấm pin
mặt trời, v.v. Hoặc bạn có thể có địa chỉ của căn nhà đó. Địa chỉ
không phải là căn nhà. Một bên là nguyên căn nhà, bên kia chỉ là vài
dòng chữ. Nhưng địa chỉ của căn nhà là một _con trỏ_ tới căn nhà đó.
Nó không phải căn nhà, nhưng nó nói cho bạn biết tìm căn nhà ở đâu.

Và ta có thể làm điều tương tự trong máy tính với dữ liệu. Bạn có thể
có một biến dữ liệu chứa giá trị nào đó. Và giá trị đó nằm trong bộ
nhớ ở một địa chỉ nào đó. Và bạn có thể có một _biến con trỏ_ khác
chứa địa chỉ của biến dữ liệu đó.

Nó không phải chính biến dữ liệu, nhưng, giống như địa chỉ nhà, nó nói
cho ta biết tìm biến đó ở đâu.

Khi có được điều đó, ta nói ta có một "con trỏ tới" mẩu dữ liệu đó.
Và ta có thể đi theo con trỏ để truy cập đến bản thân dữ liệu.

(Tuy đến giờ trông chưa có vẻ đặc biệt hữu ích, tất cả sẽ trở nên
không thể thiếu khi dùng với lời gọi hàm. Chịu khó với tôi đến khi tới
chỗ đó.)

Giờ nếu ta có một `int`, và ta muốn một con trỏ tới nó, thứ ta muốn là
cách nào đó để lấy địa chỉ của `int` đó, đúng không? Xét cho cùng, con
trỏ chỉ giữ _địa chỉ của_ dữ liệu. Bạn đoán xem ta dùng toán tử nào để
tìm _địa chỉ của_ `int`?

[i[`&` address-of operator]<]Ồ, với một bất ngờ kinh hoàng mà chắc
hẳn gây sốc tới bạn, người đọc dịu dàng, ta dùng toán tử `address-of`
(hoá ra là dấu và: "`&`") để tìm địa chỉ của dữ liệu. Ampersand (dấu
và).

Ví dụ nhanh, ta sẽ giới thiệu một _format specifier_ mới cho
`printf()` để bạn có thể in một con trỏ. Bạn đã biết `%d` in số nguyên
thập phân đúng không? Thì `%p`[i[`printf()` function-->with
pointers]] in một con trỏ. Giờ, con trỏ này sẽ trông như một con số
rác (và có thể được in ở dạng hexadecimal^[Tức cơ số 16 với các chữ số
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, và F.] thay vì thập
phân), nhưng nó chỉ là chỉ số vào bộ nhớ nơi dữ liệu được lưu. (Hoặc
chỉ số vào bộ nhớ nơi byte đầu tiên của dữ liệu được lưu, nếu dữ liệu
gồm nhiều byte.) Trong hầu như mọi tình huống, kể cả trường hợp này,
giá trị thực tế của con số được in là không quan trọng với bạn, và
tôi đưa ra đây chỉ để minh hoạ toán tử `address-of`.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i = 10;

    printf("The value of i is %d\n", i);
    printf("And its address is %p\n", (void *)&i);
}
```

> **Đoạn code trên có một _cast_** (ép kiểu) nơi ta cưỡng ép kiểu của
> biểu thức `&i` thành kiểu `void*`. Đây là để ngăn trình biên dịch
> văng ra cảnh báo. Tất cả thứ này đều là chuyện ta chưa nói tới, nên
> cứ kệ `(void*)` trong đoạn code trên và giả vờ nó không có ở đó.

Trên máy tính của tôi, đoạn này in ra:

``` {.default}
The value of i is 10
And its address is 0x7ffddf7072a4
```
[i[`&` address-of operator]>]

Nếu bạn tò mò, con số hexadecimal đó là 140.727.326.896.068 trong
thập phân (cơ số 10 y như bà của bạn từng dùng). Đó là chỉ số vào bộ
nhớ nơi dữ liệu của biến `i` được lưu. Đó là địa chỉ của `i`. Đó là vị
trí của `i`. Đó là con trỏ tới `i`.

> **Khoan, bạn có 140 terabyte RAM à?** Có! Bạn không có à? Nhưng tôi
> đang bốc phét thôi; dĩ nhiên tôi không có (khoảng 2024). Máy tính
> hiện đại dùng một công nghệ kỳ diệu gọi là [flw[virtual
> memory|Virtual_memory]] (bộ nhớ ảo) khiến các process nghĩ rằng nó
> có toàn bộ không gian bộ nhớ của máy tính cho riêng nó, bất kể bao
> nhiêu RAM vật lý thực sự hậu thuẫn. Nên dù địa chỉ là con số khổng
> lồ đó, nó được hệ thống bộ nhớ ảo của CPU ánh xạ về một địa chỉ bộ
> nhớ vật lý thấp hơn. Máy tính cụ thể này có 16 GB RAM (lại nữa,
> khoảng 2024, nhưng tôi đang chạy Linux, nên vậy là dư dả). Terabyte
> RAM à? Tôi là giáo viên, không phải tỷ phú dot-com. Chẳng có gì
> trong mấy chuyện này mà ai trong chúng ta cần lo cả, trừ phần tôi
> không phải giàu đến khủng khiếp.

Nó là con trỏ vì nó cho bạn biết `i` ở đâu trong bộ nhớ. Như địa chỉ
nhà viết trên mẩu giấy cho bạn biết có thể tìm căn nhà nào đó ở đâu,
con số này chỉ cho ta biết chỗ nào trong bộ nhớ ta có thể tìm thấy
giá trị của `i`. Nó trỏ tới `i`.

Lại nữa, thường thì ta không thực sự quan tâm con số địa chỉ chính
xác là gì. Ta chỉ quan tâm nó là con trỏ tới `i`.

## Kiểu con trỏ {#pttypes}

[i[Pointer types]<]Được rồi... tất cả thế này ổn thôi. Giờ bạn có thể
thành công lấy địa chỉ của biến và in ra màn hình. Có một chút gì đó
cho bản CV đấy nhỉ? Đây là lúc bạn tóm cổ tôi và lịch sự hỏi con trỏ
rốt cuộc có ích cái quái gì.

Câu hỏi tuyệt vời, và ta sẽ đến đó ngay sau các thông điệp từ nhà tài
trợ.

```
ACME ROBOTIC HOUSING UNIT CLEANING SERVICES. YOUR HOMESTEAD WILL BE
DRAMATICALLY IMPROVED OR YOU WILL BE TERMINATED. MESSAGE ENDS.
```

Chào mừng trở lại với một kỳ nữa của Beej's Guide. Lần gặp trước ta
đang nói về cách tận dụng con trỏ. Tốt, thứ ta sẽ làm là lưu một con
trỏ vào biến để có thể dùng sau. Bạn có thể nhận ra _kiểu con trỏ_ vì
có dấu sao (`*`) đứng trước tên biến và sau kiểu của nó:

``` {.c .numberLines}
int main(void)
{
    int i;  // i's type is "int"
    int *p; // p's type is "pointer to an int", or "int-pointer"
}
```

Này, vậy ta có một biến kiểu con trỏ, và nó có thể trỏ tới các `int`
khác. Tức là, nó có thể chứa địa chỉ của các `int` khác. Ta biết nó
trỏ tới `int`, vì kiểu của nó là `int*` (đọc là "int-pointer").

Khi bạn gán vào một biến con trỏ, kiểu của vế phải phép gán phải
trùng với kiểu biến con trỏ. May thay, khi bạn lấy `address-of` một
biến, kiểu kết quả là con trỏ tới kiểu biến đó, nên các phép gán kiểu
sau đây là hoàn hảo:

``` {.c}
int i;
int *p;  // p is a pointer, but is uninitialized and points to garbage

p = &i;  // p is assigned the address of i--p now "points to" i
```

Bên trái phép gán, ta có một biến kiểu pointer-to-`int` (`int*`), và
bên phải là biểu thức kiểu pointer-to-`int` vì `i` là `int` (bởi
address-of một `int` cho bạn một con trỏ tới `int`). Địa chỉ của một
thứ có thể được lưu trong một con trỏ tới thứ đó.

Hiểu chứ? Tôi biết vẫn chưa hợp lý lắm vì bạn chưa thấy công dụng thực
tế của biến con trỏ, nhưng ta đang đi từng bước nhỏ để không ai lạc.
Giờ ta sẽ giới thiệu toán tử nghịch-address-of. Nó hơi giống address-of
trong Thế giới Bizarro vậy.[i[Pointer types]>]

## Dereference {#deref}

[i[Dereferencing]<]Biến con trỏ có thể được xem là _refer_ (nhắc tới)
một biến khác bằng cách trỏ tới nó. Hiếm khi bạn nghe dân C nói về
"refer" hay "references" đâu, nhưng tôi nhắc tới để cái tên của toán
tử này có chút ý nghĩa hơn.

Khi bạn có con trỏ tới một biến (đại khái "một reference tới một
biến"), bạn có thể dùng biến gốc thông qua con trỏ bằng cách
_dereference_ con trỏ. (Có thể nghĩ như "de-pointering" con trỏ, nhưng
không ai từng nói "de-pointering" cả.)

Trở lại phép so sánh, việc này hơi giống nhìn địa chỉ nhà rồi đi đến
căn nhà đó.

Giờ, "truy cập đến biến gốc" nghĩa là gì? Nếu bạn có biến tên `i`, và
có con trỏ tới `i` tên `p`, bạn có thể dùng con trỏ `p` đã được
dereference _y hệt như chính biến `i` gốc_!

[i[`*` indirection operator]<]Bạn gần như có đủ kiến thức để xem ví
dụ. Mẩu cuối cùng bạn cần biết thật ra là: toán tử dereference là
gì? Nó thật ra được gọi là _indirection operator_ (toán tử gián
tiếp), vì bạn đang truy cập giá trị gián tiếp qua con trỏ. Và nó là
dấu sao, lần nữa: `*`. Giờ, đừng nhầm lẫn nó với dấu sao bạn đã dùng
lúc khai báo con trỏ, ở trước. Cùng là một ký tự, nhưng có nghĩa khác
nhau ở các bối cảnh khác nhau^[Chưa hết! Nó còn dùng trong
`/*comments*/` và trong phép nhân, và trong function prototype với
variable length array! Tất cả cùng là `*`, nhưng bối cảnh cho nó
nghĩa khác nhau.].

Đây là một ví dụ đầy đủ:

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i;
    int *p;  // this is NOT a dereference--this is a type "int*"

    p = &i;  // p now points to i, p holds address of i

    i = 10;  // i is now 10
    *p = 20; // the thing p points to (namely i!) is now 20!!

    printf("i is %d\n", i);   // prints "20"
    printf("i is %d\n", *p);  // "20"! dereference-p is the same as i!
}
```

Nhớ rằng `p` giữ địa chỉ của `i`, như bạn thấy ở chỗ ta gán cho `p` ở
dòng 8. Cái toán tử indirection làm là bảo máy tính _dùng đối tượng mà
con trỏ trỏ tới_ thay vì dùng chính con trỏ. Bằng cách này, ta đã biến
`*p` thành một dạng bí danh cho `i`.[i[Dereferencing]>][i[`*` indirection operator]>]

Tuyệt, nhưng _sao chứ_? Làm tất cả thứ này để làm gì?

## Truyền con trỏ làm đối số {#ptpass}

[i[Pointers-->as arguments]<]Đến giờ, bạn đang nghĩ là mình có khá
nhiều kiến thức về con trỏ nhưng không một chút ứng dụng, đúng chứ?
Kiểu, `*p` có ích gì nếu bạn cứ việc viết `i`?

Bạn của tôi ơi, sức mạnh thực sự của con trỏ xuất hiện khi bạn bắt
đầu truyền chúng cho hàm. Sao lại quan trọng? Bạn có thể nhớ từ trước
rằng có thể truyền đủ loại đối số cho hàm, chúng sẽ được ngoan ngoãn
sao chép vào parameter, và rồi bạn có thể thao tác các bản sao cục bộ
của biến bên trong hàm, và rồi có thể trả về một giá trị duy nhất.

Lỡ bạn muốn đem về nhiều hơn một mẩu dữ liệu từ hàm thì sao? Ý là bạn
chỉ có thể trả về một thứ duy nhất, đúng không? Lỡ tôi trả lời câu
hỏi đó bằng một câu hỏi khác? ...Ờ, hai câu hỏi?

Chuyện gì xảy ra khi bạn truyền một con trỏ làm đối số cho hàm? Một
bản sao của con trỏ có được đặt vào parameter tương ứng không? _Tất
nhiên có luôn._ Nhớ hồi trước tôi huyên thuyên về chuyện _MỌI ĐỐI SỐ_
được sao chép vào parameter và hàm dùng một bản sao của đối số không?
Cũng như vậy ở đây. Hàm sẽ nhận một bản sao của con trỏ.

Nhưng, và đây là phần thông minh: ta đã dựng sẵn con trỏ để trỏ tới
một biến... và rồi hàm có thể dereference bản sao con trỏ của nó để
quay lại biến gốc! Hàm không thấy được chính biến đó, nhưng chắc chắn
nó có thể dereference một con trỏ tới biến đó!

Việc này tương tự như viết một địa chỉ nhà lên mảnh giấy, rồi chép nó
sang một mảnh giấy khác. Giờ bạn có _hai_ con trỏ tới căn nhà đó, và
cả hai đều dẫn tốt như nhau tới chính căn nhà.

Trong trường hợp lời gọi hàm, một trong các bản sao được lưu trong một
biến con trỏ ở scope gọi hàm, bản kia được lưu trong một biến con trỏ
là parameter của hàm.

Ví dụ nào! Hãy quay lại hàm `increment()` cũ của ta, nhưng lần này làm
sao cho nó thực sự tăng giá trị ở phía người gọi.

``` {.c .numberLines}
#include <stdio.h>

void increment(int *p)  // note that it accepts a pointer to an int
{
    *p = *p + 1;        // add one to the thing p points to
}

int main(void)
{
    int i = 10;
    int *j = &i;  // note the address-of; turns it into a pointer to i

    printf("i is %d\n", i);        // prints "10"
    printf("i is also %d\n", *j);  // prints "10"

    increment(j);                  // j is an int*--to i

    printf("i is %d\n", i);        // prints "11"!
}
```

Được rồi! Có vài thứ cần để ý ở đây... không kém phần quan trọng là
hàm `increment()` nhận `int*` làm đối số. Ta truyền cho nó một `int*`
trong lời gọi bằng cách đổi biến `int` `i` thành `int*` qua toán tử
`address-of`. (Nhớ nhé, con trỏ giữ địa chỉ, nên ta tạo con trỏ tới
biến bằng cách cho chúng chạy qua toán tử `address-of`.)

Hàm `increment()` nhận một bản sao của con trỏ. Cả con trỏ gốc `j`
(trong `main()`) lẫn bản sao của nó `p` (parameter trong
`increment()`) đều trỏ tới cùng một địa chỉ, cụ thể là địa chỉ chứa
giá trị `i`. (Lại nữa, theo phép so sánh, như hai mảnh giấy cùng ghi
một địa chỉ nhà.) Dereference bất kỳ cái nào cũng cho phép bạn sửa
biến gốc `i`! Hàm có thể sửa một biến ở scope khác! Tuyệt vời!

Ví dụ trên thường được viết gọn hơn trong lời gọi bằng cách dùng
address-of ngay trong danh sách đối số:

``` {.c}
printf("i is %d\n", i);  // prints "10"
increment(&i);
printf("i is %d\n", i);  // prints "11"!
```

Quy tắc chung, nếu bạn muốn hàm sửa thứ bạn đang truyền vào sao cho
bạn thấy kết quả, bạn sẽ phải truyền một con trỏ tới thứ đó.

## Con trỏ `NULL`

[i[`NULL` pointer]<]Bất kỳ biến con trỏ kiểu nào cũng có thể được gán
một giá trị đặc biệt là `NULL`. Điều này cho biết con trỏ không trỏ
tới thứ gì.

``` {.c}
int *p;

p = NULL;
```

Vì nó không trỏ tới giá trị nào, dereference nó là hành vi không xác
định, và có lẽ sẽ dẫn đến crash:

``` {.c}
int *p = NULL;

*p = 12;  // CRASH or SOMETHING PROBABLY BAD. BEST AVOIDED.
```

Dù bị gọi là [flw[lỗi lầm tỷ đô, bởi chính người tạo ra
nó|Null_pointer#History]], con trỏ `NULL` là một
[flw[sentinel value|Sentinel_value]] tốt và là chỉ dấu chung cho thấy
một con trỏ chưa được khởi tạo.

(Dĩ nhiên, cũng giống các biến khác, con trỏ trỏ tới rác trừ khi bạn
gán rõ ràng cho nó trỏ tới một địa chỉ hoặc `NULL`.)
[i[`NULL` pointer]>]

## Một ghi chú về khai báo con trỏ

[i[Pointers-->declarations]<]Cú pháp khai báo con trỏ có thể hơi kỳ
quặc. Hãy xem ví dụ này:

``` {.c}
int a;
int b;
```

Ta có thể gộp thành một dòng chứ?

``` {.c}
int a, b;  // Same thing
```

Nên `a` và `b` đều là `int`. Không vấn đề.

Nhưng còn cái này?

``` {.c}
int a;
int *p;
```

Có gộp được thành một dòng không? Được. Nhưng `*` đặt ở đâu?

Quy tắc là `*` đứng trước bất kỳ biến nào là kiểu con trỏ. Tức là, `*`
_không phải_ là một phần của `int` trong ví dụ này, nó là một phần của
biến `p`.

Với điều đó trong đầu, ta có thể viết:

``` {.c}
int a, *p;  // Same thing
```

Quan trọng cần lưu ý rằng dòng sau đây _không_ khai báo hai con trỏ:

``` {.c}
int *p, q;  // p is a pointer to an int; q is just an int.
```

Trông càng gian trá hơn nếu lập trình viên viết dòng (hợp lệ) sau, có
chức năng giống hệt dòng trên.

``` {.c}
int* p, q;  // p is a pointer to an int; q is just an int.
```

Giờ xem cái này và xác định biến nào là con trỏ, biến nào không:

``` {.c}
int *a, b, c, *d, e, *f, g, h, *i;
```

Tôi sẽ để đáp án ở footnote^[Các biến kiểu con trỏ là `a`, `d`, `f`,
và `i`, vì đó là những biến có `*` đứng trước.].[i[Pointers-->declarations]>]

## `sizeof` và con trỏ

[i[Pointers-->with `sizeof`]<]Chỉ một chút cú pháp ở đây có thể gây
bối rối, đôi khi bạn sẽ gặp.

Nhớ rằng `sizeof` hoạt động trên _kiểu_ của biểu thức.

``` {.c}
int *p;

// Prints size of an 'int'
printf("%zu\n", sizeof(int));

// p is type 'int *', so prints size of 'int*'
printf("%zu\n", sizeof p);

// *p is type 'int', so prints size of 'int'
printf("%zu\n", sizeof *p);
```

Bạn có thể gặp code ngoài đời với `sizeof` cuối cùng đó. Chỉ cần nhớ
`sizeof` nói về kiểu của biểu thức, chứ không phải về bản thân các
biến trong biểu thức.[i[Pointers-->with
`sizeof`]>][i[Pointers]>]

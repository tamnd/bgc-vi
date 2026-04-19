<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Phép bitwise

[i[Bitwise operations]<]

Mấy phép số này cho phép bạn thao tác từng bit trong biến, cũng hợp
vì C là ngôn ngữ mức thấp^[Không phải các ngôn ngữ khác không làm
vậy, chúng có làm. Thú vị là bao nhiêu ngôn ngữ hiện đại dùng cùng
toán tử bitwise như C.].

Nếu bạn chưa quen với phép bitwise, [flw[Wikipedia có bài viết về
bitwise khá tốt|Bitwise_operation]].

## Bitwise AND, OR, XOR và NOT

Với từng phép này, [các quy đổi số học thông
thường](#usual-arithmetic-conversions) diễn ra trên toán hạng (trong
trường hợp này phải là kiểu số nguyên), rồi phép bitwise tương ứng
được thực hiện.

[i[`&` bitwise AND]]
[i[`|` bitwise OR]]
[i[`^` bitwise XOR]]
[i[`~` bitwise NOT]]

|Phép|Toán tử|Ví dụ|
|-|:-:|-|
|AND|`&`|`a = b & c`|
|OR|`|`|`a = b | c`|
|XOR|`^`|`a = b ^ c`|
|NOT|`~`|`a = ~c`|

Lưu ý chúng giống toán tử Boolean `&&` và `||`.

Mấy cái này có biến thể gán tắt tương tự `+=` và `-=`:

[i[`&=` assignment]]
[i[`|=` assignment]]
[i[`^=` assignment]]

|Toán tử|Ví dụ|Tương đương đầy đủ|
|-|-|-|
|`&=`|`a &= c`|`a = a & c`|
|`|=`|`a |= c`|`a = a | c`|
|`^=`|`a ^= c`|`a = a ^ c`|

## Bitwise shift

Với mấy cái này, [i[Integer promotions]] [integer
promotion](#integer-promotions) diễn ra trên từng toán hạng (phải là
kiểu số nguyên) rồi bitwise shift được thực hiện. Kiểu của kết quả
là kiểu của toán hạng trái sau khi promote.

Bit mới được lấp bằng 0, với một ngoại lệ có thể có được nói trong
phần hành vi implementation-defined, bên dưới.

[i[`<<` shift left]]
[i[`>>` shift right]]

|Phép|Toán tử|Ví dụ|
|-|:-:|-|
|Shift trái|`<<`|`a = b << c`|
|Shift phải|`>>`|`a = b >> c`|

Cũng có dạng viết tắt tương tự cho shift:

[i[`>>=` assignment]]
[i[`<<=` assignment]]

|Toán tử|Ví dụ|Tương đương đầy đủ|
|-|-|-|
|`>>=`|`a >>= c`|`a = a >> c`|
|`<<=`|`a <<= c`|`a = a << c`|

Coi chừng hành vi không xác định: không shift số âm, và không shift
lớn hơn kích thước của toán hạng trái sau khi promote.

Cũng coi chừng hành vi implementation-defined: nếu bạn shift phải
một số âm, kết quả là implementation-defined. (Shift phải một `int`
có dấu thì hoàn toàn ổn, chỉ cần nó là số dương.)

[i[Bitwise operations]>]

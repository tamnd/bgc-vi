<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# File Input/Output

[i[File I/O]<]
Ta đã thấy vài ví dụ về I/O với `printf()` để làm I/O ở console.

Nhưng ở chương này ta sẽ đẩy các khái niệm đó đi xa hơn một chút.

## Kiểu dữ liệu `FILE*`

[i[`FILE*` type]<]
Khi làm bất kỳ dạng I/O nào trong C, ta làm thông qua một mẩu dữ liệu
dưới dạng kiểu `FILE*`. `FILE*` này giữ mọi thông tin cần để giao tiếp
với hệ thống I/O về file bạn đang mở, vị trí hiện tại trong file, v.v.

Đặc tả gọi những thứ này là _stream_, tức một dòng dữ liệu chảy ra từ
file hoặc từ bất kỳ nguồn nào. Tôi sẽ dùng lẫn "file" với "stream",
nhưng thực sự bạn nên coi "file" là một trường hợp đặc biệt của
"stream". Có những cách khác để đẩy dữ liệu vào chương trình ngoài
chuyện đọc từ file.

Chút nữa ta sẽ xem cách đi từ một tên file tới một `FILE*` đã mở, nhưng
trước hết tôi muốn nhắc đến ba stream đã được mở sẵn và có thể dùng
ngay.

[i[`stdin` standard input]<]
[i[`stdout` standard output]<]
[i[`stderr` standard error]<]

|Tên `FILE*`|Mô tả|
|-|-|
|`stdin`|Standard Input (đầu vào chuẩn), mặc định thường là bàn phím|
|`stdout`|Standard Output (đầu ra chuẩn), mặc định thường là màn hình|
|`stderr`|Standard Error (lỗi chuẩn), mặc định cũng thường là màn hình|

Hoá ra ta đã dùng chúng ngầm suốt rồi. Chẳng hạn, hai lời gọi này là
một:

``` {.c}
printf("Hello, world!\n");
fprintf(stdout, "Hello, world!\n");  // printf to a file
```

Nhưng chuyện đó để sau.

Bạn cũng sẽ nhận ra rằng cả `stdout` và `stderr` đều ra màn hình. Nhìn
qua tưởng như sơ suất hay trùng lặp, nhưng thực ra không phải. Các hệ
điều hành điển hình cho phép bạn _redirect_ (chuyển hướng) đầu ra của
bất kỳ cái nào trong hai vào các file khác nhau, và việc tách thông
báo lỗi khỏi đầu ra thường có thể rất tiện.

Ví dụ, trong shell POSIX (như sh, ksh, bash, zsh, v.v.) trên hệ thống
kiểu Unix, ta có thể chạy chương trình và đẩy đầu ra không-lỗi
(`stdout`) vào một file, còn đầu ra lỗi (`stderr`) vào file khác.

``` {.zsh}
./foo > output.txt 2> errors.txt   # This command is Unix-specific
```

Vì lý do đó, bạn nên gửi các thông báo lỗi nghiêm trọng ra `stderr`
chứ đừng ra `stdout`.
[i[`stdin` standard input]>]
[i[`stdout` standard output]>]
[i[`stderr` standard error]>]

Cách làm sẽ nói sau.
[i[`FILE*` type]<]

## Đọc file văn bản

[i[File I/O-->text files, reading]<]
Stream được phân loại đại khái theo hai cách: _text_ (văn bản) và
_binary_ (nhị phân).

Stream văn bản được phép dịch dữ liệu khá nhiều, đáng chú ý nhất là
dịch các newline sang các biểu diễn khác nhau^[Trước kia ta có ba loại
newline dùng rộng rãi: Carriage Return (CR, dùng trên Mac đời cũ),
Linefeed (LF, dùng trên hệ Unix), và Carriage Return/Linefeed (CRLF,
dùng trên hệ Windows). May mắn là sự xuất hiện của OS X, vốn dựa trên
Unix, đã rút con số xuống còn hai.]. File văn bản về mặt logic là một
chuỗi _dòng_ được ngăn bởi newline. Để portable, dữ liệu đầu vào nên
luôn kết thúc bằng một newline.

Nhưng quy tắc chung là nếu bạn có thể chỉnh sửa file đó trong một
trình soạn thảo văn bản thông thường thì đó là file văn bản. Ngược
lại là nhị phân. Nhị phân thì để sau.

Vậy, vào việc, làm sao mở một file để đọc và kéo dữ liệu ra?

Tạo một file `hello.txt` chỉ chứa mỗi:

``` {.default}
Hello, world!
```

Rồi viết một chương trình mở file, đọc một ký tự ra, rồi đóng file
khi xong. Kế hoạch thế!

[i[`fopen()` function]<]
[i[`fgetc()` function]<]
[i[`fclose()` function]<]
``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    FILE *fp;                      // Variable to represent open file

    fp = fopen("hello.txt", "r");  // Open file for reading

    int c = fgetc(fp);             // Read a single character
    printf("%c\n", c);             // Print char to stdout

    fclose(fp);                    // Close the file when done
}
```

Thấy đấy, khi mở file với `fopen()`, nó trả `FILE*` về cho ta để dùng
sau.

(Tôi bỏ qua để gọn, nhưng `fopen()` sẽ trả về `NULL` nếu có gì trục
trặc, như không tìm thấy file, nên bạn thật sự phải check lỗi cho nó!)

Cũng chú ý chuỗi `"r"` ta truyền vào, nghĩa là "mở một stream văn bản
để đọc". (Có nhiều chuỗi khác nhau có thể truyền cho `fopen()` với ý
nghĩa khác, như ghi, append, v.v.)
[i[`fopen()` function]>]

Sau đó ta dùng hàm `fgetc()` để lấy một ký tự từ stream. Có thể bạn
thắc mắc sao tôi khai báo `c` là `int` chứ không phải `char`, giữ câu
hỏi đó lại nhé!
[i[`fgetc()` function]>]

Cuối cùng ta đóng stream khi xong. Mọi stream đều được đóng tự động
khi chương trình thoát, nhưng đóng file thủ công khi không còn dùng
nữa là phong cách tốt và cẩn thận.
[i[`fclose()` function]>]

[i[`FILE*` type]]`FILE*` theo dõi vị trí của ta trong file. Nên các
lời gọi `fgetc()` tiếp theo sẽ lấy ký tự kế, rồi ký tự kế nữa, cho tới
hết.

Nhưng nghe khổ sở quá. Xem có cách nào dễ hơn không.
[i[File I/O-->text files, reading]>]

## Hết file: `EOF`

[i[`EOF` end of file]<]
Có một ký tự đặc biệt được định nghĩa dạng macro: `EOF`. Đây là thứ
`fgetc()` sẽ trả về khi đã tới cuối file và bạn cố đọc thêm một ký tự
nữa.

Giờ kể một Fun Fact™. Hoá ra `EOF` là lý do `fgetc()` và các hàm kiểu
nó trả về `int` thay vì `char`. `EOF` không phải ký tự đúng nghĩa, và
giá trị của nó nhiều khả năng nằm ngoài miền của `char`. Vì `fgetc()`
phải có khả năng trả về bất kỳ byte nào **và** `EOF`, nó cần một kiểu
rộng hơn để chứa nhiều giá trị hơn. Nên là `int`. Nhưng trừ khi bạn
đang so sánh giá trị trả về với `EOF`, trong thâm tâm bạn có thể yên
tâm rằng đó là một `char`.

Ổn! Quay lại thực tế! Ta có thể dùng cái này để đọc cả file trong một
vòng lặp.

[i[`fopen()` function]<]
[i[`fgetc()` function]<]
[i[`fclose()` function]<]
``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    FILE *fp;
    int c;

    fp = fopen("hello.txt", "r");

    while ((c = fgetc(fp)) != EOF)
        printf("%c", c);

    fclose(fp);
}
```
[i[`fopen()` function]>]
[i[`fclose()` function]>]

(Nếu dòng 10 lạ quá, cứ mổ nó ra bắt đầu từ ngoặc lồng trong cùng. Việc
đầu tiên là gán kết quả của `fgetc()` vào `c`, _rồi_ mới so sánh _cái
đó_ với `EOF`. Ta vừa nhét tất cả vào một dòng. Đọc có vẻ khó, nhưng
nghiền ngẫm đi, đây là C idiomatic đấy.)
[i[`fgetc()` function]>]

Chạy thử, ta thấy:

``` {.default}
Hello, world!
```

Nhưng vẫn đang xử lý từng ký tự một, mà nhiều file văn bản hợp lý hơn
khi nhìn theo dòng. Chuyển sang cái đó.
[i[`EOF` end of file]>]

### Đọc từng dòng một

[i[File I/O-->line by line]<]
Vậy làm sao lấy nguyên một dòng cùng lúc? [i[`fgets()`
function]<]`fgets()` giải cứu! Tham số gồm một con trỏ tới buffer
`char` để chứa byte, số byte tối đa được đọc, và một `FILE*` để đọc
từ đó. Nó trả về `NULL` khi hết file hoặc lỗi. `fgets()` còn tử tế
đến mức NUL-terminate chuỗi khi xong^[Nếu buffer không đủ lớn để đọc
hết một dòng, nó sẽ dừng giữa dòng, và lời gọi `fgets()` kế tiếp sẽ
tiếp tục đọc phần còn lại của dòng đó.].[i[`fgets()` function]>]

Làm một vòng lặp tương tự trước, nhưng lần này với file nhiều dòng và
đọc từng dòng một.

Đây là file `quote.txt`:

``` {.default}
A wise man can learn more from
a foolish question than a fool
can learn from a wise answer.
                  --Bruce Lee
```

Và đây là đoạn code đọc file đó từng dòng một và in số dòng trước mỗi
dòng:

[i[`fgets()` function]<]
``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    FILE *fp;
    char s[1024];  // Big enough for any line this program will encounter
    int linecount = 0;

    fp = fopen("quote.txt", "r");

    while (fgets(s, sizeof s, fp) != NULL) 
        printf("%d: %s", ++linecount, s);

    fclose(fp);
}
```
[i[`fgets()` function]>]

Cho ra:

``` {.default}
1: A wise man can learn more from
2: a foolish question than a fool
3: can learn from a wise answer.
4:                   --Bruce Lee
```
[i[File I/O-->line by line]>]

## Đầu vào có định dạng

[i[File I/O-->formatted input]<]
Bạn biết cách lấy đầu ra có định dạng bằng `printf()` chứ (và cũng vậy
với `fprintf()` ta sẽ thấy ở dưới)?

[i[`fscanf()` function]<]
Bạn có thể làm y như thế với `fscanf()`.

> Trước khi bắt đầu, xin lưu ý rằng dùng các hàm kiểu `scanf()` có thể
> nguy hiểm với đầu vào không đáng tin. Nếu không chỉ định độ rộng
> trường cho `%s`, bạn có thể tràn buffer. Tệ hơn, chuyển đổi số
> không hợp lệ sẽ dẫn tới undefined behavior. Cách an toàn với đầu vào
> không đáng tin là dùng `%s` kèm độ rộng trường, rồi dùng các hàm
> như [i[`strtol` function]] `strtol()` hay [i[`strtod` function]]
> `strtod()` để chuyển đổi.

Ta có một file với một dãy bản ghi dữ liệu. Trường hợp này là cá voi,
với tên, chiều dài tính bằng mét, và cân nặng tính bằng tấn.
`whales.txt`:

``` {.default}
blue 29.9 173
right 20.7 135
gray 14.9 41
humpback 16.0 30
```

Phải, ta có thể đọc bằng [i[`fgets()` function]]`fgets()` rồi parse
chuỗi bằng `sscanf()` (và cách đó chống đỡ tốt hơn với file hỏng),
nhưng lần này cứ dùng `fscanf()` kéo thẳng vào.

Hàm `fscanf()` bỏ qua whitespace ở đầu khi đọc, và trả về `EOF` khi
hết file hoặc lỗi.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    FILE *fp;
    char name[1024];  // Big enough for any line this program will encounter
    float length;
    int mass;

    fp = fopen("whales.txt", "r");

    while (fscanf(fp, "%s %f %d", name, &length, &mass) != EOF)
        printf("%s whale, %d tonnes, %.1f meters\n", name, mass, length);

    fclose(fp);
}
```
[i[`fscanf()` function]>]

Cho ra:

``` {.default}
blue whale, 173 tonnes, 29.9 meters
right whale, 135 tonnes, 20.7 meters
gray whale, 41 tonnes, 14.9 meters
humpback whale, 30 tonnes, 16.0 meters
```
[i[File I/O-->formatted input]>]

## Ghi file văn bản

[i[File I/O-->text files, writing]<]
[i[`fputc()` function]<]
[i[`fputs()` function]<]
[i[`fprintf()` function]<]
Giống hệt như cách ta dùng `fgetc()`, `fgets()`, và `fscanf()` để đọc
stream văn bản, ta có thể dùng `fputc()`, `fputs()`, và `fprintf()`
để ghi stream văn bản.

Để làm vậy, ta phải `fopen()` file ở chế độ ghi bằng cách truyền `"w"`
làm đối số thứ hai. Mở một file đang tồn tại ở chế độ `"w"` sẽ lập
tức cắt file đó về 0 byte để ghi đè hoàn toàn.

Ta sẽ ghép một chương trình đơn giản xuất ra file `output.txt` dùng
vài hàm xuất khác nhau.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    FILE *fp;
    int x = 32;

    fp = fopen("output.txt", "w");

    fputc('B', fp);
    fputc('\n', fp);   // newline
    fprintf(fp, "x = %d\n", x);
    fputs("Hello, world!\n", fp);

    fclose(fp);
}
```
[i[`fputc()` function]>]
[i[`fputs()` function]>]
[i[`fprintf()` function]>]

Chương trình này tạo ra file `output.txt` với nội dung:

``` {.default}
B
x = 32
Hello, world!
```

Fun fact: vì `stdout` là một file, bạn có thể thay dòng 8 bằng:

``` {.c}
fp = stdout;
```

và chương trình sẽ xuất ra console thay vì ra file. Thử xem!
[i[File I/O-->text files, writing]>]

## I/O file nhị phân

[i[File I/O-->binary files]<]
Tới giờ ta mới nói file văn bản. Nhưng còn con thú còn lại nhắc hồi
đầu gọi là file _nhị phân_ (binary), hay binary stream.

Chúng hoạt động khá giống file văn bản, chỉ khác ở chỗ hệ thống I/O
không dịch dữ liệu như có thể sẽ làm với file văn bản. Với file nhị
phân, bạn có một dòng byte thô, thế thôi.

Khác biệt lớn khi mở file là phải thêm `"b"` vào mode. Tức là, để đọc
file nhị phân, mở ở mode `"rb"`. Để ghi file, mở ở mode `"wb"`.

Vì là dòng byte, và dòng byte có thể chứa ký tự NUL, mà ký tự NUL là
dấu kết chuỗi trong C, hiếm khi người ta dùng `fprintf()` và đồng bọn
để thao tác file nhị phân.

[i[`fwrite()` function]<]
Thay vào đó hai hàm phổ biến nhất là [i[`fread()` function]]`fread()`
và `fwrite()`. Các hàm này đọc và ghi một số byte chỉ định vào stream.

Demo cho biết, ta sẽ viết mấy chương trình. Một chương trình sẽ ghi
một dãy giá trị byte ra đĩa cùng lúc. Chương trình thứ hai sẽ đọc từng
byte một và in ra^[Thông thường chương trình thứ hai sẽ đọc tất cả
byte cùng lúc, _rồi_ mới in ra trong vòng lặp. Cách đó hiệu quả hơn.
Nhưng ở đây cốt là để demo.].

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    FILE *fp;
    unsigned char bytes[6] = {5, 37, 0, 88, 255, 12};

    fp = fopen("output.bin", "wb");  // wb mode for "write binary"!

    // In the call to fwrite, the arguments are:
    //
    // * Pointer to data to write
    // * Size of each "piece" of data
    // * Count of each "piece" of data
    // * FILE*

    fwrite(bytes, sizeof(char), 6, fp);

    fclose(fp);
}
```
[i[`fwrite()` function]>]

Hai đối số giữa của `fwrite()` trông hơi kỳ. Nhưng đại khái ý ta muốn
nói với hàm là: "Ta có các mục lớn _ngần này_, và muốn ghi _bấy nhiêu_
mục." Tiện lợi nếu bạn có bản ghi có độ dài cố định, và có một mảng
chúng. Bạn chỉ cần bảo kích cỡ một bản ghi và bao nhiêu bản ghi cần
ghi.

Trong ví dụ trên, ta bảo kích cỡ mỗi bản ghi là `char`, và có 6 mục.

Chạy chương trình xong ta có file `output.bin`, nhưng mở nó trong
trình soạn thảo văn bản thì chẳng thấy gì thân thiện! Đó là dữ liệu
nhị phân, không phải văn bản. Và là dữ liệu nhị phân ngẫu nhiên tôi
vừa chế ra ấy chứ!

Nếu đẩy qua chương trình [flw[hex dump|Hex_dump]], ta có thể thấy đầu
ra dưới dạng các byte:

``` {.default}
05 25 00 58 ff 0c
```

> Nhiều hệ Unix có sẵn chương trình tên `hexdump` để làm việc này. Bạn
> có thể dùng như này với cờ `-C` ("canonical") để có đầu ra đẹp:
>
> ``` {.default}
> $ hexdump -C output.bin
> 00000000  05 25 00 58 ff 0c                              |.%.X..|
> ```
>
> <!-- ` -->
>
> `00000000` là offset trong file mà dòng đầu ra này bắt đầu. `05 25
> 00 58 ff 0c` là các giá trị byte (và sẽ dài hơn, tới 16 byte mỗi
> dòng, nếu có nhiều byte hơn trong file). Và bên phải giữa hai ký
> hiệu pipe (`|`) là nỗ lực hết mình của `hexdump` để in ra các ký tự
> ứng với những byte đó. Nó in dấu chấm nếu ký tự không in được.
> Trường hợp này, vì ta chỉ in dữ liệu nhị phân ngẫu nhiên, phần đầu
> ra đó chỉ là rác. Nhưng nếu ta in một chuỗi ASCII ra file, sẽ thấy
> nó ở trong đó.

Và các giá trị hex đó khớp với các giá trị (thập phân) ta đã ghi ra.

Giờ thử đọc lại bằng một chương trình khác. Chương trình này sẽ mở
file để đọc nhị phân (mode `"rb"`) và đọc từng byte một trong vòng
lặp.

[i[`fread()` function]<]
`fread()` có đặc điểm hay ở chỗ trả về số byte đã đọc, hoặc `0` khi
EOF. Nên ta có thể lặp đến khi thấy thế, in số ra trong lúc chạy.

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    FILE *fp;
    unsigned char c;

    fp = fopen("output.bin", "rb"); // rb for "read binary"!

    while (fread(&c, sizeof(char), 1, fp) > 0)
        printf("%d\n", c);

    fclose(fp);
}
```
[i[`fread()` function]>]

Và, chạy nó, ta thấy lại các con số gốc!

``` {.default}
5
37
0
88
255
12
```

Woo hoo!
[i[File I/O-->binary files]>]

### Lưu ý về `struct` và số

[i[File I/O-->with `struct`s]<]
Như đã thấy ở phần `struct`, compiler được tự do thêm padding vào
`struct` theo cách nó thấy hợp. Và các compiler khác nhau có thể làm
khác nhau. Cùng một compiler trên kiến trúc khác nhau có thể làm
khác. Cùng một compiler trên cùng kiến trúc cũng có thể làm khác.

Ý tôi là: không portable nếu bạn chỉ `fwrite()` nguyên một `struct` ra
file khi không biết padding sẽ nằm đâu.
[i[File I/O-->with `struct`s]>]

Fix sao đây? Giữ ý đó lại, ta sẽ xem vài cách làm chuyện này sau khi
ngó qua một vấn đề liên quan khác.

[i[File I/O-->with numeric values]<]
Số!

Hoá ra không phải mọi kiến trúc đều biểu diễn số trong bộ nhớ theo
cùng một cách.

Xem một `fwrite()` đơn giản của một số 2 byte. Ta sẽ viết nó dạng hex
để mỗi byte hiện rõ. Byte có giá trị cao nhất sẽ có giá trị `0x12`,
byte thấp nhất sẽ có giá trị `0x34`.

``` {.c}
unsigned short v = 0x1234;  // Two bytes, 0x12 and 0x34

fwrite(&v, sizeof v, 1, fp);
```

Stream sẽ chứa gì?

Tưởng như phải là `0x12` rồi tới `0x34`, đúng không?

Nhưng nếu tôi chạy cái này trên máy mình và hex dump kết quả, tôi
thấy:

``` {.default}
34 12
```

Đảo ngược rồi! Chuyện gì thế?

Chuyện này liên quan tới cái gọi là
[i[Endianess]][flw[_endianess_|Endianess]] của kiến trúc. Có nơi ghi
byte có giá trị cao trước, có nơi ghi byte có giá trị thấp trước.

Điều này nghĩa là nếu bạn ghi thẳng một số nhiều byte ra từ bộ nhớ,
bạn không thể làm portable được^[Và đây là lý do tôi dùng từng byte
riêng trong các ví dụ `fwrite()` và `fread()` ở trên, khôn đấy chứ.].

Một vấn đề tương tự tồn tại với số dấu phẩy động. Hầu hết hệ thống
dùng cùng format cho số floating point, nhưng vài hệ thì không. Không
đảm bảo gì hết!

[i[File I/O-->with `struct`s]<]
Vậy... làm sao fix hết đống vấn đề này với số và `struct` để ghi dữ
liệu ra một cách portable?

Tóm gọn là [i[Data serialization]]_serialize_ (tuần tự hoá) dữ liệu,
thuật ngữ chung mang nghĩa lấy hết dữ liệu và ghi ra theo một định
dạng bạn kiểm soát, rõ ràng, và lập trình được để hoạt động giống
nhau trên mọi nền tảng.

Như bạn đoán, đây là bài toán đã giải. Có một loạt thư viện
serialization sẵn sàng để dùng, chẳng hạn [flw[_protocol
buffers_|Protocol_buffers]] của Google. Chúng lo mọi chi tiết vặt cho
bạn, và thậm chí cho phép dữ liệu từ chương trình C của bạn tương tác
với các ngôn ngữ khác hỗ trợ cùng phương thức serialization.

Làm ơn một điều cho bản thân và mọi người! Serialize dữ liệu nhị phân
khi ghi ra stream! Sẽ giữ mọi thứ gọn và portable, kể cả khi bạn
chuyển file dữ liệu từ kiến trúc này sang kiến trúc khác.
[i[File I/O-->with `struct`s]>]
[i[File I/O-->with numeric values]>]
[i[File I/O]>]

<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Atomics {#chapter-atomics}

> _"They tried and failed, all of them?"_ \
> _"Oh, no." She shook her head. "They tried and died."_
>
> ---Paul Atreides and The Reverend Mother Gaius Helen Mohiam, _Dune_

[i[Atomic variables]<]

Đây là một trong những khía cạnh thử thách hơn của đa luồng với C.
Nhưng ta sẽ cố gắng thong thả.

Về cơ bản tôi sẽ nói về các cách dùng đơn giản hơn của biến atomic,
chúng là gì, và hoạt động thế nào, vân vân. Và tôi sẽ nhắc tới vài
con đường điên-rồ-phức-tạp có sẵn cho bạn.

Nhưng tôi sẽ không đi theo những con đường đó. Không chỉ vì tôi
hiếm đủ khả năng để viết về chúng, mà tôi nghĩ nếu bạn biết mình
cần chúng, bạn đã biết nhiều hơn tôi rồi.

Nhưng ngay cả phần cơ bản cũng có những thứ lạ. Nên cài dây an toàn
nào mọi người, vì Kansas sắp tạm biệt rồi đây.

## Kiểm tra hỗ trợ Atomic

[i[`__STDC_NO_ATOMICS__` macro]<]

Atomics là tính năng tuỳ chọn. Có một macro `__STDC_NO_ATOMICS__`
bằng `1` nếu bạn _không_ có atomics.

Macro đó có thể không tồn tại trước C11, nên ta nên kiểm tra phiên
bản ngôn ngữ với `__STDC_VERSION__`^[Macro `__STDC_VERSION__` không
tồn tại trong C89 đời đầu, nên nếu bạn lo về điều đó, kiểm tra với
`#ifdef`.].

``` {.c}
#if __STDC_VERSION__ < 201112L || __STDC_NO_ATOMICS__ == 1
#define HAS_ATOMICS 0
#else
#define HAS_ATOMICS 1
#endif
```

[i[`__STDC_NO_ATOMICS__` macro]>]

[i[Atomic variables-->compiling with]<]

Nếu những test đó qua, bạn có thể an toàn include `<stdatomic.h>`,
header làm cơ sở cho phần còn lại của chương này. Nhưng nếu không có
hỗ trợ atomic, header đó có thể thậm chí không tồn tại.

Trên vài hệ thống, bạn có thể cần thêm `-latomic` vào cuối dòng
lệnh compile để dùng các hàm trong header đó.

[i[Atomic variables-->compiling with]>]

## Biến Atomic

Đây là _một phần_ của cách biến atomic hoạt động:

Nếu bạn có biến atomic chia sẻ và ghi vào nó từ một thread, lần ghi
đó sẽ là _all-or-nothing_ trong thread khác.

Tức là thread khác sẽ thấy toàn bộ lần ghi, ví dụ giá trị 32 bit.
Không phải một nửa. Không có cách nào để một thread ngắt thread
khác đang ở _giữa_ một lần ghi atomic nhiều byte.

Gần như có một cái lock nhỏ quanh việc lấy và set biến đó. (Và có
_thể_ có thật! Xem [Biến Atomic Lock-Free](#lock-free-atomic) bên
dưới.)

Nhân đây, bạn có thể thoát khỏi việc dùng atomic nếu bạn dùng mutex
để lock critical section. Chỉ là có một lớp _cấu trúc dữ liệu
lock-free_ luôn cho phép các thread khác tiến tới thay vì bị block
bởi mutex... nhưng cái này tạo ra đúng từ đầu khá khó, và là một
trong những thứ nằm ngoài phạm vi guide này, buồn thay.

Đó mới chỉ là một phần. Nhưng là phần ta bắt đầu.

Trước khi đi tiếp, làm sao khai báo biến là atomic?

Đầu tiên, include [i[`stdatomic.h` header]] `<stdatomic.h>`.

[i[`atomic_int` type]<]

Cái này cho ta các kiểu như `atomic_int`.

Rồi ta có thể đơn giản khai báo biến có kiểu đó.

Nhưng hãy làm demo có hai thread. Thread thứ nhất chạy một hồi rồi
set một biến thành giá trị cụ thể, rồi thoát. Thread kia chạy cho
đến khi thấy giá trị đó được set, rồi thoát.

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>
#include <stdatomic.h>

atomic_int x;   // THE POWER OF ATOMICS! BWHAHAHA!

int thread1(void *arg)
{
    (void)arg;

    printf("Thread 1: Sleeping for 1.5 seconds\n");
    thrd_sleep(&(struct timespec){.tv_sec=1, .tv_nsec=500000000}, NULL);

    printf("Thread 1: Setting x to 3490\n");
    x = 3490;

    printf("Thread 1: Exiting\n");
    return 0;
}

int thread2(void *arg)
{
    (void)arg;

    printf("Thread 2: Waiting for 3490\n");
    while (x != 3490) {}  // spin here

    printf("Thread 2: Got 3490--exiting!\n");
    return 0;
}

int main(void)
{
    x = 0;

    thrd_t t1, t2;

    thrd_create(&t1, thread1, NULL);
    thrd_create(&t2, thread2, NULL);

    thrd_join(t1, NULL);
    thrd_join(t2, NULL);

    printf("Main    : Threads are done, so x better be 3490\n");
    printf("Main    : And indeed, x == %d\n", x);
}
```

[i[`atomic_int` type]>]

Thread thứ hai spin tại chỗ, nhìn vào cờ và đợi nó được set thành
giá trị `3490`. Và thread thứ nhất làm điều đó.

Và tôi nhận được output này:

``` {.default}
Thread 1: Sleeping for 1.5 seconds
Thread 2: Waiting for 3490
Thread 1: Setting x to 3490
Thread 1: Exiting
Thread 2: Got 3490--exiting!
Main    : Threads are done, so x better be 3490
Main    : And indeed, x == 3490
```

Nhìn nè, mẹ ơi! Ta đang truy cập biến từ các thread khác nhau mà
không dùng mutex! Và cái đó chạy mọi lần nhờ bản chất atomic của
biến atomic.

Bạn có thể đang thắc mắc chuyện gì xảy ra nếu đó là `int` thường
không-atomic. Trên máy tôi vẫn chạy... trừ khi tôi build có
optimization thì nó hang trên thread 2 đợi thấy 3490 được
set^[Lý do là khi optimize, compiler của tôi đã đặt giá trị `x` vào
register để làm `while` loop nhanh. Nhưng register không có cách
nào biết biến đã được cập nhật trong thread khác, nên nó không bao
giờ thấy `3490`. Cái này không thực sự liên quan phần
_all-or-nothing_ của atomicity, mà liên quan hơn đến các khía cạnh
đồng bộ trong phần tiếp.].

Nhưng đó mới chỉ là đầu câu chuyện. Phần tiếp sẽ cần nhiều năng lực
não hơn và có liên quan một thứ gọi là _synchronization_.

## Synchronization

[i[Atomic variables-->synchronization]<]

Phần tiếp của câu chuyện là về khi nào các lần ghi bộ nhớ trong một
thread trở nên có thể thấy với các thread khác.

Bạn có thể nghĩ là ngay lập tức, đúng không? Nhưng không phải.
Nhiều thứ có thể sai. Sai một cách kỳ lạ.

Compiler có thể đã sắp xếp lại các truy cập bộ nhớ nên khi bạn nghĩ
đã set giá trị tương đối so với cái khác có thể không đúng. Và dù
compiler không làm, CPU của bạn có thể đã làm on the fly. Hoặc có
thể có gì đó khác về kiến trúc đó gây ra việc ghi trên một CPU bị
delay trước khi có thể thấy trên CPU khác.

Tin tốt là ta có thể gom tất cả rắc rối tiềm ẩn này vào một: các
truy cập bộ nhớ không đồng bộ có thể xuất hiện không theo thứ tự
tuỳ thread nào đang quan sát, như thể các dòng code đã bị sắp xếp
lại.

Lấy ví dụ, cái nào xảy ra trước trong code sau, ghi `x` hay ghi
`y`?

``` {.c .numberLines}
int x, y;  // global

// ...

x = 2;
y = 3;

printf("%d %d\n", x, y);
```

Đáp án: ta không biết. Compiler hay CPU có thể âm thầm đảo dòng 5
và 6 mà ta không hay. Code sẽ chạy single-thread _như thể_ nó được
thực thi theo thứ tự code.

Trong kịch bản đa luồng, ta có thể có pseudocode như vầy:

``` {.c .numberLines}
int x = 0, y = 0;

thread1() {
    x = 2;
    y = 3;
}

thread2() {
    while (y != 3) {}  // spin
    printf("x is now %d\n", x);  // 2? ...or 0?
}
```

Output của thread 2 là gì?

Nếu `x` được gán `2` _trước_ khi `y` được gán `3`, tôi kỳ vọng
output rất hợp lý là:

``` {.default}
x is now 2 
```

Nhưng cái gì đó lén lút có thể sắp xếp lại dòng 4 và 5 làm ta thấy
giá trị `0` của `x` khi in.

Nói cách khác, mọi thứ không chắc chắn trừ khi ta có thể bằng cách
nào đó nói, "Tại điểm này, tôi kỳ vọng tất cả các lần ghi trước đó
trong thread khác đều thấy được trong thread này."

Hai thread _đồng bộ_ khi chúng thống nhất về trạng thái bộ nhớ chia
sẻ. Như đã thấy, chúng không phải luôn đồng ý với code. Vậy chúng
đồng ý cách nào?

Dùng biến atomic có thể ép sự đồng ý đó^[Cho đến khi tôi nói khác,
tôi đang nói chung về các thao tác _sequentially consistent_. Nói
thêm ý nghĩa của nó sớm thôi.]. Nếu một thread ghi vào biến atomic,
nó đang nói "ai đọc biến atomic này trong tương lai cũng sẽ thấy
tất cả thay đổi tôi đã làm với bộ nhớ (atomic hay không) cho đến và
bao gồm biến atomic này".

Hay theo kiểu người hơn, cùng ngồi quanh bàn họp và bảo đảm rằng ta
cùng chung một trang về các mảnh bộ nhớ chia sẻ nào giữ giá trị
nào. Bạn đồng ý rằng các thay đổi bộ nhớ bạn đã làm cho đến và bao
gồm lần store atomic sẽ thấy được với tôi sau khi tôi load cùng
biến atomic đó.

Nên ta có thể dễ dàng sửa ví dụ:

``` {.c .numberLines}
int x = 0;
atomic int y = 0;  // Make y atomic

thread1() {
    x = 2;
    y = 3;             // Synchronize on write
}

thread2() {
    while (y != 3) {}  // Synchronize on read
    printf("x is now %d\n", x);  // 2, period.
}
```

Vì các thread đồng bộ qua `y`, tất cả các lần ghi trong thread 1
xảy ra _trước_ lần ghi vào `y` đều thấy được trong thread 2 _sau_
lần đọc từ `y` (trong `while` loop).

Quan trọng chú ý vài điều ở đây:

1. Không có gì ngủ. Synchronization không phải thao tác blocking.
   Cả hai thread chạy hết ga đến khi thoát. Thậm chí cái bị kẹt
   trong spin loop cũng không block ai khác khỏi chạy.

2. Synchronization xảy ra khi một thread đọc biến atomic mà thread
   khác đã ghi. Nên khi thread 2 đọc `y`, tất cả lần ghi bộ nhớ
   trước trong thread 1 (cụ thể là set `x`) sẽ thấy được trong
   thread 2.

3. Chú ý `x` không atomic. Vẫn OK vì ta không đang đồng bộ qua `x`,
   và việc đồng bộ qua `y` khi ta ghi nó trong thread 1 nghĩa là
   tất cả lần ghi trước, bao gồm `x`, trong thread 1 sẽ thấy được
   với các thread khác... nếu các thread đó đọc `y` để đồng bộ.

Ép synchronization này kém hiệu quả và có thể chậm hơn nhiều so
với dùng biến thường. Đây là lý do ta không dùng atomic trừ khi
phải dùng cho ứng dụng cụ thể.

Đó là cơ bản. Xem sâu hơn nào.

[i[Atomic variables-->synchronization]<]

## Acquire và Release

[i[Atomic variables-->acquire]<]
[i[Atomic variables-->release]<]

Thêm thuật ngữ! Học giờ thì có lợi sau.

Khi một thread đọc biến atomic, đó được gọi là thao tác _acquire_.

Khi một thread ghi biến atomic, đó được gọi là thao tác _release_.

Những cái này là gì? Xếp chúng vào các thuật ngữ bạn đã biết về
biến atomic:

**Read = Load = Acquire**. Như khi bạn so sánh biến atomic hay đọc
nó để copy sang giá trị khác.

**Write = Store = Release**. Như khi bạn gán giá trị vào biến
atomic.

Khi dùng biến atomic với ngữ nghĩa acquire/release, C nêu rõ chuyện
gì có thể xảy ra khi nào.

Acquire/release tạo cơ sở cho synchronization ta vừa nói.

Khi một thread acquire biến atomic, nó có thể thấy giá trị đã set
trong thread khác đã release cùng biến đó.

Nói cách khác:

Khi một thread đọc biến atomic, nó có thể thấy giá trị đã set trong
thread khác đã ghi cùng biến đó.

Synchronization xảy ra qua cặp acquire/release.

Chi tiết thêm:

Với read/load/acquire một biến atomic cụ thể:

* Tất cả lần ghi (atomic hay không) trong thread khác xảy ra trước
  khi thread đó write/store/release biến atomic này giờ thấy được
  trong thread này.

* Giá trị mới của biến atomic do thread khác set cũng thấy được
  trong thread này.

* Không có lần đọc hay ghi biến/bộ nhớ nào trong thread hiện tại có
  thể bị sắp xếp lại xảy ra trước acquire này.

* Acquire đóng vai rào chắn một chiều khi sắp xếp lại code; các
  lần đọc và ghi trong thread hiện tại có thể bị di chuyển xuống từ
  _trước_ acquire thành _sau_ nó. Nhưng quan trọng hơn với
  synchronization, không gì có thể di chuyển lên từ _sau_ acquire
  thành _trước_ nó.

Với write/store/release một biến atomic cụ thể:

* Tất cả lần ghi (atomic hay không) trong thread hiện tại xảy ra
  trước release này trở nên thấy được với các thread khác đã
  read/load/acquire cùng biến atomic.

* Giá trị thread này ghi vào biến atomic này cũng thấy được với
  các thread khác.

* Không có lần đọc hay ghi biến/bộ nhớ nào trong thread hiện tại có
  thể bị sắp xếp lại xảy ra sau release này.

* Release đóng vai rào chắn một chiều khi sắp xếp lại code: các
  lần đọc và ghi trong thread hiện tại có thể bị di chuyển lên từ
  _sau_ release thành _trước_ nó. Nhưng quan trọng hơn với
  synchronization, không gì có thể di chuyển xuống từ _trước_
  release thành _sau_ nó.

Lại nữa, kết quả là synchronization bộ nhớ từ thread này sang
thread khác. Thread thứ hai có thể chắc chắn rằng biến và bộ nhớ
được ghi theo thứ tự lập trình viên mong muốn.

```
int x, y, z = 0;
atomic_int a = 0;

thread1() {
    x = 10;
    y = 20;
    a = 999;  // Release
    z = 30;
}

thread2()
{
    while (a != 999) { } // Acquire

    assert(x == 10);  // never asserts, x is always 10
    assert(y == 20);  // never asserts, y is always 20

    assert(z == 0);  // might assert!!
}
```

Trong ví dụ trên, `thread2` có thể chắc chắn về giá trị của `x` và
`y` sau khi nó acquire `a` vì chúng được set trước khi `thread1`
release atomic `a`.

Nhưng `thread2` không thể chắc chắn về giá trị `z` vì nó xảy ra
sau release. Có thể việc gán cho `z` bị di chuyển lên trước việc
gán cho `a`.

Chú ý quan trọng: release một biến atomic không có tác dụng lên
acquire các biến atomic khác. Mỗi biến cô lập với các biến khác.

[i[Atomic variables-->acquire]>]
[i[Atomic variables-->release]>]

## Sequential Consistency

[i[Atomic variables-->sequential consistency]<]

Bạn vẫn còn trụ được chứ? Ta đã qua phần nội dung chính của cách
dùng atomic đơn giản hơn. Và vì ta không định nói về các cách dùng
phức tạp hơn ở đây, bạn có thể thư giãn chút.

_Sequential consistency_ là cái gọi là _memory ordering_. Có nhiều
memory ordering, nhưng sequential consistency là tỉnh táo nhất^[Tỉnh
táo nhất từ góc nhìn của lập trình viên.] mà C có. Nó cũng là mặc
định. Bạn phải cố tình để dùng các memory ordering khác.

Tất cả thứ ta đã nói từ đầu đến giờ đều xảy ra trong lãnh địa của
sequential consistency.

Ta đã nói về cách compiler hay CPU có thể sắp xếp lại lần đọc và
ghi bộ nhớ trong một thread miễn là tuân theo quy tắc _as-if_.

Và ta đã thấy cách phanh hành vi này bằng cách đồng bộ qua biến
atomic.

Hãy chính thức hoá thêm chút.

Nếu các thao tác là _sequentially consistent_, nghĩa là cuối ngày,
khi mọi thứ đã nói xong, tất cả các thread có thể gác chân, mở đồ
uống yêu thích, và đồng ý về thứ tự các thay đổi bộ nhớ xảy ra
trong lần chạy. Và thứ tự đó là cái được quy định bởi code.

Một cái sẽ không nói, "Nhưng chẳng phải _B_ xảy ra trước _A_ sao?"
nếu các cái khác nói, "_A_ chắc chắn xảy ra trước _B_". Tất cả đều
bạn bè nhau ở đây.

Đặc biệt, trong một thread, không có acquire và release nào có thể
bị sắp xếp lại so với nhau. Cái này thêm vào các quy tắc về những
truy cập bộ nhớ khác có thể bị sắp xếp lại quanh chúng.

Quy tắc này cho thêm một cấp độ tỉnh táo cho tiến trình các load/
acquire và store/release atomic.

Mọi memory order khác trong C đều liên quan việc nới lỏng các quy
tắc sắp xếp lại, cho acquires/releases hoặc cho các truy cập bộ
nhớ khác, atomic hay không. Bạn sẽ làm vậy nếu _thực sự_ biết mình
đang làm gì và cần tăng tốc. _Đây là đất của đội quân rồng..._

Nói thêm sau, nhưng giờ cứ dùng cái an toàn thực dụng.

[i[Atomic variables-->sequential consistency]>]

## Gán Atomic và các Toán tử

[i[Atomic variables-->assignments and operators]<]

Một số toán tử trên biến atomic là atomic. Và những cái khác thì
không.

Hãy bắt đầu với một phản ví dụ:

``` {.c}
atomic_int x = 0;

thread1() {
    x = x + 3;  // NOT atomic!
}
```

Vì có lần đọc `x` ở bên phải phép gán và lần ghi hiệu quả ở bên
trái, đây là hai thao tác. Một thread khác có thể chen vào giữa và
làm bạn phật lòng.

Nhưng bạn _có thể_ dùng shorthand `+=` để được thao tác atomic:

``` {.c}
atomic_int x = 0;

thread1() {
    x += 3;   // ATOMIC!
}
```

Trong trường hợp đó, `x` sẽ được tăng atomic thêm `3`, không thread
nào khác có thể nhảy vào giữa.

Đặc biệt, các toán tử sau là thao tác atomic read-modify-write với
sequential consistency, nên cứ dùng thoải mái trong niềm vui. (Trong
ví dụ, `a` là atomic.)

``` {.c}
a++       a--       --a       ++a
a += b    a -= b    a *= b    a /= b    a %= b
a &= b    a |= b    a ^= b    a >>= b   a <<= b
```

[i[Atomic variables-->assignments and operators]>]

## Các hàm thư viện tự đồng bộ

[i[Atomic variables-->synchronized library functions]<]

Đến giờ ta đã nói cách đồng bộ với biến atomic, nhưng hoá ra có
vài hàm thư viện tự làm đồng bộ hạn chế sau cánh gà.

``` {.c}
call_once()      thrd_create()       thrd_join()
mtx_lock()       mtx_timedlock()     mtx_trylock()
malloc()         calloc()            realloc()
aligned_alloc()
```

**`call_once()`**: Đồng bộ với tất cả các lần gọi tiếp theo tới
`call_once()` cho một flag cụ thể. Cách này các lần gọi tiếp theo
có thể yên tâm rằng nếu thread khác set flag, chúng sẽ thấy.

**`thrd_create()`**: Đồng bộ với phần đầu của thread mới. Thread
mới có thể chắc chắn nó sẽ thấy tất cả các lần ghi bộ nhớ chia sẻ
từ thread cha trước khi gọi `thrd_create()`.

**`thrd_join()`**: Khi một thread chết, nó đồng bộ với hàm này.
Thread đã gọi `thrd_join()` có thể yên tâm rằng nó có thể thấy tất
cả các lần ghi chia sẻ của thread đã chết.

**`mtx_lock()`**: Các lần gọi trước tới `mtx_unlock()` trên cùng
mutex đồng bộ với lần gọi này. Đây là trường hợp phản chiếu nhiều
nhất tiến trình acquire/release ta đã nói. `mtx_unlock()` thực
hiện release trên biến mutex, bảo đảm bất kỳ thread sau nào
acquire với `mtx_lock()` có thể thấy tất cả thay đổi bộ nhớ chia
sẻ trong critical section.

**`mtx_timedlock()`** và **`mtx_trylock()`**: Tương tự tình huống
với `mtx_lock()`, nếu lần gọi này thành công, các lần gọi trước
tới `mtx_unlock()` đồng bộ với cái này.

**Các hàm bộ nhớ động**: nếu bạn cấp phát bộ nhớ, nó đồng bộ với
lần giải phóng trước đó của cùng bộ nhớ. Và các lần cấp phát và
giải phóng vùng bộ nhớ đó xảy ra theo một thứ tự tổng thể duy nhất
mà tất cả thread có thể đồng ý. Tôi _nghĩ_ ý tưởng ở đây là lần
giải phóng có thể xoá sạch vùng nếu nó chọn, và ta muốn chắc rằng
lần cấp phát sau không thấy dữ liệu không bị xoá. Ai đó báo tôi
biết nếu còn gì khác.

[i[Atomic variables-->synchronized library functions]>]

## Bộ chỉ định kiểu Atomic, Qualifier

Hạ một chút xem ta có các kiểu nào sẵn, và làm sao tạo kiểu atomic
mới.

[i[`_Atomic` type qualifier]<]

Đầu tiên, xem các kiểu atomic có sẵn và chúng được `typedef` tới
cái gì. (Spoiler: `_Atomic` là một type qualifier!)

[i[`atomic_bool` type]]
[i[`atomic_char` type]]
[i[`atomic_schar` type]]
[i[`atomic_uchar` type]]
[i[`atomic_short` type]]
[i[`atomic_ushort` type]]
[i[`atomic_int` type]]
[i[`atomic_uint` type]]
[i[`atomic_long` type]]
[i[`atomic_ulong` type]]
[i[`atomic_llong` type]]
[i[`atomic_ullong` type]]
[i[`atomic_char16_t` type]]
[i[`atomic_char32_t` type]]
[i[`atomic_wchar_t` type]]
[i[`atomic_int_least8_t` type]]
[i[`atomic_uint_least8_t` type]]
[i[`atomic_int_least16_t` type]]
[i[`atomic_uint_least16_t` type]]
[i[`atomic_int_least32_t` type]]
[i[`atomic_uint_least32_t` type]]
[i[`atomic_int_least64_t` type]]
[i[`atomic_uint_least64_t` type]]
[i[`atomic_int_fast8_t` type]]
[i[`atomic_uint_fast8_t` type]]
[i[`atomic_int_fast16_t` type]]
[i[`atomic_uint_fast16_t` type]]
[i[`atomic_int_fast32_t` type]]
[i[`atomic_uint_fast32_t` type]]
[i[`atomic_int_fast64_t` type]]
[i[`atomic_uint_fast64_t` type]]
[i[`atomic_intptr_t` type]]
[i[`atomic_uintptr_t` type]]
[i[`atomic_size_t` type]]
[i[`atomic_ptrdiff_t` type]]
[i[`atomic_intmax_t` type]]
[i[`atomic_uintmax_t` type]]

|Kiểu Atomic|Dạng dài tương đương|
|-|-|
|`atomic_bool`|`_Atomic _Bool`|
|`atomic_char`|`_Atomic char`|
|`atomic_schar`|`_Atomic signed char`|
|`atomic_uchar`|`_Atomic unsigned char`|
|`atomic_short`|`_Atomic short`|
|`atomic_ushort`|`_Atomic unsigned short`|
|`atomic_int`|`_Atomic int`|
|`atomic_uint`|`_Atomic unsigned int`|
|`atomic_long`|`_Atomic long`|
|`atomic_ulong`|`_Atomic unsigned long`|
|`atomic_llong`|`_Atomic long long`|
|`atomic_ullong`|`_Atomic unsigned long long`|
|`atomic_char16_t`|`_Atomic char16_t`|
|`atomic_char32_t`|`_Atomic char32_t`|
|`atomic_wchar_t`|`_Atomic wchar_t`|
|`atomic_int_least8_t`|`_Atomic int_least8_t`|
|`atomic_uint_least8_t`|`_Atomic uint_least8_t`|
|`atomic_int_least16_t`|`_Atomic int_least16_t`|
|`atomic_uint_least16_t`|`_Atomic uint_least16_t`|
|`atomic_int_least32_t`|`_Atomic int_least32_t`|
|`atomic_uint_least32_t`|`_Atomic uint_least32_t`|
|`atomic_int_least64_t`|`_Atomic int_least64_t`|
|`atomic_uint_least64_t`|`_Atomic uint_least64_t`|
|`atomic_int_fast8_t`|`_Atomic int_fast8_t`|
|`atomic_uint_fast8_t`|`_Atomic uint_fast8_t`|
|`atomic_int_fast16_t`|`_Atomic int_fast16_t`|
|`atomic_uint_fast16_t`|`_Atomic uint_fast16_t`|
|`atomic_int_fast32_t`|`_Atomic int_fast32_t`|
|`atomic_uint_fast32_t`|`_Atomic uint_fast32_t`|
|`atomic_int_fast64_t`|`_Atomic int_fast64_t`|
|`atomic_uint_fast64_t`|`_Atomic uint_fast64_t`|
|`atomic_intptr_t`|`_Atomic intptr_t`|
|`atomic_uintptr_t`|`_Atomic uintptr_t`|
|`atomic_size_t`|`_Atomic size_t`|
|`atomic_ptrdiff_t`|`_Atomic ptrdiff_t`|
|`atomic_intmax_t`|`_Atomic intmax_t`|
|`atomic_uintmax_t`|`_Atomic uintmax_t`|

[i[`_Atomic` type qualifier]>]

Dùng chúng thoải mái! Chúng nhất quán với các alias atomic trong
C++, nếu điều đó có ích.

Nhưng nếu bạn muốn nhiều hơn?

Bạn có thể làm với type qualifier hoặc type specifier.

[i[`_Atomic` type specifier]<]

Đầu tiên, specifier! Đó là từ khoá `_Atomic` với kiểu trong ngoặc
sau^[Có vẻ C++23 thêm cái này như một macro.], phù hợp dùng với
`typedef`:

``` {.c}
typedef _Atomic(double) atomic_double;

atomic_double f;
```

Hạn chế với specifier: kiểu bạn đang làm atomic không thể là kiểu
mảng hay hàm, cũng không thể là atomic hay đã qualified kiểu khác.

[i[`_Atomic` type specifier]>]
[i[`_Atomic` type qualifier]<]

Tiếp, qualifier! Đó là từ khoá `_Atomic` _không_ có kiểu trong
ngoặc sau.

Nên hai cái này làm việc tương tự^[Spec lưu ý chúng có thể khác về
kích thước, biểu diễn, và căn chỉnh.]:

``` {.c}
_Atomic(int) i;   // type specifier
_Atomic int  j;   // type qualifier
```

Điểm khác là bạn có thể include type qualifier khác với cái sau:

``` {.c}
_Atomic volatile int k;   // qualified atomic variable
```

Hạn chế với qualifier: kiểu bạn đang làm atomic không thể là kiểu
mảng hay hàm.

[i[`_Atomic` type qualifier]>]

## Biến Atomic Lock-Free {#lock-free-atomic}

[i[Atomic variables-->lock-free]<]

Kiến trúc phần cứng bị hạn chế về lượng dữ liệu có thể atomic đọc
và ghi. Tuỳ vào cách nó được kết nối. Và khác nhau.

Nếu bạn dùng kiểu atomic, bạn có thể yên tâm rằng truy cập kiểu đó
sẽ atomic... nhưng có một điều: nếu phần cứng không làm được, nó
được làm bằng lock thay.

Nên truy cập atomic trở thành lock-access-unlock, chậm hơn khá và
có một số ngụ ý với signal handler.

[Atomic flags](#atomic-flags) bên dưới là kiểu atomic duy nhất được
đảm bảo lock-free trong tất cả implementation tuân chuẩn. Trong
thế giới desktop/laptop máy tính thông thường, các kiểu lớn khác
có thể cũng lock-free.

May thay, ta có vài cách để xác định liệu kiểu cụ thể có phải
atomic lock-free hay không.

Trước hết, vài macro, bạn có thể dùng ở compile time với `#if`.
Chúng áp dụng cho cả kiểu signed lẫn unsigned.

[i[`ATOMIC_BOOL_LOCK_FREE` macro]]
[i[`ATOMIC_CHAR_LOCK_FREE` macro]]
[i[`ATOMIC_CHAR16_T_LOCK_FREE` macro]]
[i[`ATOMIC_CHAR32_T_LOCK_FREE` macro]]
[i[`ATOMIC_WCHAR_T_LOCK_FREE` macro]]
[i[`ATOMIC_SHORT_LOCK_FREE` macro]]
[i[`ATOMIC_INT_LOCK_FREE` macro]]
[i[`ATOMIC_LONG_LOCK_FREE` macro]]
[i[`ATOMIC_LLONG_LOCK_FREE` macro]]
[i[`ATOMIC_POINTER_LOCK_FREE` macro]]

|Kiểu Atomic|Macro Lock Free|
|-|-|
|`atomic_bool`|`ATOMIC_BOOL_LOCK_FREE`|
|`atomic_char`|`ATOMIC_CHAR_LOCK_FREE`|
|`atomic_char16_t`|`ATOMIC_CHAR16_T_LOCK_FREE`|
|`atomic_char32_t`|`ATOMIC_CHAR32_T_LOCK_FREE`|
|`atomic_wchar_t`|`ATOMIC_WCHAR_T_LOCK_FREE`|
|`atomic_short`|`ATOMIC_SHORT_LOCK_FREE`|
|`atomic_int`|`ATOMIC_INT_LOCK_FREE`|
|`atomic_long`|`ATOMIC_LONG_LOCK_FREE`|
|`atomic_llong`|`ATOMIC_LLONG_LOCK_FREE`|
|`atomic_intptr_t`|`ATOMIC_POINTER_LOCK_FREE`|

Các macro này thú vị có thể có _ba_ giá trị khác nhau:

|Giá trị|Ý nghĩa|
|-|-|
|`0`|Không bao giờ lock-free.|
|`1`|_Đôi khi_ lock-free.|
|`2`|Luôn lock-free.|

Khoan, cái gì đó _đôi khi_ lock-free được là sao? Nghĩa là đáp án
không biết tại compile-time, nhưng có thể biết sau tại runtime. Có
thể đáp án khác tuỳ bạn đang chạy code trên Genuine Intel hay AMD
hay gì đó^[Tôi chỉ lấy ví dụ đó từ không khí. Có thể không quan
trọng trên Intel/AMD, nhưng có thể quan trọng ở đâu đó đấy!].

Nhưng bạn luôn có thể test tại runtime với hàm
[i[`atomic_is_lock_free()` function]] `atomic_is_lock_free()`. Hàm
này trả về true hay false nếu kiểu cụ thể là atomic ngay bây giờ.

Tại sao ta quan tâm?

Lock-free nhanh hơn, nên có thể có vấn đề tốc độ bạn muốn code
tránh theo cách khác. Hoặc có thể bạn cần dùng biến atomic trong
signal handler.

[i[Atomic variables-->lock-free]>]

### Signal Handlers và Atomic Lock-Free

[i[Signal handlers-->with lock-free atomics]<]
[i[Atomic variables-->with signal handlers]<]

Nếu bạn đọc hay ghi biến chia sẻ (thời lượng lưu trữ static hay
`_Thread_Local`) trong signal handler, đó là hành vi không xác định
[gasp!]... Trừ khi bạn làm một trong các điều sau:

1. Ghi vào biến kiểu `volatile sig_atomic_t`.

2. Đọc hay ghi biến atomic lock-free.

Theo tôi thấy, biến atomic lock-free là một trong số ít cách
portable lấy thông tin ra khỏi signal handler.

Spec hơi mơ hồ, theo cách tôi đọc, về memory order khi acquire hay
release biến atomic trong signal handler. C++ nói, và hợp lý, rằng
các truy cập đó là không tuần tự so với phần còn lại của chương
trình^[C++ nói thêm nếu signal là kết quả của lần gọi
[i[`raise()` function]] `raise()`, nó tuần tự _sau_ `raise()`.].
Signal có thể được raise bất cứ lúc nào. Nên tôi giả định hành vi
của C tương tự.

[i[Signal handlers-->with lock-free atomics]>]
[i[Atomic variables-->with signal handlers]>]

## Atomic Flags {#atomic-flags}

[i[Atomic variables-->atomic flags]<]
[i[`atomic_flag` type]<]

Chỉ có một kiểu mà chuẩn đảm bảo sẽ là lock-free atomic:
`atomic_flag`. Đây là kiểu mờ (opaque) cho các thao tác
[flw[test-and-set|Test-and-set]].

Nó có thể là _set_ hoặc _clear_. Bạn có thể khởi tạo nó thành clear
với:

[i[`ATOMIC_FLAG_INIT` macro]<]

``` {.c}
atomic_flag f = ATOMIC_FLAG_INIT;
```

[i[`ATOMIC_FLAG_INIT` macro]>]

[i[`atomic_flag_test_and_set()` function]<]

Bạn có thể set flag atomic với `atomic_flag_test_and_set()`, sẽ set
flag và trả về trạng thái trước của nó dưới dạng `_Bool` (true cho
set).

[i[`atomic_flag_clear()` function]<]

Bạn có thể clear flag atomic với `atomic_flag_clear()`.

Đây là ví dụ ta init flag thành clear, set hai lần, rồi clear lại.

``` {.c}
#include <stdio.h>
#include <stdbool.h>   // Not needed in C23
#include <stdatomic.h>

atomic_flag f = ATOMIC_FLAG_INIT;

int main(void)
{
    bool r = atomic_flag_test_and_set(&f);
    printf("Value was: %d\n", r);           // 0

    r = atomic_flag_test_and_set(&f);
    printf("Value was: %d\n", r);           // 1

    atomic_flag_clear(&f);
    r = atomic_flag_test_and_set(&f);
    printf("Value was: %d\n", r);           // 0
}
```

[i[`atomic_flag_clear()` function]>]
[i[`atomic_flag_test_and_set()` function]>]
[i[Atomic variables-->atomic flags]>]
[i[`atomic_flag` type]>]

## `struct` và `union` Atomic

[i[Atomic variables-->`struct` and `union`]<]

Dùng qualifier hay specifier `_Atomic`, bạn có thể tạo `struct`
hay `union` atomic! Khá đáng kinh ngạc.

Nếu không có nhiều dữ liệu bên trong (tức là vài byte), kiểu atomic
tạo ra có thể lock-free. Test bằng `atomic_is_lock_free()`.

``` {.c .numberLines}
#include <stdio.h>
#include <stdatomic.h>

int main(void)
{
    struct point {
        float x, y;
    };

    _Atomic(struct point) p;

    printf("Is lock free: %d\n", atomic_is_lock_free(&p));
}
```

Đây là cái bắt: bạn không thể truy cập field của `struct` hay
`union` atomic... nên có ý nghĩa gì? À, bạn có thể atomic _copy_
toàn bộ `struct` vào biến không-atomic rồi dùng. Bạn cũng có thể
atomic copy ngược lại.

``` {.c .numberLines}
#include <stdio.h>
#include <stdatomic.h>

int main(void)
{
    struct point {
        float x, y;
    };

    _Atomic(struct point) p;
    struct point t;

    p = (struct point){1, 2};  // Atomic copy

    //printf("%f\n", p.x);  // Error

    t = p;   // Atomic copy

    printf("%f\n", t.x);  // OK!
}
```

Bạn cũng có thể khai báo `struct` mà các field riêng lẻ là atomic.
Là implementation-defined xem kiểu atomic có được phép trên bitfield
hay không.

[i[Atomic variables-->`struct` and `union`]>]

## Con trỏ Atomic

[i[Atomic variables-->pointers]<]

Chỉ ghi chú ở đây về vị trí `_Atomic` khi nói tới con trỏ.

Đầu tiên, con trỏ tới atomic (tức là giá trị con trỏ không atomic,
nhưng thứ nó trỏ tới thì atomic):

``` {.c}
_Atomic int x;
_Atomic int *p;  // p is a pointer to an atomic int

p = &x;  // OK!
```

Thứ hai, con trỏ atomic tới giá trị không-atomic (tức là giá trị
con trỏ tự thân atomic, nhưng thứ nó trỏ tới thì không):

``` {.c}
int x;
int * _Atomic p;  // p is an atomic pointer to an int

p = &x;  // OK!
```

Cuối cùng, con trỏ atomic tới giá trị atomic (tức là con trỏ và
thứ nó trỏ tới đều atomic):

``` {.c}
_Atomic int x;
_Atomic int * _Atomic p;  // p is an atomic pointer to an atomic int

p = &x;  // OK!
```

[i[Atomic variables-->pointers]>]

## Memory Order

[i[Atomic variables-->memory order]<]
[i[Memory order]<]

Ta đã nói về sequential consistency, cái hợp lý trong nhóm. Nhưng
còn một số cái khác:

[i[`memory_order_seq_cst` enumerated type]]
[i[`memory_order_acq_rel` enumerated type]]
[i[`memory_order_release` enumerated type]]
[i[`memory_order_acquire` enumerated type]]
[i[`memory_order_consume` enumerated type]]
[i[`memory_order_relaxed` enumerated type]]

|`memory_order`|Mô tả|
|-|-|
|`memory_order_seq_cst`|Sequential Consistency|
|`memory_order_acq_rel`|Acquire/Release|
|`memory_order_release`|Release|
|`memory_order_acquire`|Acquire|
|`memory_order_consume`|Consume|
|`memory_order_relaxed`|Relaxed|

Bạn có thể chỉ định các cái khác với một số hàm thư viện. Ví dụ,
bạn có thể cộng giá trị vào biến atomic như vầy:

``` {.c}
atomic_int x = 0;

x += 5;  // Sequential consistency, the default
```

Hay bạn có thể làm tương tự với hàm thư viện này:

[i[`atomic_fetch_add()` function]<]

``` {.c}
atomic_int x = 0;

atomic_fetch_add(&x, 5);  // Sequential consistency, the default
```
[i[`atomic_fetch_add()` function]>]

Hay bạn có thể làm tương tự với memory ordering tường minh:

[i[`atomic_fetch_add_explicit()` function]<]

``` {.c}
atomic_int x = 0;

atomic_fetch_add_explicit(&x, 5, memory_order_seq_cst);
```

Nhưng nếu ta không muốn sequential consistency? Và muốn
acquire/release thay vào đó vì lý do gì đó? Cứ gọi tên nó:

``` {.c}
atomic_int x = 0;

atomic_fetch_add_explicit(&x, 5, memory_order_acq_rel);
```

[i[`atomic_fetch_add_explicit()` function]>]

Ta sẽ chia nhỏ các memory order khác bên dưới. Đừng nghịch bất kỳ
cái gì khác ngoài sequential consistency trừ khi bạn biết đang làm
gì. Rất dễ mắc lỗi gây ra các failure hiếm, khó tái hiện.

### Sequential Consistency

[i[Atomic variables-->sequential consistency]<]
[i[Memory order-->sequential consistency]<]

* Thao tác Load acquire (xem bên dưới).
* Thao tác Store release (xem bên dưới).
* Thao tác Read-modify-write acquire rồi release.

Cũng vậy, để duy trì tổng thứ tự của acquire và release, không có
acquire hay release nào bị sắp xếp lại so với nhau. (Quy tắc
acquire/release không cấm sắp xếp lại một release theo sau là
acquire. Nhưng quy tắc sequentially consistent thì cấm.)

[i[Memory order-->sequential consistency]>]
[i[Atomic variables-->sequential consistency]>]

### Acquire

[i[Atomic variables-->acquire]<]
[i[Memory order-->acquire]<]

Đây là chuyện xảy ra trên thao tác load/read một biến atomic.

* Nếu thread khác đã release biến atomic này, tất cả các lần ghi
  thread đó làm giờ thấy được trong thread này.

* Các truy cập bộ nhớ trong thread này xảy ra sau lần load này
  không thể bị sắp xếp lại trước nó.

[i[Memory order-->acquire]>]
[i[Atomic variables-->acquire]>]

### Release

[i[Atomic variables-->release]<]
[i[Memory order-->acquire]<]

Đây là chuyện xảy ra trên store/write một biến atomic.

* Nếu thread khác sau này acquire biến atomic này, tất cả các lần
  ghi bộ nhớ trong thread này trước lần ghi atomic của nó trở nên
  thấy được với thread khác đó.

* Các truy cập bộ nhớ trong thread này xảy ra trước release không
  thể bị sắp xếp lại sau nó.

[i[Atomic variables-->release]>]
[i[Memory order-->release]>]

### Consume

[i[Atomic variables-->consume]<]
[i[Memory order-->consume]<]

Cái này hơi lạ, tương tự phiên bản ít nghiêm khắc hơn của acquire.
Nó ảnh hưởng các truy cập bộ nhớ _phụ thuộc dữ liệu_ vào biến
atomic.

"Phụ thuộc dữ liệu" mơ hồ nghĩa là biến atomic được dùng trong một
phép tính.

Tức là nếu một thread consume biến atomic thì tất cả các thao tác
trong thread đó tiếp tục dùng biến atomic đó sẽ có thể thấy các
lần ghi bộ nhớ trong thread đang release.

So với acquire nơi các lần ghi bộ nhớ trong thread đang release sẽ
thấy được với _tất cả_ các thao tác trong thread hiện tại, không
chỉ những cái phụ thuộc dữ liệu.

Cũng giống acquire, có hạn chế về thao tác nào có thể bị sắp xếp
lại _trước_ consume. Với acquire, bạn không thể sắp xếp lại bất cứ
gì trước nó. Với consume, bạn không thể sắp xếp lại bất cứ gì phụ
thuộc giá trị atomic đã load trước nó.

[i[Atomic variables-->consume]>]
[i[Memory order-->consume]>]

### Acquire/Release

[i[Atomic variables-->acquire/release]<]
[i[Memory order-->acquire/release]<]

Cái này chỉ áp dụng cho thao tác read-modify-write. Là một acquire
và release gom vào một.

* Acquire xảy ra cho lần read.
* Release xảy ra cho lần write.

[i[Atomic variables-->acquire/release]>]
[i[Memory order-->acquire/release]>]

### Relaxed

[i[Atomic variables-->relaxed]<]
[i[Memory order-->relaxed]<]

Không có quy tắc; là hỗn loạn! Ai cũng có thể sắp xếp lại mọi thứ
mọi nơi! Chó với mèo sống chung, loạn lớn!

Thực ra có một quy tắc. Lần đọc và ghi atomic vẫn là all-or-nothing.
Nhưng các thao tác có thể bị sắp xếp lại tuỳ hứng và không có
synchronization giữa các thread.

Có vài use case cho memory order này, bạn có thể tìm với một ít
tìm kiếm, ví dụ các counter đơn giản.

Và bạn có thể dùng fence để ép synchronization sau một loạt lần ghi
relaxed.

[i[Atomic variables-->relaxed]>]
[i[Memory order-->relaxed]>]
[i[Memory order]>]
[i[Atomic variables-->memory order]>]

## Fences

[i[Atomic variables-->fences]<]

Bạn biết cách release và acquire biến atomic xảy ra khi bạn đọc và
ghi chúng đúng không?

Thì ra cũng có thể làm release hay acquire mà _không_ có biến
atomic.

Cái này gọi là _fence_. Nên nếu bạn muốn tất cả các lần ghi trong
một thread thấy được ở nơi khác, bạn có thể đặt release fence trong
một thread và acquire fence trong thread khác, giống cách biến
atomic hoạt động.

Vì thao tác consume không thực sự có nghĩa trên fence^[Vì consume
là về các thao tác phụ thuộc giá trị biến atomic đã acquire, và
không có biến atomic với fence.], `memory_order_consume` được xử
lý như acquire.

Bạn có thể đặt fence với bất kỳ order nào được chỉ định:

[i[`atomic_thread_fence()` function]<]

``` {.c}
atomic_thread_fence(memory_order_release);
```

[i[`atomic_thread_fence()` function]>]
[i[`atomic_signal_fence()` function]<]

Còn có phiên bản fence nhẹ để dùng với signal handler, gọi là
`atomic_signal_fence()`.

Nó hoạt động y như `atomic_thread_fence()`, trừ:

* Nó chỉ liên quan khả năng thấy giá trị trong cùng thread; không
  có synchronization với thread khác.

* Không phát ra lệnh fence phần cứng.

Nếu bạn muốn chắc rằng side effect của thao tác không-atomic (và
thao tác atomic relaxed) thấy được trong signal handler, bạn có
thể dùng fence này.

Ý tưởng là signal handler đang thực thi trong _thread này_, không
phải thread khác, nên đây là cách nhẹ hơn để đảm bảo thay đổi bên
ngoài signal handler thấy được bên trong nó (tức là chúng không bị
sắp xếp lại).

[i[`atomic_signal_fence()` function]>]
[i[Atomic variables-->fences]>]

## Tham khảo

Nếu bạn muốn học thêm về mấy thứ này, đây là một số thứ đã giúp tôi
cày qua nó:

* Herb Sutter's _`atomic<>` Weapons_ talk:
  * [fl[Part 1|https://www.youtube.com/watch?v=A8eCGOqgvH4]]
  * [fl[part 2|https://www.youtube.com/watch?v=KeLBd2EJLOU]]

* [fl[Jeff Preshing's materials|https://preshing.com/archives/]], đặc biệt:
  * [fl[An Introduction to Lock-Free Programming|https://preshing.com/20120612/an-introduction-to-lock-free-programming/]]
  * [fl[Acquire and Release Semantics|https://preshing.com/20120913/acquire-and-release-semantics/]]
  * [fl[The _Happens-Before_ Relation|https://preshing.com/20130702/the-happens-before-relation/]]
  * [fl[The _Synchronizes-With_ Relation|https://preshing.com/20130823/the-synchronizes-with-relation/]]
  * [fl[The Purpose of `memory_order_consume` in C++11|https://preshing.com/20140709/the-purpose-of-memory_order_consume-in-cpp11/]]
  * [fl[You Can Do Any Kind of Atomic Read-Modify-Write Operation|https://preshing.com/20150402/you-can-do-any-kind-of-atomic-read-modify-write-operation/]]

* CPPReference:
  * [fl[Memory Order|https://en.cppreference.com/w/c/atomic/memory_order]]
  * [fl[Atomic Types|https://en.cppreference.com/w/c/language/atomic]]

* Bruce Dawson's [fl[Lockless Programming Considerations|https://docs.microsoft.com/en-us/windows/win32/dxtecharts/lockless-programming]]

* Những người nhiệt tình và am hiểu trên [fl[r/C_Programming|https://www.reddit.com/r/C_Programming/]]

[i[Atomic variables]>]

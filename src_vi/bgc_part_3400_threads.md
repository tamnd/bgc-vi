<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Đa luồng (Multithreading)

[i[Multithreading]<]

C11 chính thức đưa đa luồng vào ngôn ngữ C. Nó giống kỳ lạ với
[flw[POSIX threads|POSIX_Threads]] nếu bạn đã từng dùng thứ đó.

Còn nếu chưa, đừng lo. Ta sẽ đi qua từng bước.

Lưu ý là tôi không định làm hướng dẫn đa luồng đầy đủ kiểu cổ
điển^[Bản thân tôi thích kiểu shared-nothing hơn, và kỹ năng của tôi
với mấy construct đa luồng cổ điển nói nhẹ là đã cũ.]; bạn phải tìm
một cuốn sách thật dày khác dành riêng cho chuyện đó. Xin lỗi nhé!

[i[`__STDC_NO_THREADS__` macro]<]

Threads là tính năng tuỳ chọn. Nếu compiler C11+ định nghĩa
`__STDC_NO_THREADS__`, threads sẽ **không** có trong thư viện. Tại
sao họ quyết định đi theo nghĩa phủ định trong cái macro đó thì tôi
chịu, nhưng nó là vậy.

Bạn có thể kiểm tra nó như vầy:

``` {.c}
#ifdef __STDC_NO_THREADS__
#error I need threads to build this program!
#endif
```

[i[`__STDC_NO_THREADS__` macro]>]

[i[`gcc` compiler-->with threads]<]

Ngoài ra, bạn có thể cần chỉ định tuỳ chọn linker khi build. Trong
trường hợp hệ Unix-like, thử thêm `-lpthreads` vào cuối dòng lệnh để
link thư viện `pthreads`^[Đúng, `pthreads` với "`p`". Viết tắt cho
POSIX threads, thư viện mà C11 đã vay mượn rất nhiều cho cách hiện
thực threads của nó.]:

``` {.zsh}
gcc -std=c11 -o foo foo.c -lpthreads
```

[i[`gcc` compiler-->with threads]>]

Nếu bạn gặp lỗi linker trên hệ thống, có thể là do thư viện phù hợp
không được include.


## Bối cảnh

Threads là cách để bạn dùng tất cả các CPU core bóng loáng mà bạn đã
trả tiền cùng làm việc cho bạn trong cùng một chương trình.

Bình thường một chương trình C chỉ chạy trên một CPU core. Nhưng
nếu bạn biết chia công việc ra, bạn có thể đưa từng phần cho một số
threads và để chúng làm song song.

Dù spec không nói, rất có khả năng trên hệ thống của bạn, C (hay OS
thay mặt nó) sẽ cố cân bằng threads trên tất cả các CPU core.

Và nếu bạn có nhiều threads hơn cores, không sao. Chỉ là bạn sẽ
không nhận được tất cả lợi ích đó nếu chúng đều cạnh tranh thời gian
CPU.

## Những thứ bạn làm được

Bạn có thể tạo một thread. Nó sẽ bắt đầu chạy hàm bạn chỉ định.
Thread cha tạo ra nó cũng sẽ tiếp tục chạy.

Và bạn có thể đợi thread kết thúc. Cái này gọi là _joining_.

Hoặc nếu bạn không quan tâm khi nào thread kết thúc và không muốn
đợi, bạn có thể _detach_ nó.

Một thread có thể _exit_ tường minh, hoặc có thể ngầm kết thúc bằng
cách return từ hàm chính của nó.

Một thread cũng có thể _sleep_ một khoảng thời gian, không làm gì
trong khi thread khác chạy.

Chương trình `main()` cũng là một thread.

Ngoài ra, ta có thread local storage, mutexes, và condition
variables. Nhưng mấy cái đó để sau. Giờ cứ xem phần cơ bản đã.

## Data Race và thư viện chuẩn

[i[Multithreading-->and the standard library]<]

Một số hàm trong thư viện chuẩn (ví dụ `asctime()` và `strtok()`)
trả về hoặc dùng các phần tử dữ liệu `static` không threadsafe. Nhưng
nhìn chung trừ khi có nói khác, thư viện chuẩn cố gắng làm nó
threadsafe^[Theo §7.1.4¶5.].

Nhưng để ý nhé. Nếu một hàm thư viện chuẩn giữ trạng thái giữa các
lần gọi trong một biến bạn không sở hữu, hay một hàm trả về con trỏ
tới thứ gì đó mà bạn không truyền vào, thì nó không threadsafe.

[i[Multithreading-->and the standard library]>]

## Tạo và đợi Threads

Hack gì đó lên thôi!

Ta sẽ tạo vài threads và đợi chúng hoàn thành (join).

Có vài thứ cần hiểu trước đã.

Mỗi thread được xác định bằng một biến mờ (opaque) kiểu
[i[`thrd_t` type]] `thrd_t`. Đây là ID duy nhất của từng thread
trong chương trình. Khi bạn tạo thread, nó được cấp ID mới.

Cũng vậy khi bạn tạo thread, bạn phải đưa cho nó con trỏ đến một hàm
để chạy, và một con trỏ đến tham số để truyền cho nó (hoặc `NULL`
nếu không có gì để truyền).

Thread sẽ bắt đầu thực thi ở hàm bạn chỉ định.

Khi bạn muốn đợi một thread hoàn thành, bạn phải chỉ định thread ID
của nó để C biết đợi cái nào.

Nên ý tưởng cơ bản là:

1. Viết một hàm đóng vai "`main`" của thread. Không phải `main()`
   chính gốc, nhưng tương tự. Thread sẽ bắt đầu chạy ở đó.
2. Từ thread chính, launch thread mới với [i[`thrd_create()`
   function]]`thrd_create()`, và truyền cho nó con trỏ đến hàm cần
   chạy.
3. Trong hàm đó, cho thread làm bất cứ gì nó phải làm.
4. Cùng lúc đó, thread chính có thể tiếp tục làm bất cứ gì _nó_ cần
   làm.
5. Khi thread chính quyết định, nó có thể đợi thread con hoàn thành
   bằng cách gọi [i[`thrd_join()` function]] `thrd_join()`. Thường
   thì bạn **phải** `thrd_join()` thread để dọn dẹp nó nếu không bạn
   sẽ leak bộ nhớ^[Trừ khi bạn `thrd_detach()`. Sẽ nói thêm sau.]

[i[`thrd_create()` function]<]
[i[`thrd_start_t` type]<]

`thrd_create()` nhận con trỏ đến hàm cần chạy, và nó có kiểu
`thrd_start_t`, tức là `int (*)(void *)`. Đó là tiếng Hy Lạp cho
"con trỏ đến một hàm nhận `void*` làm tham số và trả về `int`."

[i[`thrd_join()` function]<]

Hãy tạo một thread! Ta sẽ launch nó từ thread chính bằng
`thrd_create()` để chạy một hàm, làm vài thứ khác, rồi đợi nó hoàn
thành với `thrd_join()`. Tôi đặt tên hàm chính của thread là `run()`,
nhưng bạn có thể đặt tên gì cũng được miễn kiểu khớp với
`thrd_start_t`.

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

// This is the function the thread will run. It can be called anything.
//
// arg is the argument pointer passed to `thrd_create()`.
//
// The parent thread will get the return value back from `thrd_join()`'
// later.

int run(void *arg)
{
    int *a = arg;  // We'll pass in an int* from thrd_create()

    printf("THREAD: Running thread with arg %d\n", *a);

    return 12;  // Value to be picked up by thrd_join() (chose 12 at random)
}

int main(void)
{
    thrd_t t;  // t will hold the thread ID
    int arg = 3490;

    printf("Launching a thread\n");

    // Launch a thread to the run() function, passing a pointer to 3490
    // as an argument. Also stored the thread ID in t:

    thrd_create(&t, run, &arg);

    printf("Doing other things while the thread runs\n");

    printf("Waiting for thread to complete...\n");

    int res;  // Holds return value from the thread exit

    // Wait here for the thread to complete; store the return value
    // in res:

    thrd_join(t, &res);

    printf("Thread exited with return value %d\n", res);
}
```

[i[`thrd_start_t` type]>]

Thấy cách ta làm `thrd_create()` ở đó để gọi hàm `run()` không?
Rồi ta làm những việc khác trong `main()` rồi dừng và đợi thread
hoàn thành với `thrd_join()`.

[i[`thrd_create()` function]>]
[i[`thrd_join()` function]>]

Kết quả mẫu (của bạn có thể khác):

``` {.default}
Launching a thread
Doing other things while the thread runs
Waiting for thread to complete...
THREAD: Running thread with arg 3490
Thread exited with return value 12
```

`arg` bạn truyền cho hàm phải có lifetime đủ dài để thread có thể
lấy nó trước khi nó biến mất. Và nó cần không bị thread chính ghi đè
trước khi thread mới kịp dùng.

Xem ví dụ launch 5 threads. Một thứ cần lưu ý ở đây là cách ta dùng
mảng `thrd_t` để theo dõi tất cả thread ID.

[i[`thrd_create()` function]<]
[i[`thrd_join()` function]<]

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

int run(void *arg)
{
    int i = *(int*)arg;

    printf("THREAD %d: running!\n", i);

    return i;
}

#define THREAD_COUNT 5

int main(void)
{
    thrd_t t[THREAD_COUNT];

    int i;

    printf("Launching threads...\n");
    for (i = 0; i < THREAD_COUNT; i++)

        // NOTE! In the following line, we pass a pointer to i, 
        // but each thread sees the same pointer. So they'll
        // print out weird things as i changes value here in
        // the main thread! (More in the text, below.)

        thrd_create(t + i, run, &i);

    printf("Doing other things while the thread runs...\n");
    printf("Waiting for thread to complete...\n");

    for (int i = 0; i < THREAD_COUNT; i++) {
        int res;
        thrd_join(t[i], &res);

        printf("Thread %d complete!\n", res);
    }

    printf("All threads complete!\n");
}
```

[i[`thrd_join()` function]>]

Khi tôi chạy các threads, tôi đếm `i` từ 0 đến 4 và truyền con trỏ
tới nó cho `thrd_create()`. Con trỏ này đi tới hàm `run()` nơi ta
tạo một bản sao.

[i[`thrd_create()` function]>]

Đủ đơn giản chưa? Đây là output:

``` {.default}
Launching threads...
THREAD 2: running!
THREAD 3: running!
THREAD 4: running!
THREAD 2: running!
Doing other things while the thread runs...
Waiting for thread to complete...
Thread 2 complete!
Thread 2 complete!
THREAD 5: running!
Thread 3 complete!
Thread 4 complete!
Thread 5 complete!
All threads complete!
```

Cái giiiiiì? `THREAD 0` đâu rồi? Và sao có `THREAD 5` khi rõ ràng
`i` không bao giờ lớn hơn `4` lúc gọi `thrd_create()`? Và hai
`THREAD 2`? Điên rồi!

Đây là đang bước vào vùng đất vui vẻ của [i[Multithreading-->race
conditions]] _race conditions_. Thread chính đang sửa `i` trước khi
thread có cơ hội copy nó. Thực ra `i` đi tới tận `5` và kết thúc
vòng lặp trước khi thread cuối cùng có cơ hội copy nó.

Ta cần có biến per-thread để tham chiếu sao cho có thể truyền vào
làm `arg`.

Có thể có mảng lớn. Hoặc có thể `malloc()` chỗ (và free nó ở đâu đó,
có thể trong chính thread.)

Thử vậy xem:

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

int run(void *arg)
{
    int i = *(int*)arg;  // Copy the arg

    free(arg);  // Done with this

    printf("THREAD %d: running!\n", i);

    return i;
}

#define THREAD_COUNT 5

int main(void)
{
    thrd_t t[THREAD_COUNT];

    int i;

    printf("Launching threads...\n");
    for (i = 0; i < THREAD_COUNT; i++) {

        // Get some space for a per-thread argument:

        int *arg = malloc(sizeof *arg);
        *arg = i;

        thrd_create(t + i, run, arg);
    }

    // ...
```

Chú ý trên dòng 27-30 ta `malloc()` chỗ cho một `int` và copy giá
trị của `i` vào đó. Mỗi thread mới nhận biến `malloc()`-mới-toanh
của riêng mình và ta truyền con trỏ tới nó cho hàm `run()`.

Khi `run()` tạo bản copy của `arg` ở dòng 7, nó `free()` cái `int`
đã `malloc()`. Và giờ nó có bản sao riêng, muốn làm gì thì làm.

Và chạy cho thấy kết quả:

``` {.default}
Launching threads...
THREAD 0: running!
THREAD 1: running!
THREAD 2: running!
THREAD 3: running!
Doing other things while the thread runs...
Waiting for thread to complete...
Thread 0 complete!
Thread 1 complete!
Thread 2 complete!
Thread 3 complete!
THREAD 4: running!
Thread 4 complete!
All threads complete!
```

Đấy! Threads 0-4 đều có mặt!

Lần chạy của bạn có thể khác, thread được schedule chạy thế nào nằm
ngoài C spec. Ta thấy ví dụ trên thread 4 không thậm chí bắt đầu đến
khi threads 0-1 đã xong. Thực tế, nếu chạy lại tôi có thể ra output
khác. Ta không thể đảm bảo thứ tự thực thi thread.

## Detach Threads

Nếu bạn muốn fire-and-forget một thread (tức là bạn không phải
`thrd_join()` nó sau này), bạn làm được với `thrd_detach()`.

Cái này loại bỏ khả năng của thread cha lấy return value từ thread
con, nhưng nếu bạn không quan tâm và chỉ muốn threads tự dọn dẹp
đẹp đẽ, thì đây là cách đi.

Về cơ bản ta sẽ làm vầy:

[i[`thrd_detach()` function]<]

``` {.c}
thrd_create(&t, run, NULL);
thrd_detach(t);
```

chỗ gọi `thrd_detach()` là thread cha nói, "Này, tôi sẽ không đợi
thread con này hoàn thành với `thrd_join()`. Nên cứ tự dọn dẹp nó
khi xong nhé."

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

int run(void *arg)
{
    (void)arg;

    //printf("Thread running! %lu\n", thrd_current()); // non-portable!
    printf("Thread running!\n");

    return 0;
}

#define THREAD_COUNT 10

int main(void)
{
    thrd_t t;

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_create(&t, run, NULL);
        thrd_detach(t);               // <-- DETACH!
    }

    // Sleep for a second to let all the threads finish
    thrd_sleep(&(struct timespec){.tv_sec=1}, NULL);
}
```

[i[`thrd_detach()` function]>]

Lưu ý trong code này ta cho thread chính sleep 1 giây bằng
`thrd_sleep()`, nói thêm sau.

Ngoài ra trong hàm `run()`, tôi có dòng comment-out in thread ID
dưới dạng `unsigned long`. Cái này không portable, vì spec không nói
kiểu bên dưới của `thrd_t` là gì, nó có thể là một `struct` ai biết.
Nhưng dòng đó chạy được trên máy tôi.

Một điều thú vị tôi thấy khi chạy code trên và in thread ID là vài
threads có ID trùng nhau! Tưởng chừng là không thể, nhưng C cho
phép _reuse_ thread ID sau khi thread tương ứng đã thoát. Nên cái
tôi thấy là vài threads đã xong trước khi threads khác được launch.

## Dữ liệu cục bộ theo Thread

[i[Thread local data]<]

Threads thú vị vì chúng không có bộ nhớ riêng ngoài biến cục bộ. Nếu
bạn muốn biến `static` hay biến phạm vi file, tất cả threads sẽ thấy
cùng biến đó.

Cái này có thể dẫn tới race conditions, nơi bạn gặp _Chuyện Lạ_™
xảy ra.

Xem ví dụ này. Ta có biến `static` `foo` trong block scope ở `run()`.
Biến này sẽ thấy được bởi tất cả threads đi qua hàm `run()`. Và các
threads thực sự có thể giẫm chân nhau.

Mỗi thread copy `foo` vào biến cục bộ `x` (không chia sẻ giữa
threads, tất cả threads có call stack riêng). Nên chúng _nên_ bằng
nhau, đúng không?

Và lần đầu in ra, chúng bằng^[Dù tôi không nghĩ chúng phải vậy. Chỉ
là threads có vẻ không được reschedule cho đến khi xảy ra system
call nào đó như `printf()`... đó là lý do tôi để `printf()` trong
đó.]. Nhưng rồi ngay sau đó, ta kiểm tra xem chúng có còn bằng
không.

Và _thường thì_ bằng. Nhưng không phải luôn luôn!

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

int run(void *arg)
{
    int n = *(int*)arg;  // Thread number for humans to differentiate

    free(arg);

    static int foo = 10;  // Static value shared between threads

    int x = foo;  // Automatic local variable--each thread has its own

    // We just assigned x from foo, so they'd better be equal here.
    // (In all my test runs, they were, but even this isn't guaranteed!)

    printf("Thread %d: x = %d, foo = %d\n", n, x, foo);

    // And they should be equal here, but they're not always!
    // (Sometimes they were, sometimes they weren't!)

    // What happens is another thread gets in and increments foo
    // right now, but this thread's x remains what it was before!

    if (x != foo) {
        printf("Thread %d: Craziness! x != foo! %d != %d\n", n, x, foo);
    }

    foo++;  // Increment shared value

    return 0;
}

#define THREAD_COUNT 5

int main(void)
{
    thrd_t t[THREAD_COUNT];

    for (int i = 0; i < THREAD_COUNT; i++) {
        int *n = malloc(sizeof *n);  // Holds a thread serial number
        *n = i;
        thrd_create(t + i, run, n);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }
}
```

Đây là một output mẫu (thay đổi giữa các lần chạy):

``` {.default}
Thread 0: x = 10, foo = 10
Thread 1: x = 10, foo = 10
Thread 1: Craziness! x != foo! 10 != 11
Thread 2: x = 12, foo = 12
Thread 4: x = 13, foo = 13
Thread 3: x = 14, foo = 14
```

Trong thread 1, giữa hai lần `printf()`, giá trị `foo` bằng cách nào
đó đổi từ `10` thành `11`, dù rõ ràng không có increment nào giữa
hai `printf()`!

Đó là một thread khác chen vào (có vẻ là thread 0 theo hình thù)
và increment `foo` sau lưng thread 1!

Hãy giải quyết vấn đề này theo hai cách khác nhau. (Nếu bạn muốn tất
cả threads cùng dùng chung biến _và_ không giẫm chân nhau, bạn phải
đọc tiếp tới phần [mutex](#mutex).)

### Storage-Class `_Thread_local` {#thread-local}

[i[`_Thread_local` storage class]<]

Trước hết, cứ nhìn cách dễ nhất để đi vòng: storage-class
`_Thread_local`.

Về cơ bản ta chỉ dán nó vào trước biến `static` block scope và
chuyện sẽ chạy! Nó bảo C rằng mỗi thread nên có phiên bản riêng của
biến này, nên không có thread nào giẫm chân nhau.

[i[`thread_local` storage class]<]

File header [i[`threads.h` header file]] `<threads.h>` định nghĩa
`thread_local` là alias của `_Thread_local` nên code bạn khỏi xấu.

Lấy ví dụ trước và biến `foo` thành biến `thread_local` để ta không
chia sẻ dữ liệu.

``` {.c .numberLines startFrom="5"}
int run(void *arg)
{
    int n = *(int*)arg;  // Thread number for humans to differentiate

    free(arg);

    thread_local static int foo = 10;  // <-- No longer shared!!
```

Và chạy ta được:

``` {.default}
Thread 0: x = 10, foo = 10
Thread 1: x = 10, foo = 10
Thread 2: x = 10, foo = 10
Thread 4: x = 10, foo = 10
Thread 3: x = 10, foo = 10
```

Không còn rắc rối lạ nữa!

Một điều: nếu biến `thread_local` là block scope, nó **phải**
`static`. Luật là vậy. (Nhưng vẫn OK vì biến không-`static` đã là
per-thread rồi do mỗi thread có biến không-`static` riêng.)

Hơi nói dối chút: biến `thread_local` block scope còn có thể
`extern`.

[i[`thread_local` storage class]>]
[i[`_Thread_local` storage class]>]
[i[Thread local data]>]

### Một lựa chọn khác: Thread-Specific Storage

[i[Thread-specific storage]<]

Thread-specific storage (TSS) là một cách khác để có dữ liệu
per-thread.

Một tính năng thêm là các hàm này cho phép bạn chỉ định destructor
sẽ được gọi trên dữ liệu khi biến TSS bị xoá. Thông thường destructor
là `free()` để tự động dọn dữ liệu `malloc()` per-thread. Hoặc
`NULL` nếu không cần destroy gì.

Destructor có kiểu [i[`tss_dtor_t` type]] `tss_dtor_t` là con trỏ
đến hàm trả về `void` và nhận `void*` làm tham số (cái `void*` trỏ
tới dữ liệu lưu trong biến). Nói cách khác, nó là `void (*)(void*)`,
nếu có rõ thêm. Tôi thừa nhận chắc là không. Xem ví dụ bên dưới.

Nói chung `thread_local` chắc là lựa chọn mặc định, nhưng nếu bạn
thích ý tưởng destructor thì tận dụng.

Cách dùng hơi lạ ở chỗ ta cần biến kiểu [i[`tss_t` type]] `tss_t`
sống để đại diện giá trị trên cơ sở per-thread. Rồi ta khởi tạo nó
với [i[`tss_create()` function]] `tss_create()`. Cuối cùng xoá với
[i[`tss_delete()` function]<] `tss_delete()`. Chú ý gọi
`tss_delete()` không chạy tất cả destructors, chính `thrd_exit()`
(hoặc return từ hàm run) mới làm. `tss_delete()` chỉ giải phóng bộ
nhớ do `tss_create()` cấp phát. [i[`tss_delete()` function]>]

Ở giữa, threads có thể gọi [i[`tss_set()` function]] `tss_set()` và
[i[`tss_get()` function]] `tss_get()` để đặt và lấy giá trị.

Trong code sau, ta set up biến TSS trước khi tạo threads, rồi dọn
dẹp sau khi threads xong.

Trong hàm `run()`, threads `malloc()` chỗ cho một chuỗi và lưu con
trỏ đó vào biến TSS.

Khi thread thoát, hàm destructor (`free()` trong trường hợp này)
được gọi cho _tất cả_ threads.

[i[`tss_t` type]<]
[i[`tss_get()` function]<]
[i[`tss_set()` function]<]
[i[`tss_create()` function]<]
[i[`tss_delete()` function]<]

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

tss_t str;

void some_function(void)
{
    // Retrieve the per-thread value of this string
    char *tss_string = tss_get(str);

    // And print it
    printf("TSS string: %s\n", tss_string);
}

int run(void *arg)
{
    int serial = *(int*)arg;  // Get this thread's serial number
    free(arg);

    // malloc() space to hold the data for this thread
    char *s = malloc(64);
    sprintf(s, "thread %d! :)", serial);  // Happy little string

    // Set this TSS variable to point at the string
    tss_set(str, s);

    // Call a function that will get the variable
    some_function();

    return 0;   // Equivalent to thrd_exit(0)
}

#define THREAD_COUNT 15

int main(void)
{
    thrd_t t[THREAD_COUNT];

    // Make a new TSS variable, the free() function is the destructor
    tss_create(&str, free);

    for (int i = 0; i < THREAD_COUNT; i++) {
        int *n = malloc(sizeof *n);  // Holds a thread serial number
        *n = i;
        thrd_create(t + i, run, n);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }

    // All threads are done, so we're done with this
    tss_delete(str);
}
```

[i[`tss_t` type]>]
[i[`tss_get()` function]>]
[i[`tss_set()` function]>]
[i[`tss_create()` function]>]
[i[`tss_delete()` function]>]

Lại nữa, đây là cách làm khá đau so với `thread_local`, nên trừ khi
bạn thực sự cần chức năng destructor, tôi sẽ dùng `thread_local`.

[i[Thread-specific storage]>]

## Mutexes {#mutex}

[i[Mutexes]<]

Nếu bạn chỉ muốn cho một thread vào critical section của code tại
một thời điểm, bạn có thể bảo vệ đoạn đó bằng mutex^[Viết tắt của
"mutual exclusion", cũng gọi là "lock" trên một đoạn code mà chỉ một
thread được phép thực thi.].

Ví dụ, nếu ta có biến `static` và muốn có thể lấy và set nó bằng hai
thao tác mà không thread khác nhảy vào giữa làm hỏng, ta có thể dùng
mutex.

Bạn có thể acquire mutex hay release nó. Nếu bạn cố acquire mutex
và thành công, bạn có thể tiếp tục thực thi. Nếu cố mà thất bại (vì
người khác đang giữ), bạn sẽ _block_^[Tức là process của bạn sẽ đi
ngủ.] đến khi mutex được release.

Nếu nhiều threads bị block đợi một mutex được release, một trong
chúng sẽ được chọn để chạy (ngẫu nhiên từ góc nhìn của ta), còn các
thread khác tiếp tục ngủ.

Kế hoạch là đầu tiên ta sẽ khởi tạo biến mutex để sẵn sàng dùng bằng
[i[`mtx_init()` function]] `mtx_init()`.

Rồi các threads tiếp theo có thể gọi [i[`mtx_lock()` function]]
`mtx_lock()` và [i[`mtx_unlock` function]] `mtx_unlock()` để lấy và
release mutex.

Khi ta đã hoàn toàn xong với mutex, có thể destroy với
[i[`mtx_destroy()` function]] `mtx_destroy()`, đối lập logic của
[i[`mtx_init()` function]] `mtx_init()`.

[i[Multithreading-->race conditions]<]

Đầu tiên xem code _không_ dùng mutex và cố in ra serial number chia
sẻ (`static`) rồi tăng nó. Vì ta không dùng mutex trên việc lấy giá
trị (để in) và set (để tăng), threads có thể giẫm chân nhau trong
critical section đó.

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

int run(void *arg)
{
    (void)arg;

    static int serial = 0;   // Shared static variable!

    printf("Thread running! %d\n", serial);

    serial++;

    return 0;
}

#define THREAD_COUNT 10

int main(void)
{
    thrd_t t[THREAD_COUNT];

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_create(t + i, run, NULL);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }
}
```

Khi tôi chạy, tôi được cái gì đó như:

``` {.default}
Thread running! 0
Thread running! 0
Thread running! 0
Thread running! 3
Thread running! 4
Thread running! 5
Thread running! 6
Thread running! 7
Thread running! 8
Thread running! 9
```

Rõ ràng nhiều threads chen vào chạy `printf()` trước khi ai đó có cơ
hội cập nhật biến `serial`.

[i[Multithreading-->race conditions]>]

Cái ta muốn làm là bọc việc lấy biến và set biến thành một đoạn code
được mutex bảo vệ duy nhất.

Ta sẽ thêm biến mới đại diện mutex kiểu [i[`mtx_t` type]] `mtx_t`
trong phạm vi file, khởi tạo, rồi threads có thể lock và unlock nó
trong hàm `run()`.

[i[`mtx_lock()` function]<]
[i[`mtx_unlock()` function]<]
[i[`mtx_init()` function]<]
[i[`mtx_destroy()` function]<]

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

mtx_t serial_mtx;     // <-- MUTEX VARIABLE

int run(void *arg)
{
    (void)arg;

    static int serial = 0;   // Shared static variable!

    // Acquire the mutex--all threads will block on this call until
    // they get the lock:

    mtx_lock(&serial_mtx);           // <-- ACQUIRE MUTEX

    printf("Thread running! %d\n", serial);

    serial++;

    // Done getting and setting the data, so free the lock. This will
    // unblock threads on the mtx_lock() call:

    mtx_unlock(&serial_mtx);         // <-- RELEASE MUTEX

    return 0;
}

#define THREAD_COUNT 10

int main(void)
{
    thrd_t t[THREAD_COUNT];

    // Initialize the mutex variable, indicating this is a normal
    // no-frills, mutex:

    mtx_init(&serial_mtx, mtx_plain);        // <-- CREATE MUTEX

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_create(t + i, run, NULL);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }

    // Done with the mutex, destroy it:

    mtx_destroy(&serial_mtx);                // <-- DESTROY MUTEX
}
```

Xem cách dòng 38 và 50 trong `main()` ta khởi tạo và destroy mutex.

[i[`mtx_init()` function]>]
[i[`mtx_destroy()` function]>]

Nhưng từng thread acquire mutex ở dòng 15 và release ở dòng 24.

Giữa `mtx_lock()` và `mtx_unlock()` là _critical section_, vùng
code mà ta không muốn nhiều threads lộn xộn cùng lúc.

[i[`mtx_lock()` function]>]
[i[`mtx_unlock()` function]>]

Và giờ ta có output đúng!

``` {.default}
Thread running! 0
Thread running! 1
Thread running! 2
Thread running! 3
Thread running! 4
Thread running! 5
Thread running! 6
Thread running! 7
Thread running! 8
Thread running! 9
```

Nếu bạn cần nhiều mutexes, không vấn đề: cứ có nhiều biến mutex.

Và nhớ Quy Tắc Số Một của Nhiều Mutex: _Unlock mutexes theo thứ tự
ngược với thứ tự lock!_

### Các kiểu Mutex khác nhau

[i[Mutexes-->types]<]

Như đã gợi ý, ta có vài kiểu mutex tạo được với `mtx_init()`. (Một
số kiểu là kết quả của phép bitwise-OR, như ghi trong bảng.)

[i[Mutexes-->timeouts]<]

[i[`mtx_plain` macro]]
[i[`mtx_timed` macro]]
[i[`mtx_plain` macro]]
[i[`mtx_timed` macro]]
[i[`mtx_recursive` macro]]

|Kiểu|Mô tả|
|-|-|
|`mtx_plain`|Mutex bình thường|
|`mtx_timed`|Mutex hỗ trợ timeouts|
|`mtx_plain|mtx_recursive`|Mutex đệ quy|
|`mtx_timed|mtx_recursive`|Mutex đệ quy hỗ trợ timeouts|

"Đệ quy" nghĩa là người giữ lock có thể gọi `mtx_lock()` nhiều lần
trên cùng một lock. (Họ phải unlock một số lần bằng nhau trước khi
ai khác có thể lấy mutex.) Cái này có thể làm code dễ hơn đôi khi,
đặc biệt khi bạn gọi một hàm cần lock mutex trong khi bạn đã giữ
mutex.

Và timeout cho thread cơ hội _cố_ lấy lock một lúc, nhưng rồi bỏ
cuộc nếu không lấy được trong khung thời gian đó.

[i[`mtx_timed` macro]<]

Với mutex timeout, nhớ tạo với `mtx_timed`:

``` {.c}
mtx_init(&serial_mtx, mtx_timed);
```

[i[`mtx_timed` macro]>]

Rồi khi bạn đợi nó, bạn phải chỉ định thời gian theo UTC khi nào
unlock^[Bạn có thể đã nghĩ đó là "thời gian kể từ bây giờ", nhưng
bạn chỉ muốn nghĩ vậy thôi phải không!].

[i[`timespec_get()` function]<]

Hàm `timespec_get()` từ `<time.h>` có thể trợ giúp. Nó cho bạn thời
gian hiện tại theo UTC trong `struct timespec` đúng cái ta cần. Thực
tế có vẻ nó tồn tại chỉ vì mục đích này.

Nó có hai field: `tv_sec` giữ thời gian hiện tại tính bằng giây kể
từ epoch, và `tv_nsec` giữ nanosecond (phần tỉ của giây) như phần
"fractional".

Nên bạn có thể load nó với thời gian hiện tại, rồi cộng thêm để có
timeout cụ thể.

[i[`mtx_timedlock()` function]<]

Rồi gọi `mtx_timedlock()` thay vì `mtx_lock()`. Nếu nó trả về giá
trị `thrd_timedout`, là đã timeout.

``` {.c}
struct timespec timeout;

timespec_get(&timeout, TIME_UTC);  // Get current time
timeout.tv_sec += 1;               // Timeout 1 second after now

int result = mtx_timedlock(&serial_mtx, &timeout));

if (result == thrd_timedout) {
    printf("Mutex lock timed out!\n");
}
```

[i[`mtx_timedlock()` function]>]
[i[`timespec_get()` function]>]

Ngoài cái đó ra, timed lock giống regular lock.

[i[Mutexes-->timeouts]>]
[i[Mutexes-->types]>]
[i[Mutexes]>]

## Condition Variables

[i[Condition variables]<]

Condition Variables là mảnh ghép cuối ta cần để làm các ứng dụng
đa luồng có hiệu năng và compose các cấu trúc đa luồng phức tạp hơn.

Condition variable cung cấp cách để threads đi ngủ cho đến khi một
event nào đó trên thread khác xảy ra.

Nói cách khác, ta có thể có một số threads đã sẵn sàng chạy, nhưng
phải đợi đến khi event nào đó là true trước khi tiếp tục. Về cơ bản
chúng được bảo "đợi đấy!" đến khi được thông báo.

Và cái này đi đôi với mutex vì cái ta đợi thường phụ thuộc vào giá
trị của dữ liệu nào đó, và dữ liệu đó thường cần được mutex bảo vệ.

Quan trọng là bản thân condition variable không phải người giữ dữ
liệu cụ thể nào từ góc nhìn của ta. Nó chỉ là biến qua đó C theo dõi
trạng thái waiting/not-waiting của một thread hay nhóm threads cụ
thể.

Hãy viết một chương trình giả tạo đọc vào nhóm 5 số từ thread chính
một lần một số. Rồi khi 5 số đã nhập, thread con thức dậy, cộng 5
số đó, và in kết quả.

Các số sẽ được lưu trong mảng chia sẻ toàn cục, cũng như index của
mảng cho số sắp được nhập.

Vì đây là giá trị chia sẻ, ta ít nhất phải giấu chúng sau mutex cho
cả thread chính và thread con. (Chính sẽ ghi dữ liệu vào, còn con
sẽ đọc dữ liệu ra.)

Nhưng chừng đó chưa đủ. Thread con cần block ("ngủ") đến khi 5 số
đã được đọc vào mảng. Rồi thread cha cần đánh thức thread con để nó
làm việc.

Và khi thức dậy, nó cần đang giữ mutex đó. Và sẽ vậy! Khi một thread
đợi trên condition variable, nó cũng acquire mutex khi thức dậy.

Tất cả cái này xảy ra quanh một biến thêm kiểu [i[`cnd_t` type]]
`cnd_t` chính là _condition variable_. Ta tạo biến này bằng hàm
[i[`cnd_init()` function]] `cnd_init()` và destroy khi xong với hàm
[i[`cnd_destroy()` function]] `cnd_destroy()`.

Nhưng hoạt động thế nào? Xem phác thảo những gì thread con sẽ làm:

[i[`mtx_lock()` function]<]
[i[`mtx_unlock()` function]<]
[i[`cnd_wait()` function]<]
[i[`cnd_signal()` function]<]

1. Lock mutex với `mtx_lock()`
2. Nếu chưa nhập hết số, đợi trên condition variable với
   `cnd_wait()`
3. Làm việc cần làm
4. Unlock mutex với `mtx_unlock()`

Cùng lúc thread chính sẽ làm:

1. Lock mutex với `mtx_lock()`
2. Lưu số vừa đọc vào mảng
3. Nếu mảng đầy, signal con thức dậy với `cnd_signal()`
4. Unlock mutex với `mtx_unlock()`

[i[`mtx_lock()` function]>]
[i[`mtx_unlock()` function]>]

Nếu bạn không đọc lướt quá (OK, tôi không giận), bạn có thể nhận ra
điều lạ: làm sao thread chính giữ lock mutex và signal con, nếu con
phải giữ lock mutex để đợi signal? Cả hai không cùng giữ được!

Và đúng vậy! Có tí ma thuật sau cánh gà với condition variable: khi
bạn `cnd_wait()`, nó release mutex bạn chỉ định và thread đi ngủ. Và
khi ai đó signal thread thức dậy, nó reacquire lock như không có gì
xảy ra.

Bên `cnd_signal()` thì hơi khác. Cái này không làm gì với mutex.
Thread signal vẫn phải tự release mutex trước khi threads đang đợi
thức dậy được.

[i[`cnd_signal()` function]>]

Một điều nữa về `cnd_wait()`. Bạn sẽ gọi `cnd_wait()` nếu điều kiện
nào đó^[Và đó là lý do gọi là _condition variables_!] chưa đạt (ví
dụ trong trường hợp này, nếu chưa nhập hết số). Thoả thuận là: điều
kiện này nên trong `while` loop, không phải `if`. Tại sao?

Vì một hiện tượng bí ẩn gọi là [i[Condition variables-->spurious
wakeup]] _spurious wakeup_. Đôi khi, trong vài implementation, một
thread có thể bị đánh thức từ giấc ngủ `cnd_wait()` mà dường như
_không có lý do_. _[nhạc X-Files]_^[Tôi không nói là người ngoài
hành tinh... nhưng là người ngoài hành tinh. OK, thực tế nhiều khả
năng hơn là một thread khác đã được đánh thức và làm được việc
trước.]. Và ta phải kiểm tra xem điều kiện ta cần có còn thực sự
đúng khi thức dậy không. Nếu không, đi ngủ lại!

[i[`cnd_wait()` function]>]

Nào, làm thôi! Bắt đầu với thread chính:

* Thread chính sẽ set up mutex và condition variable, và launch
  thread con.

* Rồi trong vòng lặp vô hạn, lấy số làm input từ console.

* Nó cũng acquire mutex để lưu số đã nhập vào mảng toàn cục.

* Khi mảng có 5 số, thread chính sẽ signal thread con rằng đến lúc
  thức dậy làm việc.

* Rồi thread chính unlock mutex và quay lại đọc số tiếp từ console.

Trong khi đó, thread con làm trò riêng:

* Thread con lấy mutex.

* Trong khi điều kiện chưa đạt (tức mảng chia sẻ chưa có 5 số),
  thread con ngủ bằng cách đợi trên condition variable. Khi đợi, nó
  ngầm unlock mutex.

* Khi thread chính signal thread con thức dậy, nó thức dậy làm việc
  và lấy lại mutex lock.

* Thread con cộng các số và reset biến index vào mảng.

* Rồi release mutex và chạy lại trong vòng lặp vô hạn.

Và đây là code! Nhìn kỹ để thấy chỗ các mảnh trên được xử lý:

[i[`mtx_lock()` function]<]
[i[`mtx_unlock()` function]<]
[i[`mtx_init()` function]<]
[i[`mtx_destroy()` function]<]
[i[`cnd_init()` function]<]
[i[`cnd_destroy()` function]<]
[i[`cnd_wait()` function]<]
[i[`cnd_signal()` function]<]
[i[`cnd_t` type]<]

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

#define VALUE_COUNT_MAX 5

int value[VALUE_COUNT_MAX];  // Shared global
int value_count = 0;   // Shared global, too

mtx_t value_mtx;   // Mutex around value
cnd_t value_cnd;   // Condition variable on value

int run(void *arg)
{
    (void)arg;

    for (;;) {
        mtx_lock(&value_mtx);      // <-- GRAB THE MUTEX

        while (value_count < VALUE_COUNT_MAX) {
            printf("Thread: is waiting\n");
            cnd_wait(&value_cnd, &value_mtx);  // <-- CONDITION WAIT
        }

        printf("Thread: is awake!\n");

        int t = 0;

        // Add everything up
        for (int i = 0; i < VALUE_COUNT_MAX; i++)
            t += value[i];

        printf("Thread: total is %d\n", t);

        // Reset input index for main thread
        value_count = 0;

        mtx_unlock(&value_mtx);   // <-- MUTEX UNLOCK
    }

    return 0;
}

int main(void)
{
    thrd_t t;

    // Spawn a new thread

    thrd_create(&t, run, NULL);
    thrd_detach(t);

    // Set up the mutex and condition variable

    mtx_init(&value_mtx, mtx_plain);
    cnd_init(&value_cnd);

    for (;;) {
        int n;

        scanf("%d", &n);

        mtx_lock(&value_mtx);    // <-- LOCK MUTEX

        value[value_count++] = n;

        if (value_count == VALUE_COUNT_MAX) {
            printf("Main: signaling thread\n");
            cnd_signal(&value_cnd);  // <-- SIGNAL CONDITION
        }

        mtx_unlock(&value_mtx);  // <-- UNLOCK MUTEX
    }

    // Clean up (I know that's an infinite loop above here, but I
    // want to at least pretend to be proper):

    mtx_destroy(&value_mtx);
    cnd_destroy(&value_cnd);
}
```

[i[`mtx_lock()` function]>]
[i[`mtx_unlock()` function]>]
[i[`mtx_init()` function]>]
[i[`mtx_destroy()` function]>]
[i[`cnd_init()` function]>]
[i[`cnd_destroy()` function]>]
[i[`cnd_wait()` function]>]
[i[`cnd_signal()` function]>]
[i[`cnd_t` type]>]

Và đây là output mẫu (các số trên từng dòng là input của tôi):

``` {.default}
Thread: is waiting
1
1
1
1
1
Main: signaling thread
Thread: is awake!
Thread: total is 5
Thread: is waiting
2
8
5
9
0
Main: signaling thread
Thread: is awake!
Thread: total is 24
Thread: is waiting
```

Đây là cách dùng phổ biến của condition variable trong tình huống
producer-consumer như vầy. Nếu ta không có cách cho thread con ngủ
trong khi đợi điều kiện nào đó đạt, nó sẽ buộc phải poll, lãng phí
CPU kinh khủng.

### Timed Condition Wait

[i[Condition variables-->timeouts]<]

Có một biến thể của `cnd_wait()` cho phép bạn chỉ định timeout để
ngừng đợi.

Vì thread con phải relock mutex, cái này không có nghĩa là bạn sẽ
bật ngay trở lại khoảnh khắc timeout xảy ra; bạn vẫn phải đợi các
threads khác release mutex.

Nhưng có nghĩa là bạn sẽ không đợi cho đến khi `cnd_signal()` xảy
ra.

Để dùng, gọi [i[`cnd_timedwait()` function]] `cnd_timedwait()` thay
vì `cnd_wait()`. Nếu nó trả về giá trị [i[`thrd_timedout` macro]]
`thrd_timedout`, là timeout.

Timestamp là thời gian tuyệt đối theo UTC, không phải time-from-now.
May thay hàm [i[`timespec_get()` function]] `timespec_get()` trong
`<time.h>` có vẻ được làm ra đúng cho trường hợp này.

[i[`timespec_get()` function]<]
[i[`cnd_timedwait()` function]<]
[i[`thrd_timedout()` macro]<]

``` {.c}
struct timespec timeout;

timespec_get(&timeout, TIME_UTC);  // Get current time
timeout.tv_sec += 1;               // Timeout 1 second after now

int result = cnd_timedwait(&condition, &mutex, &timeout));

if (result == thrd_timedout) {
    printf("Condition variable timed out!\n");
}
```

[i[`timespec_get()` function]>]
[i[`cnd_timedwait()` function]>]
[i[`thrd_timedout()` macro]>]
[i[Condition variables-->timeouts]>]

### Broadcast: Đánh thức mọi Thread đang đợi

[i[Condition variables-->broadcasting]<]

[i[`cnd_signal()` function]>] `cnd_signal()` chỉ đánh thức một
thread để tiếp tục làm việc. Tuỳ logic bạn làm, có thể hợp lý khi
đánh thức nhiều hơn một thread tiếp tục khi điều kiện đạt.

Tất nhiên chỉ một thread có thể lấy mutex, nhưng nếu bạn có tình
huống:

* Thread mới thức dậy chịu trách nhiệm đánh thức thread kế, và,

* Có khả năng điều kiện loop spurious-wakeup sẽ ngăn nó làm vậy,

thì bạn sẽ muốn broadcast wake up để chắc chắn lấy được ít nhất một
thread ra khỏi loop đó để launch thread tiếp.

Cách làm?

[i[`cnd_broadcast()` function]<]

Đơn giản dùng `cnd_broadcast()` thay vì `cnd_signal()`. Dùng giống
hệt, chỉ khác `cnd_broadcast()` đánh thức **tất cả** sleeping threads
đang đợi trên condition variable đó.

[i[`cnd_broadcast()` function]>]
[i[Condition variables-->broadcasting]>]
[i[Condition variables]>]

## Chạy một hàm đúng một lần

[i[Multithreading-->one-time functions]<]

Giả sử bạn có một hàm _có thể_ được chạy bởi nhiều threads, nhưng
bạn không biết khi nào, và không đáng công viết tất cả logic đó.

Có cách đi vòng: dùng [i[`call_once()` function]] `call_once()`. Cả
đống threads có thể cố chạy hàm, nhưng chỉ thread đầu tiên được
tính^[Sự sống sót của kẻ khoẻ nhất! Đúng không? Tôi thừa nhận thực
ra không giống vậy chút nào.]

Để làm, bạn cần một biến flag đặc biệt bạn khai báo để theo dõi xem
cái đó đã được chạy chưa. Và bạn cần một hàm để chạy, không nhận
tham số và không trả giá trị.

[i[`once_flag` type]<]
[i[`ONCE_FLAG_INIT` macro]<]
[i[`call_once()` function]<]

``` {.c}
once_flag of = ONCE_FLAG_INIT;  // Initialize it like this

void run_once_function(void)
{
    printf("I'll only run once!\n");
}

int run(void *arg)
{
    (void)arg;

    call_once(&of, run_once_function);

    // ...
```

[i[`once_flag` type]>]
[i[`ONCE_FLAG_INIT` macro]>]
[i[`call_once()` function]>]

Trong ví dụ này, không quan trọng bao nhiêu threads tới hàm `run()`,
`run_once_function()` sẽ chỉ được gọi một lần duy nhất.

[i[Multithreading-->one-time functions]>]
[i[Multithreading]>]

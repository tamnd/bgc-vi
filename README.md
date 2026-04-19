# Beej's Guide to C Programming (Bản tiếng Việt)

> Tiếng Việt &middot; [English](README.en.md)

Bản dịch tiếng Việt của [Beej's Guide to C Programming][bgc],
tác giả Brian "Beej Jorgensen" Hall. Đọc miễn phí, chia sẻ thoải mái,
giống hệt như bản gốc.

> Này! C làm bạn khó chịu? Con trỏ, `malloc()`, và chuyện byte-endianness
> làm đầu óc bạn xoay mòng? Muốn viết C tử tế mà sách dày 800 trang
> trông như một cái đe?
>
> Mà đoán xem. Beej đã đi qua đống đó rồi, và giờ có cả bản tiếng Việt.

[bgc]: https://beej.us/guide/bgc/

## Có hợp với tôi không?

Nếu bạn đọc được tiếng Việt và muốn học C từ đầu đến chỗ đủ dùng trong
đời thực, thì hợp. Nếu bạn đọc tiếng Anh thoải mái, cứ
[đọc bản gốc][bgc], nó ở ngay đó thôi.

Repo này dành cho developer người Việt muốn học C nghiêm túc mà không
phải đánh vật với cú pháp tiếng Anh song song với cú pháp C.

## Bạn sẽ học được gì

Hơn bốn mươi chương, đi từ "Hello, World!" đến atomics, multithreading,
Unicode, `setjmp`/`longjmp`, VLA, và cả lô đặc tả hiếm dùng của C. Danh
sách đầy đủ ở [ROADMAP.md](ROADMAP.md).

Vài chương tiêu biểu:

- **Con trỏ** (ba chương, vì con trỏ đáng cả ba chương).
- **Cấp phát bộ nhớ thủ công.** `malloc()`, `free()`, và tại sao.
- **Kiểu dữ liệu.** Năm chương, bao gồm cả qualifier, specifier,
  compound literal, generic selection.
- **Preprocessor C.** Macro là thứ đẹp và nguy hiểm.
- **Multithreading và atomics.** Sau khi bạn đã quen với phần còn lại.
- **Unicode, locale, thời gian.** Những phần ai cũng né nhưng không
  nên.

## Tình trạng

Đường ống build và GitHub Pages đã chạy trước khi dịch bắt đầu, nên
bạn sẽ thấy một website tiếng Anh tại
<https://tamnd.github.io/bgc-vi/> ngay từ ngày đầu. Bản dịch sẽ thay
thế dần từng chương. Theo dõi tiến độ ở [ROADMAP.md](ROADMAP.md) hoặc
xem [các PR đã merge](https://github.com/tamnd/bgc-vi/pulls?q=is%3Apr+is%3Amerged).

## Bố cục repo

```
bgc-vi/
├── src/         # Bản gốc tiếng Anh (lấy từ upstream, không sửa)
├── src_vi/      # Bản dịch tiếng Việt (phần hay ho ở đây)
├── source/      # Chương trình C mẫu (giữ nguyên từ upstream)
├── translations/# Các bản dịch ngôn ngữ khác có sẵn từ upstream
├── website/     # Tài nguyên website của upstream
├── scripts/     # Build helper, release helper
├── Dockerfile.vi # Image build chứa pandoc + texlive + font
├── ROADMAP.md   # Kế hoạch và tiến độ dịch
├── UPSTREAM.md  # Đang bám upstream ở commit nào
├── LICENSE.md   # CC BY-NC-ND 3.0, giống upstream
└── README.md    # Bạn đang ở đây
```

Mỗi chương đã dịch trong `src_vi/` tương ứng một-một với file trong
`src/`, cùng tên file và cùng section anchor. Diff hai file là cách
đơn giản nhất để phát hiện chỗ lệch.

## Cách đọc

**Trên web:** <https://tamnd.github.io/bgc-vi/>, được build lại sau
mỗi lần merge vào `main`.

**Offline:** tải file ở trang
[Releases](https://github.com/tamnd/bgc-vi/releases) (PDF, EPUB,
HTML zip, source code). Hoặc đọc thẳng markdown trong `src_vi/`,
GitHub render vẫn ổn.

**Tự build:** xem [Build](#build) bên dưới.

## Đóng góp

Pull request luôn được chào đón. Vài quy tắc để giữ văn bản dễ đọc:

- **Mỗi PR một chương.** Đừng gộp. PR nhỏ được merge nhanh, PR lớn
  nằm đó.
- **Dịch ý, không dịch chữ.** Nếu bản dịch nguyên văn đọc như máy
  viết, viết lại. Giọng Beej đời thường, giọng bạn cũng nên vậy.
- **Không dịch máy.** Nghiêm túc đấy. Đọc là biết liền.
- **Giữ nguyên code block, tên hàm, tên biến, từ khóa C bằng tiếng
  Anh.** `malloc()` vẫn là `malloc()`. `struct foo` vẫn là `struct foo`.
- **Lần đầu xuất hiện một thuật ngữ kỹ thuật:** viết tiếng Anh trước,
  tiếng Việt trong ngoặc nếu có ích. Các lần sau có thể bỏ tiếng Việt.
- **Không dùng em dash trong văn xuôi tiếng Việt.** Viết lại câu hoặc
  dùng dấu phẩy.
- **Giữ nguyên section anchor.** Tiếng Anh là `{#pointers}` thì tiếng
  Việt cũng `{#pointers}`.

### Quy trình

1. Chọn một chương trong [ROADMAP.md](ROADMAP.md) đang ghi
   "not started".
2. Mở issue nói bạn đang nhận chương đó, tránh trùng việc.
3. Tạo nhánh: `translate/<slug-chương>` (ví dụ `translate/pointers`).
4. Copy `src/bgc_part_NNNN_<slug>.md` sang `src_vi/` với cùng tên file.
   Dịch trực tiếp trên đó.
5. Mở PR vào nhánh `main`. Cập nhật trạng thái chương trong ROADMAP.
6. Chờ review. Mục tiêu là văn bản đọc như một developer người Việt
   tự viết từ đầu.

## Build

Repo dùng cùng hệ thống build như upstream (`bgbspd`, `pandoc`,
`xelatex`), đóng gói sẵn trong `Dockerfile.vi`.

Build cục bộ qua Docker hoặc Podman:

```
./scripts/build_vi_docker.sh
```

Thư mục `docs/` sẽ chứa HTML, PDF, EPUB và landing page, giống hệt
bản deploy trên GitHub Pages.

Build thủ công, không dùng container: xem `scripts/build_vi.sh` để
biết các bước (cần `pandoc`, `texlive-xetex`, `python3`, `make`,
`zip`, `imagemagick`, `fonts-libertinus`).

## Đồng bộ với upstream

Commit upstream mà bản dịch này bám theo được ghi trong
[UPSTREAM.md](UPSTREAM.md).

Khi upstream có cập nhật, chúng tôi đồng bộ lại `src/` theo upstream,
rồi diff sẽ cho biết chương dịch nào cần chỉnh. Nếu bạn phát hiện
lệch, mở issue.

## Ghi công

- **Tài liệu gốc:** Brian "Beej Jorgensen" Hall, 2007 đến nay,
  https://beej.us/guide/bgc/
- **Bản dịch tiếng Việt:** Duc-Tam Nguyen (tamnd@liteio.dev) và
  [cộng đồng đóng góp](https://github.com/tamnd/bgc-vi/graphs/contributors)

## Giấy phép

[CC BY-NC-ND 3.0](LICENSE.md), giống upstream. Bạn được đọc, được
chia sẻ, và được dịch. Bạn không được bán hay tạo tác phẩm phái sinh
(trừ bản dịch, upstream cho phép rõ ràng). Code trong tài liệu là
public domain.

Toàn văn: [LICENSE.md](LICENSE.md) &middot; [trang của Creative
Commons](https://creativecommons.org/licenses/by-nc-nd/3.0/).

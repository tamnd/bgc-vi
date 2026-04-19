# Beej's Guide to C Programming (Vietnamese)

> [Tiếng Việt](README.md) &middot; English

Vietnamese translation of [Beej's Guide to C Programming][bgc] by
Brian "Beej Jorgensen" Hall. Free to read, free to share, just like
the original.

> Hey! C giving you grief? Pointers, `malloc()`, and byte-endianness
> making your head spin? You want to write real C without hauling
> around an 800-page textbook shaped like an anvil?
>
> Well, guess what. Beej already did that nasty business, and it now
> comes in Vietnamese too.

[bgc]: https://beej.us/guide/bgc/

## Is this for me?

If you read Vietnamese and want to learn C from scratch up to
"actually useful", yes. If you read English comfortably, just read
the [original guide][bgc], it's right there.

This repo is for Vietnamese developers who want to learn C seriously
without fighting English syntax in parallel with C syntax.

## What you'll learn

Forty-plus chapters, from "Hello, World!" to atomics, multithreading,
Unicode, `setjmp`/`longjmp`, VLAs, and a long tail of C specifiers
that most people never touch. Full list in
[ROADMAP.md](ROADMAP.md).

Highlights:

- **Pointers**, three chapters (because pointers deserve three).
- **Manual memory allocation.** `malloc()`, `free()`, and why.
- **Types**, five chapters, including qualifiers, specifiers,
  compound literals, generic selection.
- **The C preprocessor.** Macros are beautiful and dangerous.
- **Multithreading and atomics.** After you're comfortable with the
  rest.
- **Unicode, locale, time.** The corners everyone avoids but
  shouldn't.

## Status

The build pipeline and GitHub Pages went live before translation
started, so there's already an English site at
<https://tamnd.github.io/bgc-vi/>. Vietnamese content will replace it
one chapter at a time. Track progress in [ROADMAP.md](ROADMAP.md) or
the [merged PRs](https://github.com/tamnd/bgc-vi/pulls?q=is%3Apr+is%3Amerged).

## Repo layout

```
bgc-vi/
├── src/         # English originals (from upstream, do not edit)
├── src_vi/      # Vietnamese translations (the interesting stuff)
├── source/      # Example C programs (unchanged from upstream)
├── translations/# Other-language builds shipped by upstream
├── website/     # Upstream website assets
├── scripts/     # Build and release helpers
├── Dockerfile.vi # Container image with pandoc + texlive + fonts
├── ROADMAP.md   # Translation plan and progress
├── UPSTREAM.md  # Which upstream commit we track
├── LICENSE.md   # CC BY-NC-ND 3.0, same as upstream
└── README.md    # You are here
```

Each translated chapter in `src_vi/` matches a file in `src/`
one-to-one (same filename, same section anchors). Diffing the two is
the simplest way to spot drift.

## How to read

**Online:** <https://tamnd.github.io/bgc-vi/>, rebuilt on every merge
to `main`.

**Offline:** grab files from the
[Releases](https://github.com/tamnd/bgc-vi/releases) page (PDF,
EPUB, HTML zip, source code). Or read markdown directly in `src_vi/`,
GitHub renders it fine.

**Build it yourself:** see [Building](#building) below.

## Contributing

Pull requests welcome. A few ground rules so the text stays readable:

- **One chapter per PR.** Don't batch. Small PRs get merged, big PRs
  sit.
- **Translate meaning, not words.** If a literal translation reads
  like a robot wrote it, rewrite it. Beej's tone is casual; yours
  should be too.
- **No machine translation.** Seriously. We can tell.
- **Keep code blocks, function names, variable names, and C keywords
  in English.** `malloc()` stays `malloc()`. `struct foo` stays
  `struct foo`.
- **First use of a technical term:** give the English word first,
  then Vietnamese in parentheses if helpful. Subsequent uses can drop
  the Vietnamese.
- **No em dashes in Vietnamese prose.** Rewrite the sentence or use a
  comma.
- **Section anchors stay intact.** If the English says `{#pointers}`,
  the Vietnamese says `{#pointers}`.

### Workflow

1. Pick a chapter in [ROADMAP.md](ROADMAP.md) marked "not started".
2. Open an issue saying you're taking it, so no one duplicates work.
3. Branch: `translate/<chapter-slug>` (e.g. `translate/pointers`).
4. Copy `src/bgc_part_NNNN_<slug>.md` to `src_vi/` with the same
   filename. Translate in place.
5. Open a PR against `main`. Update the chapter status in ROADMAP.
6. Expect review. The goal is prose that reads like a Vietnamese
   developer wrote it from scratch.

## Building

The repo uses the same build system as upstream (`bgbspd`, `pandoc`,
`xelatex`), packaged in `Dockerfile.vi`.

Build locally via Docker or Podman:

```
./scripts/build_vi_docker.sh
```

The `docs/` directory will contain HTML, PDFs, EPUB, and a landing
page, identical to what lands on GitHub Pages.

Building without a container: see `scripts/build_vi.sh` for the
steps (you'll need `pandoc`, `texlive-xetex`, `python3`, `make`,
`zip`, `imagemagick`, `fonts-libertinus`).

## Sync with upstream

The upstream commit we track is recorded in
[UPSTREAM.md](UPSTREAM.md).

When upstream ships changes, we re-sync `src/` against upstream; the
diff then tells us which translated chapters need touch-ups. If you
spot drift, open an issue.

## Credits

- **Original guide:** Brian "Beej Jorgensen" Hall, 2007-present,
  https://beej.us/guide/bgc/
- **Vietnamese translation:** Duc-Tam Nguyen (tamnd@liteio.dev) and
  [contributors](https://github.com/tamnd/bgc-vi/graphs/contributors)

## License

[CC BY-NC-ND 3.0](LICENSE.md), same as upstream. You can read it,
share it, and translate it. You can't sell it or make derivative
works (other than translations, which upstream explicitly permits).
Source code in the guide is public domain.

Full text: [LICENSE.md](LICENSE.md) &middot; [Creative Commons
page](https://creativecommons.org/licenses/by-nc-nd/3.0/).

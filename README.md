# fliphtml5Downloader

Download public FlipHTML5 books as images + single PDF from the terminal. Pure Python, no Node required.

Works with both old FlipHTML5 books (`fliphtml5_pages` = plain list) and new books (`fliphtml5_pages` / `bookConfig` = encrypted `v01...` string, e.g. `https://online.fliphtml5.com/ceeok/pkdn/`).

New-format books are decrypted with FlipHTML5's own `deString` WASM (`static.fliphtml5.com/.../js/deString.js`) executed locally via the pip package `wasmtime`.

## Requirements

* Python 3.10+
* pip packages: `requests`, `img2pdf`, `Pillow`, `wasmtime`

## Setup

```bash
git clone <your-fork-url>
cd fliphtml5Downloader
pip install requests img2pdf Pillow wasmtime
```

No Node, no browser, no API key needed.

## Usage

Interactive mode:

```bash
python3 fliphtml5Downloader.py
```

You will be prompted:

```text
Enter the book link in the format *****/****:
```

Enter either the short code or the full URL. All of these work:

```text
ceeok/pkdn
https://online.fliphtml5.com/ceeok/pkdn/
https://online.fliphtml5.com/ceeok/pkdn/#p=1
https://online.fliphtml5.com/ceeok/pkdn/index.html
```

One-shot mode (no prompt):

```bash
python3 fliphtml5Downloader.py ceeok/pkdn
python3 fliphtml5Downloader.py https://online.fliphtml5.com/ceeok/pkdn/
```

Book ID rule: it is the pair after `fliphtml5.com/`. Example: `https://online.fliphtml5.com/hgzok/boba` → `hgzok/boba`.

## What it does

1. Fetches `https://online.fliphtml5.com/<book>/javascript/config.js`.
2. If `fliphtml5_pages` is encrypted, downloads and caches `.cache/deString.js` once, extracts the embedded WASM, and decodes it with `wasmtime`.
3. Downloads pages from `files/large/<hash>.webp` (with `.jpg`, `medium`, `thumb` fallbacks).
4. Saves images to a folder named after the book title, e.g.:
   `AI Engineering Building Applications with Foundation Models (Chip Huyen) (Z-Library)/1.webp ... 535.webp`
5. Converts WebP → JPEG in memory with `Pillow` and merges to `<title>.pdf` with `img2pdf`.

Example output for `ceeok/pkdn` (535 pages):

```text
Book: ceeok/pkdn
Encrypted fliphtml5_pages detected, decoding with wasmtime (no Node)...
Found 535 pages
Downloading Page 1 / 535 .....
...
Downloading Complete. Don't Close, hold-on. We are yet to make PDF.
The pdf named AI Engineering ....pdf has been saved in your working directory.
```

## Troubleshooting

* `TypeError: string indices must be integers, not 'str'` — you are on the old script. Pull the latest version, this is fixed by WASM decoding.
* `pip package 'wasmtime' is required` → `pip install wasmtime`.
* `failed ... (http:404)` on some pages — script already tries `large` → `jpg/webp` swap → `medium` → `thumb`. If all fail, that page is skipped and reported.
* `Nothing downloaded, PDF not created` — book is private, removed, or FlipHTML5 changed format again. Open an issue with the book ID.
* Large books (500+ pages) take several minutes and use ~500MB+ disk. Make sure you have space.

## Notes

* Original tutorial: https://youtu.be/RNzS0H2hspA
* Related issue example using `archive.alsharekh.org` mobile config is kept in `issue1.py` (separate site, separate logic).
* Only download books you have the right to download.

## Disclaimer

For educational purposes only. Respect copyright and FlipHTML5 terms of service.

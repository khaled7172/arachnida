# Arachnida - 42 Cybersecurity Piscine

## 1. Project Overview & Context

**Arachnida** is the introductory project to web scraping, crawling, and metadata analysis in the 42 Cybersecurity Piscine.
The project consists of two distinct programs:
1. **Spider (`./spider`)**: A web crawler/scraper that extracts images from a given URL recursively.
2. **Scorpion (`./scorpion`)**: A metadata extractor that parses and displays EXIF and file metadata from local image files.

---

## 2. Subject Breakdown & Requirements

### 2.1 Rules & Prohibitions
- **Language**: Programs can be written as **scripts** (e.g., Python with `#!/usr/bin/env python3`) or **binaries** (compiled language like C/C++/Go/Rust with a `Makefile`).
- **Forbidden Tools**: Using `wget`, `curl` CLI wrappers, or framework web scrapers like `scrapy` is strictly considered cheating and results in a grade of **0**.
- **Allowed**: Libraries/functions that handle standard HTTP requests (e.g. `requests`, `urllib`) and file handling. However, the traversal logic, recursion, URL parsing, deduplication, and extraction must be written manually.

---

### 2.2 Exercise 1: Spider (`./spider`)

#### Usage
```bash
./spider [-rlp] URL
```

#### Flags & Options
| Flag | Description | Default Value |
|---|---|---|
| `-r` | Recursively downloads images from links found on the page. | Off (single-page only if not set) |
| `-r -l [N]` | Sets the maximum recursion depth level. | `5` (when `-r` is provided without `-l`) |
| `-p [PATH]` | Specifies destination directory where images are saved. | `./data/` |
| `URL` | The target root URL to scrape. | Required |

#### Image Types to Download
- `.jpg` / `.jpeg`
- `.png`
- `.gif`
- `.bmp`

#### Core Engineering Considerations
- **URL Normalization**: Resolving relative links (`/images/pic.png`, `../logo.jpg`) to absolute URLs.
- **Domain Scope**: Restricting recursion to the same domain/origin so spider doesn't crawl the entire Internet.
- **Cycle & Duplicate Prevention**: Maintaining a visited URL set and an image checksum/seen set to prevent infinite loops and downloading the same image multiple times.
- **Error Handling**: Graceful handling of HTTP 404, 403, SSL certificate issues, timeouts, and malformed HTML.

---

### 2.3 Exercise 2: Scorpion (`./scorpion`)

#### Usage
```bash
./scorpion FILE1 [FILE2 ...]
```

#### Requirements
- Must accept one or more image files as command-line arguments.
- Must support at least the same extensions handled by spider (`.jpg`/`.jpeg`, `.png`, `.gif`, `.bmp`).
- Must extract and display:
  - **Basic File Attributes**: File name, file size, creation date, last modification date, MIME type, dimensions (width × height).
  - **EXIF Data**: Camera make/model, date/time original, exposure, ISO, GPS coordinates, software, lens specifications, copyright, etc. (where present).
- Output format is flexible, but must be clear, well-structured, and human-readable.

---

### 2.4 Bonus Features
- Option to **modify or delete** metadata from a given file (e.g. `-d` to strip EXIF data).
- A graphical user interface (GUI) or TUI for viewing and managing metadata.
> *Note: Bonus is only evaluated if mandatory requirements are 100% complete and flawless.*

---

## 3. Architecture Decision: Script vs. Compiled Binary

The subject permits either **scripts** or **binaries**. Here is the trade-off analysis:

| Criteria | Script (Python 3) | Compiled Binary (C / C++) |
|---|---|---|
| **Development Speed** | ⚡ Extremely fast; rich native & standard libraries | 🐢 Very slow; low-level memory & string management |
| **HTTP & Parsing** | Robust (`requests`, `BeautifulSoup` or `html.parser`, `urllib.parse`) | Needs `libcurl` and manual HTML tag tokenizer |
| **EXIF Parsing** | Handled natively via `Pillow` (`PIL.ExifTags`) or pure Python | Complex byte parsing or linking against `libexif` |
| **Portability / 42 Eval** | Works out of the box with standard Python 3. Shebang (`#!/usr/bin/env python3`) satisfies `./spider` requirement. | Requires a `Makefile` compiling cleanly with `-Wall -Wextra -Werror`. |
| **Defense / Code Review** | Clean, readable logic for recursion depth and visited sets. | Evaluator has to audit pointers, buffers, and memory leaks (`valgrind`). |

### Recommended Choice
**Python 3 executable scripts** (`./spider` and `./scorpion`) with:
- `chmod +x`
- Clear shebang: `#!/usr/bin/env python3`
- Minimal third-party dependencies (`requests`, `beautifulsoup4`, `Pillow`) documented in `requirements.txt`.

---

## 4. Submission Directory Structure

For 42 Vogsphere evaluations, keeping a clean root directory without compiled binaries or unneeded trash is crucial:

```text
.
├── .gitignore
├── README.md
├── aranchida.md
├── requirements.txt
├── spider            # Executable Python script (or Makefile + src if compiled)
└── scorpion          # Executable Python script (or Makefile + src if compiled)
```

*(Note: Downloaded images inside `data/` will be ignored via `.gitignore` to keep git repositories clean.)*

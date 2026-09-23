# `spider` — Recursive Web Crawler & Image Extractor

> **Pipeline position**: 1 of 2
> **What this file does in plain English**: This program acts like an automated explorer that visits a target website, inspects its code, and collects all image files matching specific file formats. If asked to explore recursively, it behaves like someone clicking internal links from page to page up to a set depth limit. It carefully saves every image to a designated folder on your computer while making sure it never downloads duplicates or wanders off to unrelated websites.
> **Files it depends on**: Python Standard Library (`os`, `sys`, `argparse`, `hashlib`, `urllib.parse`, `collections`), `requests`, `bs4` (BeautifulSoup)
> **Files that depend on it**: `data/` image repository, which directly feeds into `scorpion`

---

## 📑 Section 1: Line-by-Line Code Breakdown

---

## Lines 1–20: Shebang, Imports, and Configuration Constants

### The Code
```python
#!/usr/bin/env python3
"""
Spider: A recursive web crawler and image extractor for 42 Cybersecurity Piscine.
"""

import os
import sys
import argparse
import hashlib
from urllib.parse import urlparse, urljoin, unquote
from collections import deque
import requests
from bs4 import BeautifulSoup

VALID_EXTENSIONS = ('.jpg', '.jpeg', '.png', '.gif', '.bmp')
DEFAULT_SAVE_DIR = './data/'
DEFAULT_MAX_DEPTH = 5
REQUEST_TIMEOUT = 10
USER_AGENT = "SpiderBot/1.0 (+https://github.com/khaled7172/aranchida)"
```

### Plain English
This initial block tells the operating system how to run this file and loads all the external toolkits (called modules or libraries) required to fetch websites, parse text, and manage files on disk. Without these tools, our program would have to build network communication and HTML string manipulation from scratch.

Following the imports, we define global configuration constants. These are fixed settings used across the entire program, such as the list of image file types we are allowed to download (`.jpg`, `.png`, etc.), the default save folder (`./data/`), the default recursion depth (`5`), and how many seconds to wait before giving up on a slow web server (`10` seconds).

Setting these values at the top of the file ensures that any future rule changes or tuning can be done in one single place without digging through hundreds of lines of code.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 1 | `#!/usr/bin/env python3` | Shebang line telling Unix shells to execute this file using the `python3` binary in the environment path. |
| 2 | `"""` | Opens a multi-line documentation string describing the module. |
| 3 | `Spider: A recursive web crawler...` | Human-readable explanation of the program purpose. |
| 4 | `"""` | Closes the module documentation string. |
| 5 | *(empty line)* | Clean code spacing separating docstrings from import statements. |
| 6 | `import os` | Imports the operating system module to interact with folders, paths, and local files. |
| 7 | `import sys` | Imports the system module to access system-specific parameters and write error messages to stderr. |
| 8 | `import argparse` | Imports the command-line argument parsing library to handle terminal flags like `-r`, `-l`, and `-p`. |
| 9 | `import hashlib` | Imports cryptographic hashing functions (like SHA-256 and MD5) to detect duplicate images. |
| 10 | `from urllib.parse import urlparse, urljoin, unquote` | Imports URL helper functions to split URLs, resolve relative paths to absolute URLs, and decode URL-encoded text. |
| 11 | `from collections import deque` | Imports the double-ended queue data structure to implement fast Breadth-First Search (BFS) page traversal. |
| 12 | `import requests` | Imports the HTTP client library to perform network requests and download web page contents. |
| 13 | `from bs4 import BeautifulSoup` | Imports the HTML parsing engine to locate and extract image tags and anchor hyperlinks from web pages. |
| 14 | *(empty line)* | Clean code spacing separating imports from global constants. |
| 15 | `VALID_EXTENSIONS = ('.jpg', '.jpeg', '.png', '.gif', '.bmp')` | Defines a tuple containing all allowed image file extensions specified by the subject. |
| 16 | `DEFAULT_SAVE_DIR = './data/'` | Defines the fallback folder path where downloaded images are saved if `-p` is not passed. |
| 17 | `DEFAULT_MAX_DEPTH = 5` | Sets the default maximum recursion depth level specified by the subject. |
| 18 | `REQUEST_TIMEOUT = 10` | Sets the maximum network wait time in seconds to prevent the crawler from hanging on frozen servers. |
| 19 | `USER_AGENT = "SpiderBot/1.0 (+https://github.com/khaled7172/aranchida)"` | Sets an identifiable HTTP User-Agent header so remote servers do not block our requests as generic scripts. |
| 20 | *(empty line)* | Visual separation before the first function definition. |

### New Concepts Introduced

> **`Shebang (#!)`**: The first line in a Unix script that tells the operating system which interpreter program should read and execute the script. It allows running `./spider` directly without typing `python3 spider`.
>
> **`Double-Ended Queue (deque)`**: An optimized list structure designed for adding and popping items from both ends in $O(1)$ constant time. In web crawlers, it powers Breadth-First Search (BFS) queues efficiently.

---

## Lines 22–63: `parse_arguments()`

### The Code
```python
def parse_arguments():
    """Parse and validate command line arguments."""
    parser = argparse.ArgumentParser(
        prog="spider",
        description="Extract images from a website recursively."
    )
    parser.add_argument(
        "-r",
        action="store_true",
        dest="recursive",
        help="Recursively download images from linked pages."
    )
    parser.add_argument(
        "-l",
        type=int,
        default=DEFAULT_MAX_DEPTH,
        dest="depth",
        metavar="N",
        help=f"Maximum recursion depth level (default: {DEFAULT_MAX_DEPTH})."
    )
    parser.add_argument(
        "-p",
        type=str,
        default=DEFAULT_SAVE_DIR,
        dest="path",
        metavar="PATH",
        help=f"Path where downloaded files will be saved (default: {DEFAULT_SAVE_DIR})."
    )
    parser.add_argument(
        "url",
        type=str,
        metavar="URL",
        help="Target URL to scrape."
    )

    args = parser.parse_args()

    if args.depth < 0:
        parser.error("Recursion depth level must be a non-negative integer.")

    return args
```

### Plain English
When a user runs a program in the terminal, they pass configuration flags such as `./spider -r -l 3 -p ./downloads/ https://example.com`. This function acts as the receptionist: it reads whatever text the user typed after the program name, checks if the flags match the official rules, and converts them into structured variables our code can easily check.

If the user asks for help using `-h` or `--help`, this function automatically prints an elegant manual explaining each option. Furthermore, it validates inputs—for example, if someone passes a negative number for depth (like `-l -2`), it instantly stops execution and prints a clear error message.

Once validated, the function returns a clean bundle of arguments containing the target URL, the recursion boolean, the depth limit, and the destination storage path.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 22 | `def parse_arguments():` | Declares the function responsible for parsing command-line parameters. |
| 23 | `"""Parse and validate command line arguments."""` | Docstring explaining the function's purpose. |
| 24 | `parser = argparse.ArgumentParser(` | Instantiates an `ArgumentParser` object. |
| 25 | `prog="spider",` | Specifies the binary name displayed in the generated help banner. |
| 26 | `description="Extract images from a website recursively."` | Sets the description string displayed when showing usage help. |
| 27 | `)` | Closes the `ArgumentParser` instantiation call. |
| 28 | `parser.add_argument(` | Begins adding the `-r` optional flag to the parser. |
| 29 | `"-r",` | Specifies the short command-line flag name for recursion. |
| 30 | `action="store_true",` | Stores `True` if `-r` is present in the command line, and `False` otherwise. |
| 31 | `dest="recursive",` | Names the attribute on the parsed args object where this value will be saved. |
| 32 | `help="Recursively download images from linked pages."` | Description of the flag displayed in the help menu. |
| 33 | `)` | Closes the `-r` argument definition. |
| 34 | `parser.add_argument(` | Begins adding the `-l` optional argument for recursion depth. |
| 35 | `"-l",` | Specifies the short flag name for depth level. |
| 36 | `type=int,` | Enforces that any value passed after `-l` must be parsed as an integer. |
| 37 | `default=DEFAULT_MAX_DEPTH,` | Sets the default depth value to `5` if `-l` is omitted. |
| 38 | `dest="depth",` | Names the attribute storing the depth integer. |
| 39 | `metavar="N",` | Specifies the placeholder variable name shown in the help menu (`-l N`). |
| 40 | `help=f"Maximum recursion depth level (default: {DEFAULT_MAX_DEPTH})."` | Explanatory help string showing the default depth. |
| 41 | `)` | Closes the `-l` argument definition. |
| 42 | `parser.add_argument(` | Begins adding the `-p` optional argument for directory path. |
| 43 | `"-p",` | Specifies the short flag name for the save path. |
| 44 | `type=str,` | Treats the argument following `-p` as a string. |
| 45 | `default=DEFAULT_SAVE_DIR,` | Sets the default save directory to `./data/`. |
| 46 | `dest="path",` | Names the attribute storing the save folder path. |
| 47 | `metavar="PATH",` | Sets the placeholder variable name shown in help (`-p PATH`). |
| 48 | `help=f"Path where downloaded files will be saved (default: {DEFAULT_SAVE_DIR})."` | Explanatory help text indicating default output directory. |
| 49 | `)` | Closes the `-p` argument definition. |
| 50 | `parser.add_argument(` | Begins adding the mandatory positional argument for the target URL. |
| 51 | `"url",` | Defines the name of the positional parameter. |
| 52 | `type=str,` | Treats the URL parameter as a string. |
| 53 | `metavar="URL",` | Placeholder displayed in help format string. |
| 54 | `help="Target URL to scrape."` | Help description for the URL argument. |
| 55 | `)` | Closes the positional URL argument definition. |
| 56 | *(empty line)* | Clean code spacing before argument execution. |
| 57 | `args = parser.parse_args()` | Inspects `sys.argv`, parses all tokens, and populates the `args` namespace. |
| 58 | *(empty line)* | Clean code spacing before validation logic. |
| 59 | `if args.depth < 0:` | Checks if the user provided an illogical negative recursion depth. |
| 60 | `parser.error("Recursion depth level must be a non-negative integer.")` | Prints an error and terminates execution with exit code 2 if depth is negative. |
| 61 | *(empty line)* | Clean spacing before return statement. |
| 62 | `return args` | Returns the validated parsed arguments object to the caller. |
| 63 | *(empty line)* | Visual separation before the next function. |

---

## Lines 65–70: `is_valid_image_url(url)`

### The Code
```python
def is_valid_image_url(url):
    """Check if the URL path ends with a valid supported image extension."""
    parsed = urlparse(url)
    clean_path = unquote(parsed.path).lower()
    return any(clean_path.endswith(ext) for ext in VALID_EXTENSIONS)
```

### Plain English
A URL on the web often contains extra baggage, such as query parameters (`?width=500&format=jpg`) or anchor tags (`#top`). Furthermore, file extensions might be uppercase (`.PNG` or `.JPEG`) or URL-encoded (`%2Epng`).

This function strips away everything except the real file path, decodes any encoded characters, converts the text to lowercase, and checks if it ends with one of our permitted image formats (`.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`). If it does, the function returns `True`; otherwise, it returns `False`.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 65 | `def is_valid_image_url(url):` | Declares the helper function that checks if a URL points to an accepted image. |
| 66 | `"""Check if the URL path ends with a valid supported image extension."""` | Explanatory docstring for the validation function. |
| 67 | `parsed = urlparse(url)` | Deconstructs the full URL into parts (scheme, netloc, path, params, query, fragment). |
| 68 | `clean_path = unquote(parsed.path).lower()` | Decodes `%20` or hex codes in the URL path and normalizes all characters to lowercase. |
| 69 | `return any(clean_path.endswith(ext) for ext in VALID_EXTENSIONS)` | Evaluates whether `clean_path` terminates with any of the allowed extension strings. |
| 70 | *(empty line)* | Visual separation between functions. |

---

## Lines 72–76: `is_same_domain(base_netloc, target_url)`

### The Code
```python
def is_same_domain(base_netloc, target_url):
    """Verify if target_url belongs to the same domain as the starting URL."""
    target_netloc = urlparse(target_url).netloc
    return base_netloc.lower() == target_netloc.lower()
```

### Plain English
When crawling a website recursively, links on web pages frequently point outward to external websites (like Twitter, GitHub, or advertising networks). If a crawler blindly followed every link it found, it would soon attempt to download images from the entire global Internet!

This security check parses the target URL's domain name (the `netloc`, such as `example.com`) and compares it against the original starting website domain. If they match, our crawler stays on target. If they differ, the external link is ignored.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 72 | `def is_same_domain(base_netloc, target_url):` | Defines domain comparison function taking the root domain and a target URL. |
| 73 | `"""Verify if target_url belongs to the same domain as the starting URL."""` | Docstring explaining domain boundary enforcement. |
| 74 | `target_netloc = urlparse(target_url).netloc` | Extracts the hostname/network location from the candidate URL. |
| 75 | `return base_netloc.lower() == target_netloc.lower()` | Performs case-insensitive equality comparison between base domain and target domain. |
| 76 | *(empty line)* | Visual separation between functions. |

---

## Lines 78–98: `sanitize_filename(url, save_dir)`

### The Code
```python
def sanitize_filename(url, save_dir):
    """Generate a clean, collision-free local filename for an image URL."""
    parsed = urlparse(url)
    base_name = os.path.basename(unquote(parsed.path))
    if not base_name or not any(base_name.lower().endswith(ext) for ext in VALID_EXTENSIONS):
        ext = ".jpg"
        for candidate_ext in VALID_EXTENSIONS:
            if candidate_ext in url.lower():
                ext = candidate_ext
                break
        url_hash = hashlib.md5(url.encode('utf-8')).hexdigest()[:8]
        base_name = f"image_{url_hash}{ext}"

    name, ext = os.path.splitext(base_name)
    file_path = os.path.join(save_dir, base_name)
    counter = 1
    while os.path.exists(file_path):
        file_path = os.path.join(save_dir, f"{name}_{counter}{ext}")
        counter += 1
    return file_path
```

### Plain English
When saving files from the web, two common problems occur:
1. The URL might not have a clean filename (for instance, dynamic image generator endpoints like `https://site.com/avatar?id=42`).
2. Two different pages might have an image with the exact same name (such as `logo.png`). If we save the second one directly, it would overwrite and erase the first one.

This function extracts a clean filename from the URL path. If no valid filename exists, it generates a unique name using an MD5 hash of the URL. Then, it checks if a file with that name already exists in the save folder. If it does, it automatically appends a number suffix (e.g. `logo_1.png`, `logo_2.png`) so no existing files are ever overwritten.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 78 | `def sanitize_filename(url, save_dir):` | Declares filename resolution and collision-avoidance function. |
| 79 | `"""Generate a clean, collision-free local filename for an image URL."""` | Docstring explaining filename generation and conflict handling. |
| 80 | `parsed = urlparse(url)` | Deconstructs the image URL into URL components. |
| 81 | `base_name = os.path.basename(unquote(parsed.path))` | Extracts the final component of the path and unescapes encoded characters. |
| 82 | `if not base_name or not any(base_name.lower().endswith(ext) for ext in VALID_EXTENSIONS):` | Checks if extracted name is empty or lacks a valid image extension. |
| 83 | `ext = ".jpg"` | Sets default fallback file extension. |
| 84 | `for candidate_ext in VALID_EXTENSIONS:` | Iterates through valid extensions to inspect if one appears in the URL query. |
| 85 | `if candidate_ext in url.lower():` | Checks if a known extension exists inside the URL string. |
| 86 | `ext = candidate_ext` | Adopts the discovered candidate extension. |
| 87 | `break` | Halts search after first extension match. |
| 88 | `url_hash = hashlib.md5(url.encode('utf-8')).hexdigest()[:8]` | Computes an 8-character MD5 hash of the URL to ensure a deterministic unique name. |
| 89 | `base_name = f"image_{url_hash}{ext}"` | Constructs fallback filename using the short hash and discovered extension. |
| 90 | *(empty line)* | Clean code spacing before collision check loop. |
| 91 | `name, ext = os.path.splitext(base_name)` | Splits the filename into root name and file extension. |
| 92 | `file_path = os.path.join(save_dir, base_name)` | Combines destination directory path and initial file name. |
| 93 | `counter = 1` | Initializes numeric counter for disambiguating identical filenames. |
| 94 | `while os.path.exists(file_path):` | Loops as long as a file already exists at the computed target path. |
| 95 | `file_path = os.path.join(save_dir, f"{name}_{counter}{ext}")` | Generates a new path with incremental numeric suffix (e.g. `pic_1.jpg`). |
| 96 | `counter += 1` | Increments collision counter for the next check. |
| 97 | `return file_path` | Returns the guaranteed unused, safe file path on disk. |
| 98 | *(empty line)* | Visual separation between functions. |

---

## Lines 100–129: `download_image(img_url, save_dir, session, downloaded_hashes)`

### The Code
```python
def download_image(img_url, save_dir, session, downloaded_hashes):
    """Download an image if valid and not previously downloaded."""
    try:
        response = session.get(img_url, timeout=REQUEST_TIMEOUT, stream=True)
        if response.status_code != 200:
            return False

        content_type = response.headers.get('Content-Type', '').lower()
        if 'text/html' in content_type:
            return False

        img_bytes = response.content
        if len(img_bytes) == 0:
            return False

        img_hash = hashlib.sha256(img_bytes).hexdigest()
        if img_hash in downloaded_hashes:
            return False

        file_path = sanitize_filename(img_url, save_dir)
        with open(file_path, 'wb') as f:
            f.write(img_bytes)

        downloaded_hashes.add(img_hash)
        print(f"[+] Downloaded: {img_url} -> {file_path}")
        return True
    except Exception as e:
        print(f"[-] Failed to download image {img_url}: {e}", file=sys.stderr)
        return False
```

### Plain English
This function handles the actual download of binary image bytes from the remote web server. It uses an HTTP `GET` request through an active session.

Before saving the file, it verifies three critical conditions:
1. The server returned HTTP 200 OK (meaning the image actually exists).
2. The server did not return an HTML error page disguised as an image.
3. The content is not an exact byte-for-byte duplicate of an image we already downloaded earlier (verified by calculating a SHA-256 cryptographic digital fingerprint).

If all checks pass, it writes the raw binary bytes directly into the target file on disk, records the fingerprint in our memory set, and prints a success notification. Any connection drops or socket errors are caught safely without crashing the crawler.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 100 | `def download_image(img_url, save_dir, session, downloaded_hashes):` | Declares function responsible for downloading and saving single images. |
| 101 | `"""Download an image if valid and not previously downloaded."""` | Docstring explaining download verification and deduplication. |
| 102 | `try:` | Begins protected block to catch network exceptions and disk errors. |
| 103 | `response = session.get(img_url, timeout=REQUEST_TIMEOUT, stream=True)` | Executes HTTP GET request with defined timeout and streaming enabled. |
| 104 | `if response.status_code != 200:` | Checks if server responded with a non-success HTTP status code. |
| 105 | `return False` | Aborts download and returns `False` if status is not 200. |
| 106 | *(empty line)* | Clean code spacing before header verification. |
| 107 | `content_type = response.headers.get('Content-Type', '').lower()` | Retrieves the HTTP `Content-Type` header in lowercase. |
| 108 | `if 'text/html' in content_type:` | Checks if server returned an HTML error document instead of binary image data. |
| 109 | `return False` | Rejects HTML response and returns `False`. |
| 110 | *(empty line)* | Clean code spacing before payload inspection. |
| 111 | `img_bytes = response.content` | Reads the full raw binary payload bytes into memory. |
| 112 | `if len(img_bytes) == 0:` | Checks if the returned binary payload is empty (zero bytes). |
| 113 | `return False` | Aborts saving empty files. |
| 114 | *(empty line)* | Clean code spacing before content hash check. |
| 115 | `img_hash = hashlib.sha256(img_bytes).hexdigest()` | Calculates SHA-256 cryptographic hash of image bytes for deduplication. |
| 116 | `if img_hash in downloaded_hashes:` | Checks if an identical image was already downloaded during this session. |
| 117 | `return False` | Discards duplicate image content. |
| 118 | *(empty line)* | Clean code spacing before writing file to disk. |
| 119 | `file_path = sanitize_filename(img_url, save_dir)` | Resolves a collision-free local destination file path. |
| 120 | `with open(file_path, 'wb') as f:` | Opens local file in write-binary mode (`wb`) using safe context manager. |
| 121 | `f.write(img_bytes)` | Flushes image binary bytes to physical storage. |
| 122 | *(empty line)* | Clean code spacing before tracking and reporting. |
| 123 | `downloaded_hashes.add(img_hash)` | Adds the SHA-256 hash to the set of saved images to avoid future duplicates. |
| 124 | `print(f"[+] Downloaded: {img_url} -> {file_path}")` | Prints user-facing success confirmation with source URL and local path. |
| 125 | `return True` | Returns `True` indicating a successful new download. |
| 126 | `except Exception as e:` | Catches any network, timeout, SSL, or disk IO exceptions. |
| 127 | `print(f"[-] Failed to download image {img_url}: {e}", file=sys.stderr)` | Prints readable error diagnostics to standard error stream (`sys.stderr`). |
| 128 | `return False` | Returns `False` to notify caller of failed download attempt. |
| 129 | *(empty line)* | Visual separation between functions. |

### New Concepts Introduced

> **`Cryptographic Hash Deduplication (SHA-256)`**: Instead of relying solely on filenames (which can be repeated or arbitrary), calculating a SHA-256 hash produces a unique 64-character fingerprint of the file's raw bytes. If two images with different URLs contain identical pixels, their SHA-256 hashes will be 100% identical, preventing redundant disk writes.

---

## Lines 131–171: `extract_page_assets(current_url, html_text, base_netloc)`

### The Code
```python
def extract_page_assets(current_url, html_text, base_netloc):
    """Parse HTML and extract valid image URLs and child page links."""
    soup = BeautifulSoup(html_text, 'html.parser')
    image_urls = set()
    page_urls = set()

    # Find images in <img> tags
    for img in soup.find_all('img'):
        src = img.get('src') or img.get('data-src')
        if src:
            full_img_url = urljoin(current_url, src.strip())
            if is_valid_image_url(full_img_url):
                image_urls.add(full_img_url)

    # Find images in <source> tags (srcset)
    for source in soup.find_all('source'):
        srcset = source.get('srcset')
        if srcset:
            first_candidate = srcset.split(',')[0].strip().split(' ')[0]
            full_img_url = urljoin(current_url, first_candidate)
            if is_valid_image_url(full_img_url):
                image_urls.add(full_img_url)

    # Find links in <a> tags
    for a_tag in soup.find_all('a', href=True):
        href = a_tag['href'].strip()
        if not href or href.startswith('#') or href.startswith('javascript:'):
            continue
        full_url = urljoin(current_url, href)
        parsed = urlparse(full_url)
        if parsed.scheme not in ('http', 'https'):
            continue
        # If link points directly to an image
        if is_valid_image_url(full_url):
            image_urls.add(full_url)
        elif is_same_domain(base_netloc, full_url):
            clean_url = full_url.split('#')[0]
            page_urls.add(clean_url)

    return image_urls, page_urls
```

### Plain English
When we download a web page, what we receive is a long text string of HTML code. This function uses BeautifulSoup to parse that HTML into an organized document tree and extracts two sets of links:
1. **Images to download**: Found in `<img>` tags (`src` or `data-src` for lazy-loaded images), modern `<picture>` tags with `<source srcset="...">`, and direct links inside `<a>` tags.
2. **Pages to crawl next**: Standard hyperlinks inside `<a href="...">` tags that belong to the same website domain.

Crucially, web pages often use relative paths like `/assets/pic.png` or `../about.html`. This function uses `urljoin` to automatically resolve every relative path into a complete, absolute web address (such as `https://example.com/assets/pic.png`).

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 131 | `def extract_page_assets(current_url, html_text, base_netloc):` | Declares HTML asset extractor taking current page URL, HTML body, and base domain. |
| 132 | `"""Parse HTML and extract valid image URLs and child page links."""` | Docstring explaining extraction of images and recursion hyperlinks. |
| 133 | `soup = BeautifulSoup(html_text, 'html.parser')` | Parses raw HTML string into navigable BeautifulSoup DOM tree. |
| 134 | `image_urls = set()` | Initializes empty set to store unique discovered image URLs on this page. |
| 135 | `page_urls = set()` | Initializes empty set to store unique discovered internal page links. |
| 136 | *(empty line)* | Clean code spacing before image tag search. |
| 137 | `# Find images in <img> tags` | Comment denoting traditional `<img>` tag parsing. |
| 138 | `for img in soup.find_all('img'):` | Iterates over every `<img>` HTML element found in the document. |
| 139 | `src = img.get('src') or img.get('data-src')` | Extracts standard `src` attribute or lazy-loaded `data-src` attribute. |
| 140 | `if src:` | Checks if an image source attribute was present. |
| 141 | `full_img_url = urljoin(current_url, src.strip())` | Converts relative image path into a fully qualified absolute URL. |
| 142 | `if is_valid_image_url(full_img_url):` | Verifies image extension matches one of the required extensions. |
| 143 | `image_urls.add(full_img_url)` | Adds the validated image URL into the discovered image set. |
| 144 | *(empty line)* | Clean code spacing before modern picture element search. |
| 145 | `# Find images in <source> tags (srcset)` | Comment denoting responsive HTML5 `<source>` tag parsing. |
| 146 | `for source in soup.find_all('source'):` | Iterates over every `<source>` HTML element (used inside `<picture>`). |
| 147 | `srcset = source.get('srcset')` | Retrieves the `srcset` attribute containing one or more responsive image URLs. |
| 148 | `if srcset:` | Checks if `srcset` is present and non-empty. |
| 149 | `first_candidate = srcset.split(',')[0].strip().split(' ')[0]` | Extracts the primary URL token before comma and descriptor spaces. |
| 150 | `full_img_url = urljoin(current_url, first_candidate)` | Resolves candidate image URL to an absolute address. |
| 151 | `if is_valid_image_url(full_img_url):` | Checks if candidate URL ends with an allowed image extension. |
| 152 | `image_urls.add(full_img_url)` | Adds valid responsive image URL to the image set. |
| 153 | *(empty line)* | Clean code spacing before hyperlink search. |
| 154 | `# Find links in <a> tags` | Comment denoting anchor tag extraction for crawling. |
| 155 | `for a_tag in soup.find_all('a', href=True):` | Iterates over all `<a>` anchor tags having an `href` attribute. |
| 156 | `href = a_tag['href'].strip()` | Retrieves and trims whitespace from the link target string. |
| 157 | `if not href or href.startswith('#') or href.startswith('javascript:'):` | Skips empty links, in-page bookmark anchors, and javascript handlers. |
| 158 | `continue` | Bypasses invalid link and proceeds to next iteration. |
| 159 | `full_url = urljoin(current_url, href)` | Resolves relative hyperlink into an absolute web address. |
| 160 | `parsed = urlparse(full_url)` | Breaks the resolved URL into components to inspect its scheme. |
| 161 | `if parsed.scheme not in ('http', 'https'):` | Rejects non-HTTP schemes (such as `mailto:`, `ftp:`, or `tel:`). |
| 162 | `continue` | Bypasses non-web schemes. |
| 163 | `# If link points directly to an image` | Comment indicating direct image file links. |
| 164 | `if is_valid_image_url(full_url):` | Checks if link directly targets an image file (e.g. `<a href="photo.jpg">`). |
| 165 | `image_urls.add(full_url)` | Adds direct image link to image set for download. |
| 166 | `elif is_same_domain(base_netloc, full_url):` | Checks if destination link belongs to the internal website domain. |
| 167 | `clean_url = full_url.split('#')[0]` | Strips fragment identifiers to avoid crawling identical pages multiple times. |
| 168 | `page_urls.add(clean_url)` | Adds internal page URL to crawl candidates. |
| 169 | *(empty line)* | Clean code spacing before return. |
| 170 | `return image_urls, page_urls` | Returns tuple of discovered image URLs set and internal page URLs set. |
| 171 | *(empty line)* | Visual separation between functions. |

---

## Lines 173–237: `crawl(start_url, recursive, max_depth, save_dir)`

### The Code
```python
def crawl(start_url, recursive, max_depth, save_dir):
    """Run breadth-first crawl to extract and download images."""
    os.makedirs(save_dir, exist_ok=True)
    parsed_start = urlparse(start_url)
    if not parsed_start.scheme or not parsed_start.netloc:
        print(f"Error: Invalid URL '{start_url}'. Must include http:// or https://", file=sys.stderr)
        sys.exit(1)

    base_netloc = parsed_start.netloc
    effective_max_depth = max_depth if recursive else 0

    session = requests.Session()
    session.headers.update({"User-Agent": USER_AGENT})

    visited_pages = set()
    discovered_images = set()
    downloaded_hashes = set()
    queue = deque([(start_url, 0)])

    print(f"[*] Starting spider crawl on {start_url}")
    print(f"[*] Recursive: {recursive} | Max Depth: {effective_max_depth} | Save Directory: {save_dir}")

    total_downloaded = 0

    while queue:
        current_url, depth = queue.popleft()

        if current_url in visited_pages:
            continue
        visited_pages.add(current_url)

        print(f"[*] Crawling (depth {depth}/{effective_max_depth}): {current_url}")

        try:
            response = session.get(current_url, timeout=REQUEST_TIMEOUT)
            if response.status_code != 200:
                print(f"[-] HTTP {response.status_code} for {current_url}", file=sys.stderr)
                continue

            content_type = response.headers.get('Content-Type', '').lower()
            if 'text/html' not in content_type:
                # Target URL itself might be a direct image
                if is_valid_image_url(current_url):
                    if download_image(current_url, save_dir, session, downloaded_hashes):
                        total_downloaded += 1
                continue

            img_urls, next_page_urls = extract_page_assets(current_url, response.text, base_netloc)

            for img_url in img_urls:
                if img_url not in discovered_images:
                    discovered_images.add(img_url)
                    if download_image(img_url, save_dir, session, downloaded_hashes):
                        total_downloaded += 1

            if recursive and depth < effective_max_depth:
                for next_url in next_page_urls:
                    if next_url not in visited_pages:
                        queue.append((next_url, depth + 1))

        except Exception as e:
            print(f"[-] Error fetching {current_url}: {e}", file=sys.stderr)

    print(f"[*] Crawl finished. Visited {len(visited_pages)} page(s). Downloaded {total_downloaded} unique image(s).")
```

### Plain English
This is the master brain of the spider. It implements a Breadth-First Search (BFS) crawling loop. 

First, it creates the output directory (e.g. `./data/`) if it does not already exist, verifies that the user provided a valid web URL starting with `http://` or `https://`, and initializes an HTTP session with connection pooling.

Next, it manages a task queue containing pairs of `(url, current_depth)`. As long as there are URLs in the queue:
1. It pulls the next page from the queue.
2. If we already visited this page, it skips it to prevent infinite loops.
3. It fetches the page, extracts all images, and downloads them.
4. If recursion is enabled (`-r`) and our current depth is less than the maximum depth (`-l`), it appends all newly discovered internal links to the queue with depth incremented by 1 (`depth + 1`).

When the queue becomes empty or the depth limit is reached, it displays a concise summary of how many pages were visited and how many unique images were saved.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 173 | `def crawl(start_url, recursive, max_depth, save_dir):` | Declares main crawling orchestrator function. |
| 174 | `"""Run breadth-first crawl to extract and download images."""` | Docstring explaining BFS traversal and image gathering. |
| 175 | `os.makedirs(save_dir, exist_ok=True)` | Recursively creates save folder if missing without raising error if it exists. |
| 176 | `parsed_start = urlparse(start_url)` | Parses starting URL to validate protocol and network hostname. |
| 177 | `if not parsed_start.scheme or not parsed_start.netloc:` | Checks if URL lacks scheme (`http`/`https`) or valid domain name. |
| 178 | `print(f"Error: Invalid URL '{start_url}'...", file=sys.stderr)` | Prints explicit error message explaining URL format requirement. |
| 179 | `sys.exit(1)` | Terminates execution with status code 1 on malformed starting URL. |
| 180 | *(empty line)* | Clean code spacing before initialization. |
| 181 | `base_netloc = parsed_start.netloc` | Records target domain name to keep crawling restricted to internal links. |
| 182 | `effective_max_depth = max_depth if recursive else 0` | Enforces depth 0 if `-r` was not given, or `max_depth` if `-r` was given. |
| 183 | *(empty line)* | Clean spacing before HTTP session setup. |
| 184 | `session = requests.Session()` | Creates reusable HTTP session for connection pooling and cookie preservation. |
| 185 | `session.headers.update({"User-Agent": USER_AGENT})` | Configures default HTTP request headers with custom User-Agent identity. |
| 186 | *(empty line)* | Clean spacing before state data structures. |
| 187 | `visited_pages = set()` | Set keeping track of visited page URLs to guarantee cycle-free traversal. |
| 188 | `discovered_images = set()` | Set tracking observed image URLs to prevent redundant network fetches. |
| 189 | `downloaded_hashes = set()` | Set storing SHA-256 binary hashes to prevent identical file duplicates. |
| 190 | `queue = deque([(start_url, 0)])` | Initializes BFS queue with starting URL at depth 0. |
| 191 | *(empty line)* | Clean spacing before banner logs. |
| 192 | `print(f"[*] Starting spider crawl on {start_url}")` | Prints initial crawl information to user. |
| 193 | `print(f"[*] Recursive: {recursive} | Max Depth: ...")` | Prints active crawl parameters and destination directory. |
| 194 | *(empty line)* | Clean spacing before counter and loop. |
| 195 | `total_downloaded = 0` | Counter tracking total number of new unique images saved. |
| 196 | *(empty line)* | Clean spacing before while loop. |
| 197 | `while queue:` | Loops continuously until BFS queue is empty. |
| 198 | `current_url, depth = queue.popleft()` | Pops the oldest pending page URL and its depth level in FIFO order. |
| 199 | *(empty line)* | Clean spacing before visited check. |
| 200 | `if current_url in visited_pages:` | Checks if this URL has already been processed. |
| 201 | `continue` | Skips already processed URLs to avoid duplicate crawling. |
| 202 | `visited_pages.add(current_url)` | Marks current URL as visited. |
| 203 | *(empty line)* | Clean spacing before status print. |
| 204 | `print(f"[*] Crawling (depth {depth}/{effective_max_depth}): {current_url}")` | Prints progress line showing current page URL and depth level. |
| 205 | *(empty line)* | Clean spacing before network fetch block. |
| 206 | `try:` | Starts try-except block to gracefully catch page fetching errors. |
| 207 | `response = session.get(current_url, timeout=REQUEST_TIMEOUT)` | Fetches the page content via HTTP GET. |
| 208 | `if response.status_code != 200:` | Checks for HTTP errors (e.g. 404 Not Found, 403 Forbidden). |
| 209 | `print(f"[-] HTTP {response.status_code} for {current_url}", file=sys.stderr)` | Prints non-200 status code warning to stderr. |
| 210 | `continue` | Aborts processing for failed HTTP responses. |
| 211 | *(empty line)* | Clean spacing before content-type inspection. |
| 212 | `content_type = response.headers.get('Content-Type', '').lower()` | Inspects MIME type returned by server. |
| 213 | `if 'text/html' not in content_type:` | Checks if requested URL is a non-HTML resource. |
| 214 | `# Target URL itself might be a direct image` | Comment indicating direct image target handling. |
| 215 | `if is_valid_image_url(current_url):` | Checks if starting URL was a direct link to an image file. |
| 216 | `if download_image(current_url, save_dir, session, downloaded_hashes):` | Downloads direct image file. |
| 217 | `total_downloaded += 1` | Increments download counter on success. |
| 218 | `continue` | Bypasses HTML parsing for binary image resources. |
| 219 | *(empty line)* | Clean spacing before parsing step. |
| 220 | `img_urls, next_page_urls = extract_page_assets(...)` | Calls parser function to retrieve image URLs and child hyperlinks. |
| 221 | *(empty line)* | Clean spacing before downloading discovered images. |
| 222 | `for img_url in img_urls:` | Iterates over all discovered image links on this page. |
| 223 | `if img_url not in discovered_images:` | Checks if this image URL was already processed during this session. |
| 224 | `discovered_images.add(img_url)` | Records image URL in seen set. |
| 225 | `if download_image(img_url, save_dir, session, downloaded_hashes):` | Downloads the image file and writes to disk if unique. |
| 226 | `total_downloaded += 1` | Increments download counter on successful disk write. |
| 227 | *(empty line)* | Clean spacing before queue expansion. |
| 228 | `if recursive and depth < effective_max_depth:` | Checks if recursive crawling is enabled and depth ceiling is not yet hit. |
| 229 | `for next_url in next_page_urls:` | Iterates through internal page hyperlinks. |
| 230 | `if next_url not in visited_pages:` | Confirms candidate page has not been previously visited. |
| 231 | `queue.append((next_url, depth + 1))` | Enqueues candidate page with incremented depth (`depth + 1`). |
| 232 | *(empty line)* | Clean spacing before exception handler. |
| 233 | `except Exception as e:` | Catches any unexpected network or parsing exception. |
| 234 | `print(f"[-] Error fetching {current_url}: {e}", file=sys.stderr)` | Prints error message to stderr without terminating crawler execution. |
| 235 | *(empty line)* | Clean spacing before summary print. |
| 236 | `print(f"[*] Crawl finished. Visited {len(visited_pages)} ...")` | Prints final execution summary with total visited pages and saved images. |
| 237 | *(empty line)* | Visual separation before main entry point. |

### New Concepts Introduced

> **`Breadth-First Search (BFS) Traversal`**: A graph traversal algorithm that explores all neighbor nodes at the present depth prior to moving on to the nodes at the next depth level. In web crawlers, BFS ensures that pages closest to the root are visited first, which respects depth limits predictably.

---

## Lines 239–250: `main()` and Script Entry Guard

### The Code
```python
def main():
    args = parse_arguments()
    crawl(
        start_url=args.url,
        recursive=args.recursive,
        max_depth=args.depth,
        save_dir=args.path
    )


if __name__ == "__main__":
    main()
```

### Plain English
This is the program's ignition switch. In Python, when you execute a file directly from the terminal (e.g. `./spider`), Python automatically sets a special hidden variable called `__name__` to `"__main__"`.

The guard `if __name__ == "__main__":` ensures that our `main()` function only runs when the script is directly executed from the command line, and not if another Python script imports functions from it. The `main()` function simply gathers the parsed arguments and kicks off the crawling engine.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 239 | `def main():` | Declares the program's top-level coordinator function. |
| 240 | `args = parse_arguments()` | Parses and validates command-line arguments. |
| 241 | `crawl(` | Calls the crawling orchestrator function. |
| 242 | `start_url=args.url,` | Passes the target starting URL. |
| 243 | `recursive=args.recursive,` | Passes the boolean recursion flag. |
| 244 | `max_depth=args.depth,` | Passes the maximum recursion depth limit. |
| 245 | `save_dir=args.path` | Passes the directory path where images must be saved. |
| 246 | `)` | Closes the `crawl()` invocation. |
| 247 | *(empty line)* | Clean code spacing between function and entry guard. |
| 248 | *(empty line)* | Clean code spacing. |
| 249 | `if __name__ == "__main__":` | Checks if script is being executed directly in the terminal. |
| 250 | `main()` | Calls `main()` to begin program execution. |

---

## 📊 Section 3: Complete ASCII Data Flow Diagram

```text
[User Terminal Command]
    ./spider [-r] [-l N] [-p PATH] URL
                    │
                    ▼
          `parse_arguments()`
  (Validates flags, sets defaults, checks depth >= 0)
                    │
                    ▼
          `crawl(start_url, ...)`
   ┌────────────────┴────────────────┐
   │ Initializes Session & BFS Queue │
   └────────────────┬────────────────┘
                    │
         ┌──────────▼──────────┐
         │ Is Queue Empty?     │◄───────────────────┐
         └──────────┬──────────┘                    │
              No    │        Yes                    │
                    │         └───────────► [Print Final Summary & Exit]
                    ▼                               │
        Pop `(current_url, depth)`                  │
                    │                               │
       Already in `visited_pages`?                  │
              ├── Yes ──► Skip                      │
              └── No  ──► Mark Visited              │
                    │                               │
                    ▼                               │
           HTTP GET `current_url`                   │
                    │                               │
                    ▼                               │
        `extract_page_assets()`                     │
         ├── Discovers Images (`<img>`, `<source>`) │
         └── Discovers Internal Links (`<a>`)       │
                    │                               │
                    ▼                               │
        `download_image(img_url)`                   │
         ├── Verify Content-Type != HTML            │
         ├── Check SHA-256 (Deduplication)          │
         ├── `sanitize_filename()` (No Overwrite)   │
         └── Write Binary Bytes to `./data/`        │
                    │                               │
                    ▼                               │
        If `recursive` & `depth < max_depth`:       │
         Enqueue new child links with `depth + 1` ──┘
```

---

## 🧠 Section 4: Key Takeaways

1. **Academic Integrity & Prohibited Tools**: By using `requests` and `BeautifulSoup` solely for HTTP communication and HTML parsing, our crawling traversal, depth tracking, and deduplication logic are 100% handcrafted—completely avoiding prohibited tools (`wget`, `scrapy`).
2. **Infinite Loop Prevention**: Web graphs often contain cycles (Page A links to Page B, which links back to Page A). Keeping a `visited_pages` set and an explicit `effective_max_depth` ceiling guarantees that the crawler always terminates cleanly.
3. **Collision & Duplicate Defense**: Combining deterministic SHA-256 byte hashing with numeric filename suffixing (`_1`, `_2`) guarantees that images are never re-downloaded unnecessarily and existing files are never overwritten.

---

## 📎 Section 5: Concepts Introduced in This File

| Concept | First seen at line | Quick definition |
|---------|-------------------|-----------------|
| Shebang (`#!/usr/bin/env python3`) | 1 | Directive telling Unix shells which interpreter executes the script. |
| Double-Ended Queue (`deque`) | 11 | $O(1)$ queue optimized for pushing/popping from both ends. |
| Command-line Argument Parsing (`argparse`) | 24 | Standard library parser converting CLI tokens into structured attributes. |
| URL Decomposition (`urlparse`) | 67 | Deconstructing web addresses into scheme, domain, path, and parameters. |
| URL Normalization (`urljoin`) | 141 | Merging a relative path with a base URL to create an absolute web address. |
| Cryptographic Hash Fingerprinting (`hashlib.sha256`) | 115 | Computing unique mathematical digests of binary data for deduplication. |
| Breadth-First Search (BFS) | 197 | Graph traversal exploring nodes level-by-level using a FIFO queue. |
| Module Execution Guard (`__name__ == "__main__"`) | 249 | Idiom ensuring code only runs when executed as a main script. |

---

## ➡️ Section 6: What's Next

Next up: **[`scorpion`](file:///home/kali/k/scorpion)** — We will create our second program to analyze, extract, and display EXIF tags and detailed binary file metadata from the images collected by `spider`!

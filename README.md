# Arachnida

An introductory project to web scraping, recursive crawling, and image metadata analysis for the **42 Cybersecurity Piscine**.

---

## 📖 Overview

Arachnida consists of two CLI tools designed to interact with web assets and inspect binary file metadata:

1. **`spider`**: A recursive web crawler and image extractor that gathers media files from a target URL up to a configurable depth.
2. **`scorpion`**: An image metadata analyzer that parses and displays basic file properties along with EXIF tags (such as camera metadata, timestamps, and GPS details).

---

## 🛠️ Requirements & Installation

- **Python 3.8+**
- Dependencies:
  - `requests` (HTTP requests)
  - `beautifulsoup4` (HTML parsing)
  - `Pillow` (Image & EXIF extraction)

To install dependencies:
```bash
pip install -r requirements.txt
```

Ensure the scripts have execution permissions:
```bash
chmod +x spider scorpion
```

---

## 🚀 Usage

### 1. Spider (`./spider`)

Extracts images from a website recursively.

```bash
./spider [-rlp] URL
```

#### Options:
- `-r`: Recursively downloads images linked within the target page.
- `-l [N]`: Specifies maximum recursion depth level (defaults to `5` when `-r` is active).
- `-p [PATH]`: Specifies the directory where images are saved (defaults to `./data/`).

#### Supported Extensions:
- `.jpg` / `.jpeg`
- `.png`
- `.gif`
- `.bmp`

#### Examples:
```bash
# Single page scrape into default ./data/
./spider https://example.com

# Recursive scrape with depth 3 into custom directory ./downloads/
./spider -r -l 3 -p ./downloads/ https://example.com
```

---

### 2. Scorpion (`./scorpion`)

Parses and displays EXIF and file metadata from local image files.

```bash
./scorpion FILE1 [FILE2 ...]
```

#### Displays:
- **Basic Attributes**: File name, file size, MIME type, creation/modification dates, dimensions.
- **EXIF Attributes**: Camera model, exposure, aperture, ISO, software, GPS coordinates, etc.

#### Example:
```bash
./scorpion data/sample.jpg data/photo.png
```

---

## ⚖️ Project Structure

```text
.
├── .gitignore
├── README.md
├── aranchida.md
├── requirements.txt
├── spider
└── scorpion
```

---

## 🛡️ License & Academic Integrity

Developed as part of the 42 School Cybersecurity Piscine. Standard rules against prohibited scrapers (e.g. `wget`, `scrapy`) apply.

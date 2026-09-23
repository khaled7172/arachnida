# Arachnida

An introductory project to web scraping, recursive crawling, and image metadata analysis for the **42 Cybersecurity Piscine**.

---

## 📖 Overview

Arachnida consists of two CLI tools designed to interact with web assets and inspect binary file metadata:

1. **`spider`**: A recursive web crawler and image extractor that gathers media files from a target URL up to a configurable depth.
2. **`scorpion`**: An image metadata analyzer that parses and displays basic file properties along with EXIF tags. Includes bonus features to **modify**, **delete**, and manage metadata via an interactive **GUI**.

---

## 🛠️ Requirements & Installation

- **Python 3.8+**
- **Dependencies**:
  - `requests` (HTTP requests)
  - `beautifulsoup4` (HTML parsing)
  - `Pillow` (Image & EXIF extraction and modification)
  - `PyGObject` / `GTK 3` (For bonus GUI, preinstalled on Kali/Debian systems)

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

Parses and displays EXIF and file metadata from local image files. Includes bonus modification, deletion, and GUI.

```bash
./scorpion [-h] [-d] [-m TAG=VALUE] [-g] [FILE ...]
```

#### Options:
- `FILE...`: One or more image files to inspect (Mandatory).
- `-m TAG=VALUE`, `--modify TAG=VALUE`: Modify or add EXIF metadata tag(s) on target files (Bonus).
- `-d`, `--delete`: Strip and delete all metadata from target files (Bonus).
- `-g`, `--gui`: Launch the graphical interface for viewing and managing metadata (Bonus).

#### Examples:
```bash
# Mandatory: Inspect metadata
./scorpion data/sample.jpg data/photo.png

# Bonus 1: Modify EXIF metadata
./scorpion -m "Artist=Hacker 42" -m "Model=Nikon Z9" data/sample.jpg

# Bonus 1: Strip / Delete metadata
./scorpion -d data/sample.jpg

# Bonus 2: Launch Graphical User Interface
./scorpion --gui
./scorpion -g data/sample.jpg
```

---

## ⚖️ Project Structure

```text
.
├── .gitignore
├── README.md
├── aranchida.md
├── flags.md
├── requirements.txt
├── spider
├── spider.md
├── scorpion
├── scorpion.md
└── template.md
```

---

## 🛡️ License & Academic Integrity

Developed as part of the 42 School Cybersecurity Piscine. Standard rules against prohibited scrapers (e.g. `wget`, `scrapy`) apply.

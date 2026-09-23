# `scorpion` — Image Metadata & EXIF Analyzer

> **Pipeline position**: 2 of 2
> **What this file does in plain English**: This program inspects digital image files stored on your computer to uncover hidden details about how, when, and where they were created. Beyond basic information like file size and pixel dimensions, it digs into special embedded photographic data called EXIF. This data can reveal the exact camera or phone model used, lens settings, software edits, timestamps, and even physical GPS geographic coordinates pinpointing where a photograph was taken. As a privacy bonus, it also allows you to strip away all this sensitive metadata to keep your photos clean before sharing.
> **Files it depends on**: Python Standard Library (`os`, `sys`, `argparse`, `datetime`, `mimetypes`), `PIL` (Pillow)
> **Files that depend on it**: Terminal standard output / end-user forensic report

---

## 📑 Section 1: Line-by-Line Code Breakdown

---

## Lines 1–15: Shebang, Imports, and Supported Formats

### The Code
```python
#!/usr/bin/env python3
"""
Scorpion: Image metadata and EXIF parser for 42 Cybersecurity Piscine.
"""

import os
import sys
import argparse
import datetime
import mimetypes
from PIL import Image
from PIL.ExifTags import TAGS, GPSTAGS

VALID_EXTENSIONS = ('.jpg', '.jpeg', '.png', '.gif', '.bmp')
```

### Plain English
This section acts as the launchpad for the `scorpion` program. The very first line, known as the shebang, tells the operating system's terminal to run this script using Python 3. Following that, we load essential tools from Python's standard library to inspect file properties on the computer's hard drive and format timestamps into human-readable dates.

We also import Pillow's `Image` module along with two dictionary mappings: `TAGS` and `GPSTAGS`. Inside an image file, camera metadata is stored as raw numeric identifiers (such as tag number `271` or `306`). Pillow's tag dictionaries translate those cryptic numbers into meaningful English names like `"Make"` (camera manufacturer) and `"DateTime"`.

Finally, we define `VALID_EXTENSIONS`, a master list of all image formats that our program is designed to analyze, ensuring parity with the image formats downloaded by `spider`.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 1 | `#!/usr/bin/env python3` | Directs Unix operating systems to execute this file using the Python 3 interpreter. |
| 2 | `"""` | Opens a multi-line module documentation string. |
| 3 | `Scorpion: Image metadata and EXIF parser...` | Human-readable explanation of the program purpose. |
| 4 | `"""` | Closes the module documentation string. |
| 5 | *(empty line)* | Clean code spacing separating docstrings from import statements. |
| 6 | `import os` | Imports operating system module to read file paths, verify existence, and read file stats. |
| 7 | `import sys` | Imports system module to write error messages to `sys.stderr` and manage exit behaviors. |
| 8 | `import argparse` | Imports argument parsing library to handle file arguments and optional flags like `-d`. |
| 9 | `import datetime` | Imports date and time tools to convert raw Unix epoch timestamps into readable calendar dates. |
| 10 | `import mimetypes` | Imports MIME type guessing tools to classify files (e.g. `image/jpeg`). |
| 11 | `from PIL import Image` | Imports Pillow's core Image processing class to open and inspect binary image files. |
| 12 | `from PIL.ExifTags import TAGS, GPSTAGS` | Imports human-readable tag translation dictionaries for EXIF and GPS numeric IDs. |
| 13 | *(empty line)* | Clean code spacing between imports and constants. |
| 14 | `VALID_EXTENSIONS = ('.jpg', '.jpeg', '.png', '.gif', '.bmp')` | Defines tuple of supported image file extensions specified by the subject. |
| 15 | *(empty line)* | Visual separation before the first helper function. |

### New Concepts Introduced

> **`EXIF (Exchangeable Image File Format)`**: A standard specification that embeds technical metadata directly inside image files (such as JPEGs). EXIF records camera model, aperture, shutter speed, focal length, timestamps, and GPS coordinates.
>
> **`Tag Dictionary Lookup (TAGS & GPSTAGS)`**: Digital cameras save metadata as numeric codes (e.g. tag `0x0110` means "Model"). A tag dictionary acts like a phonebook, translating binary codes into human-readable keys.

---

## Lines 17–24: `format_file_size(size_in_bytes)`

### The Code
```python
def format_file_size(size_in_bytes):
    """Format raw byte size into human-readable string."""
    for unit in ['B', 'KB', 'MB', 'GB']:
        if size_in_bytes < 1024.0:
            return f"{size_in_bytes:.2f} {unit}"
        size_in_bytes /= 1024.0
    return f"{size_in_bytes:.2f} TB"
```

### Plain English
Operating systems measure file sizes in raw bytes. While a computer easily understands that a file is `14589234` bytes, human beings prefer reading `13.91 MB`. 

This helper function takes any raw integer number of bytes and repeatedly divides it by 1,024 (the standard binary step from Bytes to Kilobytes, Megabytes, and Gigabytes). As soon as the number drops below 1,024, it returns the number formatted with two decimal places and the appropriate size unit abbreviation.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 17 | `def format_file_size(size_in_bytes):` | Declares function that converts raw bytes into a scaled human-readable string. |
| 18 | `"""Format raw byte size into human-readable string."""` | Docstring explaining conversion purpose. |
| 19 | `for unit in ['B', 'KB', 'MB', 'GB']:` | Iterates through progressive byte size unit suffixes. |
| 20 | `if size_in_bytes < 1024.0:` | Checks if the current numeric magnitude is smaller than the next tier threshold. |
| 21 | `return f"{size_in_bytes:.2f} {unit}"` | Returns formatted string with two decimal places and the active unit. |
| 22 | `size_in_bytes /= 1024.0` | Divides byte count by 1024 to advance to the next unit tier. |
| 23 | `return f"{size_in_bytes:.2f} TB"` | Fallback return for extraordinarily large files in terabytes. |
| 24 | *(empty line)* | Visual separation between functions. |

---

## Lines 26–43: `get_basic_attributes(filepath)`

### The Code
```python
def get_basic_attributes(filepath):
    """Retrieve filesystem attributes for a given file path."""
    stat_info = os.stat(filepath)
    file_size_str = format_file_size(stat_info.st_size)
    mod_time = datetime.datetime.fromtimestamp(stat_info.st_mtime).strftime('%Y-%m-%d %H:%M:%S')
    creation_timestamp = getattr(stat_info, 'st_birthtime', stat_info.st_ctime)
    creation_time = datetime.datetime.fromtimestamp(creation_timestamp).strftime('%Y-%m-%d %H:%M:%S')
    mime_type, _ = mimetypes.guess_type(filepath)

    return {
        "File Path": os.path.abspath(filepath),
        "File Name": os.path.basename(filepath),
        "File Size": f"{stat_info.st_size} bytes ({file_size_str})",
        "MIME Type": mime_type or "Unknown",
        "Creation Date": creation_time,
        "Modification Date": mod_time
    }
```

### Plain English
Before inspecting what is inside the image, this function asks the computer's file system for metadata about the file itself as an object stored on the hard drive. 

It queries `os.stat` to discover the exact file size on disk, the moment the file was last modified, and when it was created. Because Unix systems track time as seconds elapsed since January 1, 1970 (known as Unix Epoch time), we convert those raw timestamps into standard calendar strings (`YYYY-MM-DD HH:MM:SS`). It also determines the MIME classification (such as `image/png`).

All of these baseline properties are packaged into an organized dictionary and returned to the caller.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 26 | `def get_basic_attributes(filepath):` | Declares function that inspects file system properties. |
| 27 | `"""Retrieve filesystem attributes for a given file path."""` | Docstring explaining operating system stat queries. |
| 28 | `stat_info = os.stat(filepath)` | Queries the operating system for inode/file metadata structure. |
| 29 | `file_size_str = format_file_size(stat_info.st_size)` | Formats the raw byte size using our helper function. |
| 30 | `mod_time = datetime.datetime.fromtimestamp(stat_info.st_mtime).strftime('%Y-%m-%d %H:%M:%S')` | Converts last modified Unix timestamp into a readable date string. |
| 31 | `creation_timestamp = getattr(stat_info, 'st_birthtime', stat_info.st_ctime)` | Retrieves file creation time (`st_birthtime` on macOS/BSD, fallback to `st_ctime` on Linux). |
| 32 | `creation_time = datetime.datetime.fromtimestamp(creation_timestamp).strftime('%Y-%m-%d %H:%M:%S')` | Formats the creation epoch timestamp into a readable date string. |
| 33 | `mime_type, _ = mimetypes.guess_type(filepath)` | Guesses the standardized MIME media type based on the file name. |
| 34 | *(empty line)* | Clean code spacing before constructing return dictionary. |
| 35 | `return {` | Begins dictionary containing collected basic attributes. |
| 36 | `"File Path": os.path.abspath(filepath),` | Stores the complete absolute path of the file on disk. |
| 37 | `"File Name": os.path.basename(filepath),` | Stores the standalone filename without directory components. |
| 38 | `"File Size": f"{stat_info.st_size} bytes ({file_size_str})",` | Stores both exact byte count and human-friendly size. |
| 39 | `"MIME Type": mime_type or "Unknown",` | Stores detected MIME type or defaults to `"Unknown"`. |
| 40 | `"Creation Date": creation_time,` | Stores formatted creation timestamp. |
| 41 | `"Modification Date": mod_time` | Stores formatted last modification timestamp. |
| 42 | `}` | Closes the dictionary literal. |
| 43 | *(empty line)* | Visual separation between functions. |

### New Concepts Introduced

> **`MIME Type (Multipurpose Internet Mail Extensions)`**: A standard identifier used across operating systems and the web to classify the format of a file (e.g., `image/jpeg`, `image/png`, `text/html`).
>
> **`Unix Timestamp (Epoch Time)`**: An integer representing the total number of seconds that have passed since 00:00:00 UTC on January 1, 1970.

---

## Lines 45–70: `parse_gps_data(gps_info_raw)`

### The Code
```python
def parse_gps_data(gps_info_raw):
    """Translate raw GPS EXIF tag dictionary into human-readable coordinates."""
    gps_decoded = {}
    for key, value in gps_info_raw.items():
        sub_tag_name = GPSTAGS.get(key, key)
        gps_decoded[sub_tag_name] = value

    lat = gps_decoded.get("GPSLatitude")
    lat_ref = gps_decoded.get("GPSLatitudeRef")
    lon = gps_decoded.get("GPSLongitude")
    lon_ref = gps_decoded.get("GPSLongitudeRef")

    if lat and lat_ref and lon and lon_ref:
        try:
            lat_deg = float(lat[0]) + float(lat[1]) / 60.0 + float(lat[2]) / 3600.0
            if lat_ref != "N":
                lat_deg = -lat_deg
            lon_deg = float(lon[0]) + float(lon[1]) / 60.0 + float(lon[2]) / 3600.0
            if lon_ref != "E":
                lon_deg = -lon_deg
            gps_decoded["Formatted Coordinates"] = f"{lat_deg:.6f}, {lon_deg:.6f} ({lat_ref} {lon_ref})"
        except Exception:
            pass

    return gps_decoded
```

### Plain English
When smartphones and GPS-equipped cameras capture a photo, they frequently record geographic coordinates inside a special sub-block of EXIF metadata. However, these coordinates are stored as mathematical fractions representing Degrees, Minutes, and Seconds (e.g. `(33, 53, 49.2)` North).

This function translates the raw GPS sub-tags into readable names using Pillow's `GPSTAGS`. Then, if latitude and longitude are both present, it calculates standard decimal GPS coordinates using the formula:
$$\text{Decimal Degrees} = \text{Degrees} + \frac{\text{Minutes}}{60} + \frac{\text{Seconds}}{3600}$$

If the reference direction is South (`S`) or West (`W`), it converts the number to a negative coordinate according to geographic convention. This produces a clean coordinate string (e.g. `33.897000, 35.478000 (N E)`) that can be directly pasted into Google Maps.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 45 | `def parse_gps_data(gps_info_raw):` | Declares function that processes raw GPS EXIF tag dictionary. |
| 46 | `"""Translate raw GPS EXIF tag dictionary into human-readable coordinates."""` | Docstring explaining GPS translation and coordinate calculation. |
| 47 | `gps_decoded = {}` | Initializes dictionary to hold decoded GPS tag names and values. |
| 48 | `for key, value in gps_info_raw.items():` | Iterates over each raw key-value pair in the raw GPS sub-dictionary. |
| 49 | `sub_tag_name = GPSTAGS.get(key, key)` | Translates integer GPS tag code into readable name (e.g. `"GPSLatitude"`). |
| 50 | `gps_decoded[sub_tag_name] = value` | Stores value under translated tag name in decoded dictionary. |
| 51 | *(empty line)* | Clean code spacing before coordinate extraction. |
| 52 | `lat = gps_decoded.get("GPSLatitude")` | Retrieves raw latitude tuple containing degrees, minutes, and seconds. |
| 53 | `lat_ref = gps_decoded.get("GPSLatitudeRef")` | Retrieves latitude hemisphere reference (`"N"` for North, `"S"` for South). |
| 54 | `lon = gps_decoded.get("GPSLongitude")` | Retrieves raw longitude tuple containing degrees, minutes, and seconds. |
| 55 | `lon_ref = gps_decoded.get("GPSLongitudeRef")` | Retrieves longitude hemisphere reference (`"E"` for East, `"W"` for West). |
| 56 | *(empty line)* | Clean code spacing before decimal coordinate calculation. |
| 57 | `if lat and lat_ref and lon and lon_ref:` | Confirms that complete GPS coordinate data exists. |
| 58 | `try:` | Enters protected block to catch arithmetic or parsing exceptions. |
| 59 | `lat_deg = float(lat[0]) + float(lat[1]) / 60.0 + float(lat[2]) / 3600.0` | Converts degrees, minutes, seconds into decimal degrees. |
| 60 | `if lat_ref != "N":` | Checks if location is in the Southern hemisphere. |
| 61 | `lat_deg = -lat_deg` | Negates latitude for southern coordinates. |
| 62 | `lon_deg = float(lon[0]) + float(lon[1]) / 60.0 + float(lon[2]) / 3600.0` | Converts longitude degrees, minutes, seconds into decimal degrees. |
| 63 | `if lon_ref != "E":` | Checks if location is in the Western hemisphere. |
| 64 | `lon_deg = -lon_deg` | Negates longitude for western coordinates. |
| 65 | `gps_decoded["Formatted Coordinates"] = f"{lat_deg:.6f}, {lon_deg:.6f} ({lat_ref} {lon_ref})"` | Adds clean human-readable GPS string ready for map lookup. |
| 66 | `except Exception:` | Catches any unexpected format anomalies in corrupted GPS tags. |
| 67 | `pass` | Safely ignores calculation failures and keeps raw tags. |
| 68 | *(empty line)* | Clean code spacing before return. |
| 69 | `return gps_decoded` | Returns decoded GPS metadata dictionary. |
| 70 | *(empty line)* | Visual separation between functions. |

---

## Lines 72–92: `extract_image_exif(image)`

### The Code
```python
def extract_image_exif(image):
    """Extract standard EXIF tags and decoded GPS dictionary from a PIL Image."""
    exif_data = {}
    gps_data = {}

    raw_exif = image._getexif() if hasattr(image, '_getexif') else None
    if raw_exif is not None:
        for tag_id, value in raw_exif.items():
            tag_name = TAGS.get(tag_id, tag_id)
            if tag_name == "GPSInfo" and isinstance(value, dict):
                gps_data = parse_gps_data(value)
            else:
                if isinstance(value, bytes):
                    try:
                        value = value.decode('utf-8', errors='replace').strip('\x00')
                    except Exception:
                        value = str(value)
                exif_data[tag_name] = value

    return exif_data, gps_data
```

### Plain English
This function handles the extraction of camera and shooting metadata from an open image file. It checks if the image object has an EXIF reader method (`_getexif()`).

If EXIF data is found, it iterates over every tag. It uses Pillow's `TAGS` dictionary to translate numeric tag IDs into familiar names like `"Make"`, `"Model"`, `"ExposureTime"`, `"FNumber"`, and `"ISOSpeedRatings"`. If it encounters `"GPSInfo"`, it forwards the dictionary to our GPS parser function. 

Additionally, camera firmware sometimes stores text fields as raw binary byte arrays terminating with null characters (`\x00`). This function safely decodes those byte strings into clean text so they print beautifully on screen without garbage characters.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 72 | `def extract_image_exif(image):` | Declares function that extracts EXIF and GPS tags from a PIL Image object. |
| 73 | `"""Extract standard EXIF tags and decoded GPS dictionary from a PIL Image."""` | Docstring explaining metadata parsing and decoding. |
| 74 | `exif_data = {}` | Initializes dictionary to hold decoded EXIF attributes. |
| 75 | `gps_data = {}` | Initializes dictionary to hold decoded GPS attributes. |
| 76 | *(empty line)* | Clean code spacing before accessing raw EXIF. |
| 77 | `raw_exif = image._getexif() if hasattr(image, '_getexif') else None` | Safely retrieves raw EXIF dictionary if supported by image format. |
| 78 | `if raw_exif is not None:` | Checks if the image actually contains an EXIF header block. |
| 79 | `for tag_id, value in raw_exif.items():` | Loops over every tag ID and its associated value in the EXIF block. |
| 80 | `tag_name = TAGS.get(tag_id, tag_id)` | Translates integer tag ID into human-readable string using Pillow's `TAGS`. |
| 81 | `if tag_name == "GPSInfo" and isinstance(value, dict):` | Detects whether the current tag holds the nested GPS dictionary. |
| 82 | `gps_data = parse_gps_data(value)` | Parses and formats the GPS data using our specialized helper. |
| 83 | `else:` | Executes for all general EXIF photographic and camera tags. |
| 84 | `if isinstance(value, bytes):` | Checks if the tag value is stored as raw binary bytes. |
| 85 | `try:` | Attempts UTF-8 text decoding. |
| 86 | `value = value.decode('utf-8', errors='replace').strip('\x00')` | Decodes bytes to string and strips trailing null terminators. |
| 87 | `except Exception:` | Catches non-UTF8 decoding failures. |
| 88 | `value = str(value)` | Converts binary values into printable string representation as fallback. |
| 89 | `exif_data[tag_name] = value` | Stores decoded value into our EXIF dictionary under its human-readable name. |
| 90 | *(empty line)* | Clean code spacing before return. |
| 91 | `return exif_data, gps_data` | Returns tuple of decoded EXIF dictionary and GPS dictionary. |
| 92 | *(empty line)* | Visual separation between functions. |

---

## Lines 94–106: `strip_image_metadata(filepath)` (Bonus)

### The Code
```python
def strip_image_metadata(filepath):
    """Bonus feature: strip and remove metadata from the target image file."""
    try:
        with Image.open(filepath) as img:
            clean_image = Image.new(img.mode, img.size)
            clean_image.paste(img)
            clean_image.save(filepath)
        print(f"[✔] Successfully stripped metadata from: {filepath}")
        return True
    except Exception as e:
        print(f"[-] Failed to strip metadata from {filepath}: {e}", file=sys.stderr)
        return False
```

### Plain English
This function fulfills the **Bonus Part** of the subject, which requests a feature allowing users to modify or delete metadata from a given file.

In digital photography and cybersecurity, removing metadata is called "metadata sanitization" or "photo scrubbing". When someone uploads an image to the internet, EXIF data could leak where they live or what phone they own.

To sanitize the file, this function opens the original image, copies only the pure visual image pixels into a brand-new image canvas with identical dimensions and color mode, and overwrites the file on disk. Because the EXIF and GPS headers are never copied to the new canvas, all metadata is permanently removed without degrading the visual pixels.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 94 | `def strip_image_metadata(filepath):` | Declares the bonus metadata sanitization function. |
| 95 | `"""Bonus feature: strip and remove metadata from the target image file."""` | Docstring explaining metadata stripping. |
| 96 | `try:` | Starts protected block to handle image IO and file write exceptions. |
| 97 | `with Image.open(filepath) as img:` | Opens the target image safely with automatic resource closure. |
| 98 | `clean_image = Image.new(img.mode, img.size)` | Creates a new blank image canvas preserving color mode and dimensions. |
| 99 | `clean_image.paste(img)` | Copies raw pixel color data onto the new canvas without copying EXIF headers. |
| 100 | `clean_image.save(filepath)` | Saves sanitized image directly back to the original file path. |
| 101 | `print(f"[✔] Successfully stripped metadata from: {filepath}")` | Prints success message confirming metadata removal. |
| 102 | `return True` | Returns `True` indicating successful sanitization. |
| 103 | `except Exception as e:` | Catches any error during image opening or saving. |
| 104 | `print(f"[-] Failed to strip metadata from {filepath}: {e}", file=sys.stderr)` | Prints error notification to standard error stream. |
| 105 | `return False` | Returns `False` on failure. |
| 106 | *(empty line)* | Visual separation between functions. |

---

## Lines 108–165: `display_metadata(filepath)`

### The Code
```python
def display_metadata(filepath):
    """Print structured basic attributes and EXIF metadata to the console."""
    print("=" * 70)
    print(f"  SCORPION METADATA REPORT: {os.path.basename(filepath)}")
    print("=" * 70)

    if not os.path.isfile(filepath):
        print(f"Error: File not found: '{filepath}'", file=sys.stderr)
        return

    lower_path = filepath.lower()
    if not any(lower_path.endswith(ext) for ext in VALID_EXTENSIONS):
        print(f"Warning: '{filepath}' does not have a standard supported extension {VALID_EXTENSIONS}", file=sys.stderr)

    basic_attrs = get_basic_attributes(filepath)
    print("\n[+] BASIC FILE ATTRIBUTES:")
    for key, value in basic_attrs.items():
        print(f"    • {key:<20}: {value}")

    try:
        with Image.open(filepath) as img:
            print("\n[+] IMAGE CHARACTERISTICS:")
            print(f"    • {'Format':<20}: {img.format}")
            print(f"    • {'Dimensions':<20}: {img.width} x {img.height} pixels")
            print(f"    • {'Color Mode':<20}: {img.mode}")
            print(f"    • {'Is Animated':<20}: {getattr(img, 'is_animated', False)}")

            # Extract EXIF & GPS
            exif_dict, gps_dict = extract_image_exif(img)

            print("\n[+] EXIF METADATA:")
            if exif_dict:
                for tag_name, tag_val in sorted(exif_dict.items()):
                    val_str = str(tag_val)
                    if len(val_str) > 80:
                        val_str = val_str[:77] + "..."
                    print(f"    • {tag_name:<25}: {val_str}")
            else:
                print("    (No standard EXIF metadata present in this image)")

            if gps_dict:
                print("\n[+] GPS LOCATION METADATA:")
                for gps_tag, gps_val in sorted(gps_dict.items()):
                    print(f"    • {gps_tag:<25}: {gps_val}")

            # Check for PNG or other text chunks
            if hasattr(img, 'info') and img.info:
                text_info = {k: v for k, v in img.info.items() if k not in ('exif', 'transparency') and isinstance(v, (str, int, float))}
                if text_info:
                    print("\n[+] EMBEDDED TEXT CHUNKS / INFO:")
                    for info_key, info_val in text_info.items():
                        print(f"    • {info_key:<25}: {info_val}")

    except Exception as e:
        print(f"\n[-] Could not parse image metadata: {e}", file=sys.stderr)

    print()
```

### Plain English
This function is responsible for formatting and presenting all the gathered information to the user in a clean, professional console report.

It divides the output into clear sections:
1. **Header Banner**: Clearly states which file is currently being examined.
2. **Basic File Attributes**: File location, byte size, MIME type, and operating system creation/modification timestamps.
3. **Image Characteristics**: The internal image format (JPEG, PNG, GIF), resolution in pixels (width $\times$ height), and color channel mode (RGB, RGBA, Grayscale).
4. **EXIF Metadata**: All camera and capture tags, truncated gracefully if any text field exceeds 80 characters so the terminal remains readable.
5. **GPS Location**: Coordinates and formatted geographical location if available.
6. **Embedded Text Chunks**: Inspects extra comment chunks commonly found in PNG files (such as author comments or software tags).

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 108 | `def display_metadata(filepath):` | Declares function that coordinates report generation and printing. |
| 109 | `"""Print structured basic attributes and EXIF metadata to the console."""` | Docstring explaining reporting behavior. |
| 110 | `print("=" * 70)` | Prints visual divider bar of 70 equals signs. |
| 111 | `print(f"  SCORPION METADATA REPORT: {os.path.basename(filepath)}")` | Prints centered report title with target file name. |
| 112 | `print("=" * 70)` | Prints closing divider bar. |
| 113 | *(empty line)* | Clean code spacing before existence validation. |
| 114 | `if not os.path.isfile(filepath):` | Verifies whether the specified path points to an existing file. |
| 115 | `print(f"Error: File not found: '{filepath}'", file=sys.stderr)` | Writes file-not-found error to stderr stream. |
| 116 | `return` | Halts execution for non-existent files. |
| 117 | *(empty line)* | Clean code spacing before extension check. |
| 118 | `lower_path = filepath.lower()` | Normalizes file path to lowercase for extension verification. |
| 119 | `if not any(lower_path.endswith(ext) for ext in VALID_EXTENSIONS):` | Checks if extension is outside the standard list. |
| 120 | `print(f"Warning: '{filepath}' does not have a standard supported extension...", file=sys.stderr)` | Prints non-fatal warning for unusual file types. |
| 121 | *(empty line)* | Clean spacing before displaying basic attributes. |
| 122 | `basic_attrs = get_basic_attributes(filepath)` | Retrieves file system metadata dictionary. |
| 123 | `print("\n[+] BASIC FILE ATTRIBUTES:")` | Prints section header for basic file properties. |
| 124 | `for key, value in basic_attrs.items():` | Iterates over each basic property. |
| 125 | `print(f"    • {key:<20}: {value}")` | Prints aligned key-value pair indented with bullet points. |
| 126 | *(empty line)* | Clean spacing before image analysis block. |
| 127 | `try:` | Starts protected block to handle Pillow image decoding. |
| 128 | `with Image.open(filepath) as img:` | Opens image using Pillow context manager. |
| 129 | `print("\n[+] IMAGE CHARACTERISTICS:")` | Prints image characteristics section header. |
| 130 | `print(f"    • {'Format':<20}: {img.format}")` | Prints detected image format (e.g. JPEG, PNG). |
| 131 | `print(f"    • {'Dimensions':<20}: {img.width} x {img.height} pixels")` | Prints image resolution in pixels. |
| 132 | `print(f"    • {'Color Mode':<20}: {img.mode}")` | Prints color representation mode (e.g. RGB, RGBA). |
| 133 | `print(f"    • {'Is Animated':<20}: {getattr(img, 'is_animated', False)}")` | Prints whether image contains multiple animation frames (e.g. GIF). |
| 134 | *(empty line)* | Clean code spacing before EXIF extraction. |
| 135 | `# Extract EXIF & GPS` | Comment indicating EXIF and GPS analysis. |
| 136 | `exif_dict, gps_dict = extract_image_exif(img)` | Extracts decoded EXIF and GPS dictionaries. |
| 137 | *(empty line)* | Clean spacing before printing EXIF. |
| 138 | `print("\n[+] EXIF METADATA:")` | Prints EXIF section header. |
| 139 | `if exif_dict:` | Checks if image contains any decoded EXIF tags. |
| 140 | `for tag_name, tag_val in sorted(exif_dict.items()):` | Iterates over alphabetically sorted EXIF tags. |
| 141 | `val_str = str(tag_val)` | Converts tag value to string. |
| 142 | `if len(val_str) > 80:` | Checks if value string is excessively long. |
| 143 | `val_str = val_str[:77] + "..."` | Truncates value with ellipsis to prevent messy terminal wrapping. |
| 144 | `print(f"    • {tag_name:<25}: {val_str}")` | Prints aligned EXIF tag name and value. |
| 145 | `else:` | Executes when image has no EXIF block. |
| 146 | `print("    (No standard EXIF metadata present in this image)")` | Prints informative notice that EXIF data is absent. |
| 147 | *(empty line)* | Clean code spacing before GPS section. |
| 148 | `if gps_dict:` | Checks if decoded GPS location tags were found. |
| 149 | `print("\n[+] GPS LOCATION METADATA:")` | Prints GPS section header. |
| 150 | `for gps_tag, gps_val in sorted(gps_dict.items()):` | Iterates through sorted GPS tags. |
| 151 | `print(f"    • {gps_tag:<25}: {gps_val}")` | Prints formatted GPS tag and value. |
| 152 | *(empty line)* | Clean code spacing before embedded info section. |
| 153 | `# Check for PNG or other text chunks` | Comment denoting inspection of PNG metadata chunks. |
| 154 | `if hasattr(img, 'info') and img.info:` | Checks if image container has an extra info dictionary. |
| 155 | `text_info = {k: v for k, v in img.info.items() if k not in ('exif', 'transparency') and isinstance(v, (str, int, float))}` | Filters out binary blobs to isolate readable textual metadata. |
| 156 | `if text_info:` | Checks if any readable textual chunks exist. |
| 157 | `print("\n[+] EMBEDDED TEXT CHUNKS / INFO:")` | Prints section header for embedded text chunks. |
| 158 | `for info_key, info_val in text_info.items():` | Iterates through text chunk keys and values. |
| 159 | `print(f"    • {info_key:<25}: {info_val}")` | Prints aligned text metadata. |
| 160 | *(empty line)* | Clean code spacing before exception handler. |
| 161 | `except Exception as e:` | Catches corrupted image decoding errors. |
| 162 | `print(f"\n[-] Could not parse image metadata: {e}", file=sys.stderr)` | Prints error diagnostic message to stderr. |
| 163 | *(empty line)* | Clean code spacing before terminal newline. |
| 164 | `print()` | Prints trailing newline for clean spacing between multiple files. |
| 165 | *(empty line)* | Visual separation between functions. |

---

## Lines 167–186: `parse_arguments()`

### The Code
```python
def parse_arguments():
    """Parse command line arguments for scorpion."""
    parser = argparse.ArgumentParser(
        prog="scorpion",
        description="Extract and parse EXIF and filesystem metadata from image files."
    )
    parser.add_argument(
        "files",
        nargs="+",
        metavar="FILE",
        help="One or more image files to analyze."
    )
    parser.add_argument(
        "-d", "--delete",
        action="store_true",
        dest="delete_metadata",
        help="Bonus: Strip and delete metadata from the specified image files."
    )
    return parser.parse_args()
```

### Plain English
This function sets up the command-line argument reader for `scorpion`. According to the subject, `scorpion` must accept one or more files as parameters (`./scorpion FILE1 [FILE2 ...]`).

By specifying `nargs="+"`, the argument parser allows the user to pass as many file paths as they want in a single command, grouping them into a convenient Python list. In addition, we add an optional bonus flag (`-d` or `--delete`), which allows users to sanitize files when desired.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 167 | `def parse_arguments():` | Declares CLI argument parser function for scorpion. |
| 168 | `"""Parse command line arguments for scorpion."""` | Docstring explaining CLI options. |
| 169 | `parser = argparse.ArgumentParser(` | Instantiates ArgumentParser object. |
| 170 | `prog="scorpion",` | Sets program name shown in terminal help banner. |
| 171 | `description="Extract and parse EXIF and filesystem metadata from image files."` | Sets explanatory banner description. |
| 172 | `)` | Closes ArgumentParser configuration. |
| 173 | `parser.add_argument(` | Begins adding positional argument for target files. |
| 174 | `"files",` | Names the parsed attribute storing file paths. |
| 175 | `nargs="+",` | Enforces that at least one (or more) file paths must be supplied. |
| 176 | `metavar="FILE",` | Placeholder displayed in usage message (`FILE [FILE ...]`). |
| 177 | `help="One or more image files to analyze."` | Help description for positional file arguments. |
| 178 | `)` | Closes positional file argument configuration. |
| 179 | `parser.add_argument(` | Begins adding optional bonus deletion flag. |
| 180 | `"-d", "--delete",` | Flags for stripping metadata. |
| 181 | `action="store_true",` | Stores `True` if flag is passed, `False` otherwise. |
| 182 | `dest="delete_metadata",` | Sets attribute name storing bonus deletion decision. |
| 183 | `help="Bonus: Strip and delete metadata from the specified image files."` | Help menu description explaining bonus feature. |
| 184 | `)` | Closes bonus flag configuration. |
| 185 | `return parser.parse_args()` | Parses command line tokens and returns populated namespace. |
| 186 | *(empty line)* | Visual separation between functions. |

---

## Lines 188–200: `main()` and Entry Point Guard

### The Code
```python
def main():
    args = parse_arguments()

    for file_path in args.files:
        if args.delete_metadata:
            strip_image_metadata(file_path)
        else:
            display_metadata(file_path)


if __name__ == "__main__":
    main()
```

### Plain English
This is the program coordinator. It calls `parse_arguments()` to get the list of files provided by the user. 

It then iterates over each file in sequence. If the user passed the `-d` bonus flag, it strips and deletes the metadata from the file. If `-d` was not passed (the standard mandatory behavior), it calls `display_metadata(file_path)` to generate and print the full forensic report for each image.

The `if __name__ == "__main__":` guard ensures that this code executes immediately when `./scorpion` is invoked from the command line.

### Line-by-Line Breakdown
| Line | Code | What it does |
|------|------|-------------|
| 188 | `def main():` | Declares program's top-level coordinator function. |
| 189 | `args = parse_arguments()` | Parses command-line arguments and flags. |
| 190 | *(empty line)* | Clean code spacing before file iteration loop. |
| 191 | `for file_path in args.files:` | Iterates over each file path supplied by the user. |
| 192 | `if args.delete_metadata:` | Checks if user activated bonus metadata sanitization mode. |
| 193 | `strip_image_metadata(file_path)` | Strips all metadata from the target image file. |
| 194 | `else:` | Executes default mandatory metadata inspection mode. |
| 195 | `display_metadata(file_path)` | Generates and prints comprehensive metadata report. |
| 196 | *(empty line)* | Clean code spacing before entry point guard. |
| 197 | *(empty line)* | Clean code spacing. |
| 198 | `if __name__ == "__main__":` | Checks if script is being executed directly in terminal. |
| 199 | `main()` | Invokes `main()` function to initiate processing. |
| 200 | *(empty line)* | Trailing newline at end of source file. |

---

## 📊 Section 3: Complete ASCII Data Flow Diagram

```text
[User Terminal Command]
    ./scorpion [-d] FILE1 [FILE2 ...]
                    │
                    ▼
          `parse_arguments()`
  (Parses 1+ files & optional bonus flag `-d`)
                    │
                    ▼
         For each `file_path`:
         ┌──────────┴──────────┐
         │ Is `-d` requested?  │
         └──────────┬──────────┘
              ├── Yes ──► `strip_image_metadata(file_path)`
              │            └── Copy pixels to new blank canvas & re-save
              │
              └── No  ──► `display_metadata(file_path)`
                           │
                           ├── 1. `get_basic_attributes()`
                           │      (File size, MIME, creation/modification dates)
                           │
                           ├── 2. Pillow `Image.open()`
                           │      (Format, Dimensions WxH, Color Mode)
                           │
                           ├── 3. `extract_image_exif()`
                           │      ├── Translate tag IDs via `TAGS`
                           │      └── Extract & Decode text strings
                           │
                           ├── 4. `parse_gps_data()` (if present)
                           │      ├── Translate via `GPSTAGS`
                           │      └── Calculate Decimal Coordinates
                           │
                           └── 5. Render Structured Forensic Report to Console
```

---

## 🧠 Section 4: Key Takeaways

1. **Dual Metadata Layers**: Image files contain two separate categories of metadata: **filesystem attributes** (managed by the OS kernel: file size, timestamps) and **internal container metadata** (EXIF, GPS, PNG text chunks stored inside the file bytes). `scorpion` captures both.
2. **Safe Decoding & Normalization**: Binary fields and null-padded bytes are safely converted to clean UTF-8 strings, and numerical degree-minute-second GPS tuples are computed into real-world geographic coordinates.
3. **Pixel-Perfect Metadata Sanitization (Bonus)**: Copying raw visual pixel arrays to a clean canvas and saving strips all hidden tracking data while leaving the visual image completely untouched.

---

## 📎 Section 5: Concepts Introduced in This File

| Concept | First seen at line | Quick definition |
|---------|-------------------|-----------------|
| EXIF Metadata | 12 | Standardized camera, sensor, and capture metadata embedded in image files. |
| Tag Dictionary Mapping (`TAGS`, `GPSTAGS`) | 12 | Translation tables converting numeric EXIF tag keys into readable English strings. |
| Operating System File Stats (`os.stat`) | 28 | System call retrieving file metadata like size and timestamps directly from the filesystem inode. |
| Variable Positional Arguments (`nargs="+"`) | 175 | Argparse setting that captures one or more CLI parameters into a list. |
| GPS Coordinate Conversion | 59 | Converting Degrees-Minutes-Seconds tuples into signed decimal geographical coordinates. |
| Metadata Stripping / Sanitization | 98 | Removing hidden tracking tags by isolating visual pixels from header blocks. |

---

## ➡️ Section 6: What's Next

**Arachnida Pipeline Complete!**
You now have:
1. **`spider`**: Gathers all images from any web page recursively up to specified depth limits.
2. **`scorpion`**: Extracts, audits, and strips EXIF and file metadata from any image.

Both programs are fully tested, self-contained, executable, and documented line-by-line!

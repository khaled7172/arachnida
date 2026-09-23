# Arachnida — CLI Flags & Usage Guide (`flags.md`)

This guide provides a comprehensive breakdown of all command-line flags, options, syntax variations, and edge cases for **`spider`** and **`scorpion`**.

---

## 🕷️ 1. Spider Flags (`./spider`)

### Synopsis
```bash
./spider [-rlp] URL
```

### Options Breakdown

| Flag | Argument | Default Value | Description |
|---|---|---|---|
| *(none)* | *(none)* | Non-recursive | Crawls **only** the single page at `URL` without following any hyperlinks. |
| `-r` | None | `False` | Enables recursive download. Follows internal page hyperlinks up to the maximum depth. |
| `-l` | `N` (Integer $\ge 0$) | `5` | Sets the maximum recursion depth. If `-l` is omitted when `-r` is active, depth defaults to `5`. |
| `-p` | `PATH` (String) | `./data/` | Specifies the local folder where downloaded images are saved. Creates the directory if it does not exist. |
| `URL` | String | *(Required)* | The target web address (must include `http://` or `https://`). |

---

### Syntax Variations Supported

Thanks to standard POSIX argument parsing rules, all the following invocations are valid:

1. **Separated flags (Standard)**:
   ```bash
   ./spider -r -l 3 -p ./my_images/ https://example.com
   ```
2. **Joined flag and value**:
   ```bash
   ./spider -r -l3 -p./my_images/ https://example.com
   ```
3. **Combined short flags**:
   ```bash
   ./spider -rl 3 -p ./my_images/ https://example.com
   ```
4. **Minimal invocation (Single page, default folder `./data/`)**:
   ```bash
   ./spider https://example.com
   ```
5. **Recursive with all defaults (Depth `5`, folder `./data/`)**:
   ```bash
   ./spider -r https://example.com
   ```

---

### Behavior Matrix

| Command | Recursive? | Max Depth | Save Folder | Target |
|---|---|---|---|---|
| `./spider https://example.com` | ❌ No | `0` (Root only) | `./data/` | Only images directly on `example.com` |
| `./spider -p ./downloads https://example.com` | ❌ No | `0` (Root only) | `./downloads/` | Only images on root, saved to `./downloads` |
| `./spider -r https://example.com` | ✅ Yes | `5` (Default) | `./data/` | Follows links up to 5 levels deep |
| `./spider -r -l 2 https://example.com` | ✅ Yes | `2` | `./data/` | Follows links up to 2 levels deep |
| `./spider -r -l 3 -p ./saved https://example.com` | ✅ Yes | `3` | `./saved/` | Follows links 3 levels deep, saved to `./saved` |

---

### Edge Cases & Validations

1. **Negative Depth (`-l -1`)**:
   ```bash
   ./spider -r -l -1 https://example.com
   # Output: spider: error: Recursion depth level must be a non-negative integer. (Exit code 2)
   ```
2. **Missing Scheme (`example.com` without `https://`)**:
   ```bash
   ./spider example.com
   # Output: Error: Invalid URL 'example.com'. Must include http:// or https:// (Exit code 1)
   ```
3. **Non-Existent Save Directory**:
   - `spider` automatically creates the entire folder tree via `os.makedirs(save_dir, exist_ok=True)`.
4. **Duplicate Images on Different URLs**:
   - Computes SHA-256 fingerprint of image bytes. Identical images will **not** be saved twice.
5. **Identical File Names with Different Contents**:
   - Automatically appends numeric suffixes (e.g. `logo.png` $\to$ `logo_1.png` $\to$ `logo_2.png`) to prevent overwriting existing files.

---

## 🦂 2. Scorpion Flags (`./scorpion`)

### Synopsis
```bash
./scorpion [-d] FILE1 [FILE2 ...]
```

### Options Breakdown

| Flag | Argument | Mode | Description |
|---|---|---|---|
| *(none)* | `FILE [FILE ...]` | **Inspect** (Default) | Extracts and displays basic filesystem attributes, image format, dimensions, EXIF tags, GPS coordinates, and embedded text chunks. |
| `-d`, `--delete` | `FILE [FILE ...]` | **Sanitize** (Bonus) | Strips and permanently deletes all EXIF, GPS, and metadata from the specified file(s) while preserving visual pixels. |
| `-h`, `--help` | None | **Help** | Displays help message and exits. |

---

### Usage Examples

1. **Inspect a Single Image**:
   ```bash
   ./scorpion data/sample.jpg
   ```

2. **Inspect Multiple Images in One Run**:
   ```bash
   ./scorpion data/pic1.jpg data/pic2.png data/animation.gif
   ```

3. **Inspect with Shell Globbing / Wildcards**:
   ```bash
   ./scorpion data/*.jpg
   ```

4. **Bonus: Delete / Strip Metadata from a File**:
   ```bash
   ./scorpion -d data/private_photo.jpg
   ```

5. **Bonus: Strip Metadata from Multiple Files**:
   ```bash
   ./scorpion -d data/*.jpg
   ```

---

### Verification Workflow for Peer Evaluation

You can demonstrate that both programs and the bonus feature work end-to-end using this simple 4-step sequence:

```bash
# Step 1: Download images using spider
./spider -r -l 1 -p ./eval_data/ https://httpbin.org/

# Step 2: Check metadata on a downloaded file
./scorpion eval_data/sample.jpg

# Step 3: Strip metadata using the bonus flag (-d)
./scorpion -d eval_data/sample.jpg

# Step 4: Re-inspect the file to confirm metadata was stripped
./scorpion eval_data/sample.jpg
```
*(On Step 4, EXIF metadata will display: `(No standard EXIF metadata present in this image)`, proving successful sanitization!)*

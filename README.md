# ImgsOptimizer

> Lightweight batch image optimization daemon for Linux — automatically compresses JPEG and PNG files as they appear in a watched directory.

![Shell](https://img.shields.io/badge/language-Shell-89e051?logo=gnu-bash&logoColor=white)
![Platform](https://img.shields.io/badge/platform-Linux-lightgrey?logo=linux)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## How It Works

1. **On start** — scans the target directory and optimizes all images that haven't been processed yet
2. **While running** — listens for new files via `inotifywait` and optimizes them as they appear
3. **Tracking** — each processed file is marked with an extended filesystem attribute (`optimized=yes`) so it is never processed twice

```
new file detected
      │
      ▼
jpgpngoptimizer
  ├── JPEG → jpegoptim (lossless, strip metadata)
  └── PNG  → optipng → advpng → pngcrush (strip auxiliary chunks)
      │
      ▼
attr -s optimized -V yes <file>
```

---

## Scripts

| Script | Description |
|---|---|
| `imgsoptimizer` | Main daemon — watches a directory and triggers optimization on new files |
| `jpgpngoptimizer` | Optimizes a single JPEG or PNG file |
| `findimgswithoutattr` | Scans a directory and optimizes all files not yet marked as optimized |
| `imgsoptimizer-init` | LSB-compliant init.d script for running the daemon as a system service |

---

## Requirements

**Install dependencies (Debian/Ubuntu):**

```bash
sudo apt-get install jpegoptim optipng advancecomp pngcrush inotify-tools attr
```

| Tool | Purpose |
|---|---|
| `jpegoptim` | Lossless JPEG compression, strips EXIF/metadata |
| `optipng` | PNG optimization (re-encodes with maximum compression) |
| `advpng` | Further PNG compression using zlib/7zip |
| `pngcrush` | Strips auxiliary PNG chunks (gamma, color profile, timestamps) |
| `inotify-tools` | Filesystem event monitoring (`inotifywait`) |
| `attr` | Read/write extended filesystem attributes |

**Filesystem requirement:** the target partition must be mounted with the `user_xattr` option to support extended attributes.

```bash
# Example: enable xattr on mount
mount -o remount,user_xattr /dev/sdXY /mount/point

# Or add to /etc/fstab permanently:
# UUID=...  /mount/point  ext4  defaults,user_xattr  0 2
```

---

## Installation

**1. Clone the repository:**

```bash
git clone https://github.com/Incred/imgsoptimizer.git
cd imgsoptimizer
chmod +x imgsoptimizer jpgpngoptimizer findimgswithoutattr
```

The scripts automatically detect their own location, so they will find each other
regardless of where you place them — no manual path editing required.

**2. (Optional) Install as a system service:**

Open `imgsoptimizer-init` and edit the three variables at the top of the file:

```bash
DAEMON=/usr/local/bin/imgsoptimizer   # Full path to the imgsoptimizer script
USER="www-data"                        # User to run the daemon as
WATCHING_DIR="/var/www/media/"         # Directory to watch for new images
```

Then install and start the service:

```bash
sudo cp imgsoptimizer-init /etc/init.d/imgsoptimizer
sudo chmod +x /etc/init.d/imgsoptimizer
sudo update-rc.d imgsoptimizer defaults
sudo service imgsoptimizer start
```

---

## Usage

**Run the daemon** (watches a directory for new images):

```bash
./imgsoptimizer /path/to/images/
```

**Process existing files** (one-time scan of a directory):

```bash
./findimgswithoutattr /path/to/images/
```

**Optimize a single file:**

```bash
./jpgpngoptimizer /path/to/image.jpg
./jpgpngoptimizer /path/to/image.png
```

**Manage the service** (if installed via init.d):

```bash
sudo service imgsoptimizer start
sudo service imgsoptimizer stop
sudo service imgsoptimizer status
sudo service imgsoptimizer restart
```

---

## Notes

- Optimization is **lossless** for both JPEG and PNG — no quality degradation
- JPEG files have their metadata (EXIF, IPTC) stripped during optimization
- PNG files have auxiliary chunks (gamma, color profile, timestamps) removed
- Logs are written to `/var/log/imgsoptimizer.log` when running as a service

---

## License

MIT © [Incred](https://github.com/Incred)

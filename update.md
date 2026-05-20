# parse-uav-imgs — DJI Drone Image Sorter

Sort and map geotagged UAV images by flight. Reads EXIF metadata, groups images into per-flight subfolders, and optionally exports a point shapefile of image centroids.

**Original script:** Andy Lyons, 2017 (v0.91)  
**Updated for:** Python 3.10+, Linux (Ubuntu), DJI multispectral workflows

---

## Requirements

### System
- Python 3.x
- `exiftool` — must be on your PATH

Install exiftool on Ubuntu:
```bash
sudo apt install libimage-exiftool-perl
```

### Python packages
```bash
python3 -m pip install colorama
```

Optional (for shapefile export):
```bash
python3 -m pip install gdal
```

> **Important:** Always use `python3 -m pip install` on this machine to ensure packages install to the correct Python environment.

---

## Updates Made (from original)

The following changes were made to the original script to run on Python 3.10+ / Linux:

1. **Replaced deprecated imports**
   ```python
   # Old
   import imp
   from distutils import spawn

   # New
   import importlib.util
   import shutil
   ```

2. **Fixed colorama check**
   ```python
   # Old
   imp.find_module('colorama')

   # New
   if importlib.util.find_spec('colorama') is None:
   ```

3. **Fixed exiftool check**
   ```python
   # Old
   spawn.find_executable("exiftool")

   # New
   shutil.which("exiftool")
   ```

4. **Fixed osgeo/GDAL check**
   ```python
   # Old
   imp.find_module('osgeo')

   # New
   if importlib.util.find_spec('osgeo') is not None:
   ```

5. **Removed `os.system("pause")`** — Windows-only, causes silent failures on Linux. Replaced with `input("Press Enter to continue...")` where needed.

6. **Fixed `\w` syntax warning** — raw string added to Windows path print statement.

7. **Relaxed exiftool error check** — exit code `1` means exiftool skipped non-image files (`.nav`, `.bin`, `.MRK`), not a real error. Only exit code `2` is treated as failure:
   ```python
   if created_csv == 2:
   ```

8. **Removed filesize filter** — the `-if "$filesize# > 0"` condition caused all files to fail on newer exiftool versions. Removed entirely.

---

## Configuration

Edit these options at the top of the script before running:

| Option | Default | Description |
|--------|---------|-------------|
| `add_yaw` | `True` | Read DJI yaw tags. Set `False` if tags are missing (check with `exiftool image.JPG \| grep -i yaw`) |
| `m2s_YN` | `False` | Set `True` to actually move/copy files into flight subfolders |
| `m2s_MoveCopy` | `"move"` | Set to `"copy"` to keep originals in place |
| `m2s_DivideTifJpgYN` | `False` | Set `True` to split JPGs and TIFs into separate subfolders (useful for multispectral) |
| `shpCreateYN` | `False` | Set `True` to export a point shapefile per flight (requires GDAL) |
| `m2s_ThreshVal` | `10` | Gap multiplier used to detect flight breaks |
| `m2s_SubdirTemplate` | `Flt{FltNum}_{StartTime}_{EndTime}` | Naming template for output subfolders |

---

## Running

```bash
python parse-uav-imgs.py /path/to/your/images/
```

**Example:**
```bash
python parse-uav-imgs.py /Extra2/s3_staging/2026-04-06/
```

The script will:
1. Run exiftool on all images in the directory
2. Write an `exif_info.csv` to the same folder
3. Show an interactive menu — press `y` to proceed or adjust options before confirming

### Interactive menu options

| Key | Action |
|-----|--------|
| `y` | Proceed with current settings |
| `n` | Cancel and quit |
| `d` | Toggle flight sorting on/off |
| `m` / `c` | Switch between move and copy |
| `p` | Toggle JPG/TIF separation |
| `s` | Toggle shapefile export |
| `f` | Set first flight number |
| `v` | Set time gap threshold value |

---

## Output

| File/Folder | Description |
|-------------|-------------|
| `exif_info.csv` | EXIF metadata table for all images (GPS, timestamp, yaw) |
| `Flt1_HHMMSS_HHMMSS/` | Per-flight subfolders (if sorting enabled) |
| `Flt1_HHMMSS_HHMMSS_pts.shp` | Point shapefile of image centroids (if enabled) |

The shapefile can be loaded directly into QGIS for flight QA.

---

## Notes

- The script detects flight breaks by looking for time gaps between consecutive images. The default threshold is 10× the median capture interval.
- Non-image files (`.nav`, `.bin`, `.MRK`) in the folder are skipped by exiftool — this is expected and not an error.
- Tested on: Ubuntu 22.04, Python 3.10, exiftool 12.x, DJI Mavic 3 Multispectral imagery.

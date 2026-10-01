# 🚀 ROMMULATOR: God-Mode Filesystem Firmware Edition

> **Custom Fork of RomM (v3.10.3 Stable)**  
> Modified for seamless, zero-UI, filesystem-first firmware management, RetroArch native fallbacks, and multi-file Kickstart/BIOS injection.

---

## 💡 The "Why" & The Problem Solved

### The Official RomM Limitation
In official RomM builds, firmware (BIOS) files are managed through a database-driven UI. This approach introduces several major pain points for self-hosted server owners:
1. **Manual Upload Bottleneck:** You are forced to manually upload every single BIOS file through a web UI.
2. **Case Sensitivity & Naming Failures:** Cores like Amiga PUAE strictly require exact filenames (`kick34005.A500` with capital `A500`). Official RomM often rejects or fails to stream these properly.
3. **Single-File Restriction:** Official builds force the user to pick a *single* BIOS file per console. RetroArch cores (like PSX, Sega CD, or Amiga) natively support multiple region/version BIOS files with automatic fallback logic.
4. **Archive Filtering:** Official builds strip out `.zip` archives or fail on multi-file firmware sets.

### The ROMMULATOR Solution ("God-Mode" Firmware)
ROMMULATOR completely bypasses the database for firmware.
- **Zero-UI Configuration:** Put your BIOS files in `/romm/library/bios/{platform_slug}/` on your server drive. They are automatically detected and served.
- **Direct Filesystem Endpoint (`/fs_content/`):** A custom FastAPI endpoint streams BIOS files directly from disk without touching database BLOBs.
- **RetroArch Native Mounting (`EJS_externalFiles`):** Every BIOS file inside a platform's folder is simultaneously mounted into the browser's WebAssembly RetroArch filesystem (`/home/web_user/retroarch/userdata/system/`).
- **Automatic Core Fallback:** Because all BIOS files for a console are mounted at once, RetroArch automatically uses the exact BIOS needed for the region/game, handling fallbacks natively.

---

## 🏗️ Architecture & Code Modifications

### 1. Backend (`backend/endpoints/firmware.py`)
- **`get_platform_firmware` Interception:** Overridden to bypass `db_firmware_handler`. When a game loads, Python scans `/romm/library/bios/{platform_slug}/` on the host filesystem and returns a dynamic, deterministic list of `FirmwareSchema` objects with MD5-hashed IDs.
- **New Endpoint (`/api/firmware/fs_content/{platform_id}/{file_name}`):** Streams raw files securely directly from `/romm/library/bios/{platform_slug}/{file_name}`.

### 2. Frontend (`frontend/src/views/Player/EmulatorJS/Player.vue`)
- **Global `EJS_externalFiles` Declaration:** Added support for mapping external files to EmulatorJS.
- **God-Mode Injection Hook:** On launch, the player fetches the filesystem BIOS list for the game's `platform_id` and mounts every file into `/home/web_user/retroarch/userdata/system/{file_name}`.

---

## 📁 Recommended Directory Structure

Keep your BIOS organized by the platform's official IGDB slug:
```text
/romm/library/bios/
├── amiga/
│   ├── kick34005.A500
│   └── kick40068.A1200
├── psx/
│   ├── scph5500.bin
│   ├── scph5501.bin
│   └── scph5502.bin
└── sega-cd/
    ├── bios_CD_U.bin
    └── bios_CD_E.bin
```

---

## 🛠️ How to Build & Deploy

### 1. Build the Custom Image
Run on your server inside the `ROMMULATOR` repo directory:
```bash
sudo docker build --no-cache -t romm-custom .
```

### 2. Deploy via Docker Compose
In your `docker-compose.yml`:
```yaml
services:
  romm:
    image: romm-custom
    volumes:
      - "/media/chris/4tb/4TB/Rom Library:/romm/library"
      # ... existing volumes
```

Deploy without losing any database or library data:
```bash
sudo docker compose up -d
```

---

## 🔄 Updating & Upstream Merges
When new official RomM stable versions release, update your custom fork by rebasing onto the tag:
```bash
git fetch upstream
git rebase tags/vX.Y.Z
```
Ensure `backend/endpoints/firmware.py` and `Player.vue` retain the `/fs_content/` and `EJS_externalFiles` logic.

---

# Original RomM Documentation Below

<div align="center">
  <img src=".github/resources/isotipo.png" height="180px" width="auto" alt="romm logo">
  <br />
  <img src=".github/resources/logotipo.png" height="45px" width="auto" alt="romm logotype">
  <h3 style="font-size: 25px;">A beautiful, powerful, self-hosted rom manager.</h3>
</div>

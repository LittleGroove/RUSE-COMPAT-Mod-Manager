# RUSE Compat Mod Manager

THIS REPOSITORY IS OBSOLETE, USE **[Latest release →](https://github.com/LittleGroove/RUSE-Mod-Manager/releases/latest)**


























##START

A standalone Windows application that fundamentally redefines modding for **R.U.S.E. Compat** — the community-maintained multiplayer version of R.U.S.E. Instead of distributing full replacement `.dat` files that break each other, mods are now **surgical patch files** that describe only what they change, allowing multiple mods to coexist and stack cleanly.

---

## The Problem with Traditional RUSE Mods

Classic RUSE mods work by replacing entire `.dat` archive files. If two mods touch the same `.dat`, only one can be active — they are fundamentally incompatible. Players had to pick one or the other.

## The Solution: `.rmod` Files

The Mod Manager introduces the **`.rmod`** format — a small JSON file that records only the specific changes a mod makes. Rather than saying *"replace this entire archive"*, an `.rmod` says *"find this unit and change these two properties"*. Any number of mods can patch the same file without conflict. Load order determines who wins when two mods edit the same property.

---

## Features

### Mod Manager Tab
The main interface for controlling what mods are active.

- **Load order list** — drag mods up/down to set priority. Mods higher in the list load first and can be overridden by mods below them
- **Enable / Disable individual mods** — toggle any mod on or off without removing it
- **One-click Deploy** — applies all enabled mods in order to your R.U.S.E. Compat install. Only enabled mods are applied; disabled mods are skipped entirely
- **Restore Original Files** — reverts the game back to its backed-up originals at any time
- **Setup Checklist** — two-step guided setup (Set Game Root → Create Backup) with live red/green status indicators so first-time users know exactly what to do before deploying
- **Share Load Order** — copies your current enabled mod list (names + versions, numbered) to the clipboard to send to a friend
- **Import Load Order** — paste a shared load order; the manager verifies that you have all the listed mods at the correct versions, switches matched mods on, and turns everything else off so you are in sync

### Convert Tab
Turns old-style mods (full `.dat` replacement) into `.rmod` patch files automatically.

- **Scan for Changes** — compares a modded `.dat` folder against your original game files and lists every `.dat` that was modified
- **Auto-diff** — diffs each changed NDF binary file inside each `.dat` and extracts only the properties that actually changed
- **Fill Mod Info** — name, ID, version, author, description fields. ID auto-fills from the name
- **Output to Mods Folder** — the generated `.rmod` drops straight into your mods folder and is immediately available in the Mod Manager tab
- Non-NDF file changes (textures, audio, etc.) are captured as raw file replacements inside the same `.rmod`

> **Important:** Your mod folder must mirror the game's `Data\PC\` structure. Place `.dat` files inside matching subfolders — for example `YourMod\99\ZZ_GladPatchableWin.dat` — so the converter can match them against the originals.

### Create Tab
A built-in `.rmod` editor / DAT explorer for building mods from scratch.

- **DAT Explorer** — open any `.dat` archive from your game install and browse its NDF binary files
- **Class & Instance Browser** — navigate NDF classes → instances → properties with their live values
- **Add to Patch** — select any property and click to stage it as a change. The type is pre-filled from the current data; you supply the new value
- **Load .rmod to Edit** — open an existing `.rmod` and continue editing it
- Supports all NDF types (see type reference below)

### Settings Tab
- **Game Root Directory** — the path to your R.U.S.E. Compat `Data/` folder. Setting this automatically triggers a first-time backup
- **Mods Folder** — where `.rmod` files are stored (defaults to `mods/` next to the exe)
- **Working Directory** — scratch space for converter output and backups
- **Game File Backup** — create or re-create a full backup of the original game `.dat` files. The Deploy button refuses to run without a valid backup
- All settings save automatically 600 ms after any change — no Save button needed

---

## The `.rmod` Format

An `.rmod` is a plain JSON file. You can read and edit it in any text editor.

```json
{
  "$schema": "ruse-mod/v1",
  "id":      "my-mod",
  "name":    "My Mod",
  "version": "1.0.0",
  "author":  "You",
  "description": "What this mod does",
  "patches": [
    {
      "dat": "PC/99/ZZ_GladPatchableWin.dat",
      "ndf": "genglad/patchable/clustergfx/everything.cpp.gladndfbin",
      "changes": [
        {
          "action": "patch",
          "table":  "TUniteAuSolDescriptor",
          "match":  { "ClassNameForDebug": "Unit_Stug_III_B" },
          "set": {
            "SeuilMort":      { "type": "Float32", "value": 600.0 },
            "ProductionTime": { "type": "Int32",   "value": 8 }
          }
        }
      ]
    }
  ]
}
```

### Supported Actions

| Action | Description |
|---|---|
| `patch` | Find matching instance(s) and update named properties |
| `create` | Add a new instance to a class table with given properties |
| `delete` | Remove matching instance(s) from the NDF entirely |
| `delete_props` | Remove specific properties from matching instance(s) |

### Matching Instances

The `match` block is a set of property → expected-value conditions (logical AND). All conditions must match.

| Match key | When to use |
|---|---|
| `ClassNameForDebug` | Primary stable identifier — use this whenever it exists |
| `DescriptorId` | Fallback when ClassNameForDebug is absent |
| `_index` | Positional index — last resort for tables without a debug name |

### Supported NDF Types

All NDF binary types are fully supported in both the converter and the applier:

**Scalars:** `Bool`, `Int8`, `Int16`, `UInt16`, `Int32`, `UInt32`, `Long`, `Float32`, `Float64`

**Strings:** `StringRef`, `PathRef`, `WideStr`

**Vectors & colours:** `Vector3`, `Color32`, `Color128`, `TripleInt`, `Int2`, `Float2`, `Matrix`

**References:** `ObjRef` (`{"inst": N, "class": C}`), `TransRef` (`{"trans": N}`)

**Collections:** `List<T>` (any element type, including mixed ObjRef/TransRef), `Map<K,V>`

**Other:** `Blob` (base64), `ZipBlob` (compressed blob), `Hash`, `Guid`, `LocHash`

---

## Load Order & Conflict Resolution

When two mods patch the **same property** on the **same instance**, the mod **lower** in the list wins (later overrides earlier). Everything else from both mods is applied without conflict.

The Share / Import Load Order feature lets groups of players synchronise their exact mod list and order by copying a short text block:

```
--- RUSE Load Order ---
1. Balance Overhaul Mod | v2.1.0
2. No Artillery | v1.0.0
3. Admin Buildings 1942 | v1.0.0
--- end ---
```

The importer verifies versions and warns if a mod is missing or on the wrong version.

---

## Getting Started

1. **Download** `RUSE_ModManager_vX.X.X.exe` and place it anywhere alongside a `mods/` folder
2. Open the app — go to **Settings** and set your **Game Root Directory** to your R.U.S.E. Compat `Data/` folder
3. A backup is created automatically the first time you set the game root
4. Drop `.rmod` files into the `mods/` folder (or use the **+** button in Mod Manager to copy them in)
5. Enable the mods you want, arrange the load order, and click **Deploy**
6. To revert, click **Restore Original Files**

---

## Converting Existing Mods

If you have an old mod that distributes replacement `.dat` files:

1. Go to the **Convert** tab
2. Set **Mod Folder** to the folder containing the modded `.dat` files, structured to mirror `Data\PC\` (e.g. `YourMod\99\ZZ_GladPatchableWin.dat`)
3. Click **Scan for Changes** — the converter finds every modified NDF property automatically
4. Fill in the mod name, author, and description
5. Click **Convert to .rmod** — the output drops straight into your mods folder

---

## Building a New Mod from Scratch

1. Go to the **Create** tab
2. Open `PC/99/ZZ_GladPatchableWin.dat` in the DAT Explorer (most unit/building/weapon data lives here)
3. Browse: NDF Files → class → instance → property
4. Click **Add to Patch →** for each property you want to change, fill in the new value
5. Fill in the Mod Info fields and click **Save as .rmod**

---

## Technical Notes

- Mods are applied to a working copy of the `.dat` files in the output folder, leaving backups untouched
- When multiple mods target the same `.dat`, later mods in the load order layer on top of the already-patched output — the originals are never re-read mid-chain
- All changes are logged with before/after values for every property touched
- The exe is a self-contained single file built with PyInstaller — no Python install required

# ltoctl

`ltoctl` is a Bash utility script designed to simplify LTO tape drive management and automate directory-based backups. It provides a wrapper around standard `mt` commands and implements a sequential backup strategy using `tar`.

## Features

- **Tape Management**: Easy-to-use commands for common tape operations (status, rewind, eject, seeking).
- **Automated Backup**: Iterates through a source directory and writes each item (file or folder) as a separate archive on the tape.
- **Restore Capability**: Retrieve specific archives or full backups from tape to a specified location.
- **Resume Capability**: Supports resuming a backup job from a specific file/folder if a previous run was interrupted.
- **Logging**: Automatically logs operations to a file and the console.

## Prerequisites

- Linux/Unix environment
- `mt-st` (or standard `mt`) utility installed
- `tar`
- An LTO tape drive configured (e.g., `/dev/nst0`)

## Configuration

Open the `ltoctl` script and adjust the configuration variables at the top of the file to match your environment:

```bash
TAPE_DEVICE="/dev/nst0"          # Your non-rewinding tape device
LOG_FILE_BASE="/var/log/lto_backup" # Base path for logs
BLOCK_SIZE="1024"                # Block factor for tar
```

> **Note:** It is critical to use the **non-rewinding** device node (usually `/dev/nst0` or `/dev/nst1`) so that the tape does not rewind after every command or file write.

## Usage

Make the script executable:

```bash
chmod +x ltoctl
```

Run the script with a subcommand:

```bash
./ltoctl <command> [arguments]
```

### Tape Management Commands

| Command | Argument | Description |
| :--- | :--- | :--- |
| `status` | - | Check the status of the tape drive. |
| `rewind` | - | Rewind the tape to the beginning. |
| `eject` | - | Rewind and eject the tape (alias for `rewoffl`). |
| `eof` | `[N]` | Write `N` End-of-File marks (default: 1). |
| `fsf` | `[N]` | Forward space `N` file marks (skip `N` archives). |
| `bsf` | `[N]` | Backward space `N` file marks. |

**Examples:**

```bash
./ltoctl status
./ltoctl rewind
./ltoctl fsf 2  # Skip past the next 2 archives
```

### Backup Command

The backup command iterates through the immediate children of the source directory and writes them to tape one by one.

**Syntax:**
```bash
./ltoctl backup <SOURCE_DIRECTORY> [START_FROM_ITEM]
```

**1. Full Backup:**
Back up all items in `/mnt/data` to the tape.

```bash
./ltoctl backup /mnt/data
```

**2. Resume Backup:**
If a backup failed or was stopped, you can resume it starting from a specific folder or file name. The script will skip all items alphabetically before this name.

#### Resume starting from the folder "photos_2023"
```bash
./ltoctl backup /mnt/data photos_2023
```

## Restore Command

The restore command allows you to retrieve archives from the tape. You can specify which archive index to start from, where to restore them, and how many archives to retrieve.

**Syntax:**
```bash
./ltoctl restore [START_INDEX] [DEST_PATH] [NUM_ARCHIVES]
```

- `START_INDEX`: The 0-based index of the archive on tape to start restoring. (Default: 0)
- `DEST_PATH`: The destination directory. (Default: current directory `.`)
- `NUM_ARCHIVES`: The number of sequential archives to restore. (Default: Restore until end of tape or error)

**Examples:**

**1. Restore everything to current directory:**
```bash
./ltoctl restore
```

**2. Restore the first archive (index 0) to `/tmp/restore`:**
```bash
./ltoctl restore 0 /tmp/restore 1
```

**3. Restore 5 archives starting from index 3:**
```bash
./ltoctl restore 3 /mnt/recovery 5
```

## Logging
## Logging

Logs are generated with a timestamp in the filename:
`_YYYYMMDD_HHMMSS.log`

By default, they are stored at `/var/log/lto_backup*`. Ensure the user running the script has write permissions to this directory.

## License

[MIT](LICENSE)

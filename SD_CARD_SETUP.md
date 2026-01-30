# microSD card setup (64GB) for patterns and data

This guide prepares new 64GB microSDXC cards with:
- a data partition for pattern and image files

It assumes a Linux host. If you are using another OS, use its partitioning
tool and keep the same partition layout and labels.

## Recommended layout
- Partition 1 (DATA): full card, exFAT (or FAT32 if your device requires it)

## Step 1: identify the card device
Plug in the card and find its device path:

```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,MODEL
```

Set a variable to the correct device (example uses /dev/sdX):

```bash
DEV=/dev/sdX
```

If your system exposes the card as /dev/mmcblk1, partitions will be
/dev/mmcblk1p1.

## Step 2: wipe existing signatures
This removes any old filesystem metadata.

```bash
sudo wipefs -a "$DEV"
```

## Step 3: create the partitions

```bash
sudo parted "$DEV" --script \
  mklabel gpt \
  mkpart primary 1MiB 100%
```

## Step 4: format and label
After partitioning, confirm the new partition paths and set variables:

```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT "$DEV"
PART1=/dev/sdX1
```

Use unique labels for each card (A, B) so they are easy to mount.

```bash
sudo mkfs.exfat -n OPP_PATTERNS_A "$PART1"
```

If your device or library requires FAT32, use:

```bash
sudo mkfs.fat -F 32 -n OPP_PATTERNS_A "$PART1"
```

## Step 5: mount the data partition and create folders

```bash
sudo mkdir -p /mnt/opp-patterns
sudo mount /dev/disk/by-label/OPP_PATTERNS_A /mnt/opp-patterns
mkdir -p /mnt/opp-patterns/patterns /mnt/opp-patterns/images
```

To enable the mount automatically on boot, add an entry to /etc/fstab
(adjust the mount point and filesystem type as needed):

```
/dev/disk/by-label/OPP_PATTERNS_A /mnt/opp-patterns exfat defaults,uid=1000,gid=1000,umask=022 0 2
```

## Teensy 4.1 fast-path notes
For highest throughput on a Teensy 4.1:

- Use the built-in microSD slot (SDIO, 4-bit) rather than SPI.
- Use the SdFat library with exFAT enabled for SDXC cards.
- Keep files contiguous (pre-allocate or create contiguous files in code).
- Use a shallow directory structure and fewer small files.
- If your formatter lets you choose allocation unit size, 128 KiB is a good
  starting point for large sequential reads/writes on 64GB cards.

If you are using the standard Arduino SD library, FAT32 is the safest
filesystem choice, but it is usually slower than SdFat + exFAT on SDIO.

## Repeat for the second card
Re-run the steps for the second card and use labels like
OPP_PATTERNS_B.

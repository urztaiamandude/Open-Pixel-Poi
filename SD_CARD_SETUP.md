# microSD card setup (64GB) for patterns and swap

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
/dev/mmcblk1p1 and /dev/mmcblk1p2.

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

If your device requires FAT32, use:

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

## Notes for Teensy data access
Teensy firmware reads files through an SD library, so the filesystem must
match what your firmware supports. If you are using the standard SD
library, FAT32 is typically the safest choice. If you are using SdFat or
another library that supports exFAT, exFAT is fine for 64GB cards.

## Repeat for the second card
Re-run the steps for the second card and use labels like
OPP_PATTERNS_B.

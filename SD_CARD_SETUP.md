# microSD card setup (64GB) for patterns and swap

This guide prepares new 64GB microSDXC cards with:
- a data partition for pattern and image files
- an optional Linux swap partition for extra memory headroom

It assumes a Linux host. If you are using another OS, use its partitioning
tool and keep the same partition layout and labels.

## Recommended layout
- Partition 1 (DATA): 56 GiB, exFAT (or FAT32 if your device requires it)
- Partition 2 (SWAP): remaining space, Linux swap

If you prefer a different split, adjust the sizes below.

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
  mkpart primary 1MiB 56GiB \
  mkpart primary linux-swap 56GiB 100%
```

## Step 4: format and label
After partitioning, confirm the new partition paths and set variables:

```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT "$DEV"
PART1=/dev/sdX1
PART2=/dev/sdX2
```

Use unique labels for each card (A, B) so they are easy to mount.

```bash
sudo mkfs.exfat -n OPP_PATTERNS_A "$PART1"
sudo mkswap -L OPP_SWAP_A "$PART2"
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

## Step 6: enable swap (optional)

```bash
sudo swapon /dev/disk/by-label/OPP_SWAP_A
swapon --show
```

To enable both automatically on boot, add entries to /etc/fstab
(adjust the mount point and filesystem type as needed):

```
/dev/disk/by-label/OPP_PATTERNS_A /mnt/opp-patterns exfat defaults,uid=1000,gid=1000,umask=022 0 2
/dev/disk/by-label/OPP_SWAP_A none swap sw 0 0
```

## Notes on swap wear
Swap on a microSD card is slower and adds write wear. If you only need
occasional memory relief, keep swap small (2 to 4 GiB) and set a low
swappiness value, for example:

```bash
sudo sysctl vm.swappiness=10
```

## Repeat for the second card
Re-run the steps for the second card and use labels like
OPP_PATTERNS_B and OPP_SWAP_B.

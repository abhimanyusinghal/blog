Exporting an image of a physical server's disk is a different process from virtual machines or cloud instances. Here's how you can do it:

### 1. Boot from a Live Linux Environment:

To ensure a consistent snapshot of the disk and to avoid potential data corruption, it's recommended to boot the server from a Live Linux environment, such as a Live Ubuntu USB stick.

### 2. Identify the Disk:

Once you've booted into the Live environment, open a terminal and use the `lsblk` command to identify the disk you want to image. You'll likely see it listed as `/dev/sda`, `/dev/sdb`, etc.

### 3. Create a Raw Disk Image:

Using the `dd` command, you can create a raw image of the disk. For instance, if you want to create an image of `/dev/sda`, you'd use:

```bash
sudo dd if=/dev/sda of=/path/to/output/ubuntu-18.04.img bs=4M status=progress
```

Replace `/path/to/output/` with the directory where you want the image to be saved. The `bs=4M` parameter sets the block size for the operation, and `status=progress` will show the progress as the image is being created.

**Note**: The `dd` command can be dangerous if misused, as it can overwrite data. Ensure you've specified the `if` (input file) and `of` (output file) parameters correctly.

### 4. Compress the Image (Optional):

Raw disk images can be quite large. If you're planning to transfer the image to another location, it might be helpful to compress it:

```bash
gzip /path/to/output/ubuntu-18.04.img
```

This will compress the image using the gzip compression algorithm, resulting in a file named `ubuntu-18.04.img.gz`.

### 5. Transfer the Image:

If you need to transfer the image to another machine or storage location, you can use tools like `scp`, `rsync`, or even a USB drive, depending on your needs and network configuration.

---

For a physical server, it's essential to be cautious and ensure you have adequate backups, as the imaging process can be more invasive than with virtualized environments. Always double-check the commands and paths to avoid data loss.
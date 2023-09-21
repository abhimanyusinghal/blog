If your VM is hosted in Azure, you'll need to go through a few steps to get the raw disk image. Here's a guide to help you export your Azure VM's disk:

### 1. Stop the Virtual Machine:

Before exporting the disk, it's a good practice to stop the virtual machine to ensure that no changes are being written to the disk during the export.

```bash
az vm stop --resource-group YOUR_RESOURCE_GROUP --name YOUR_VM_NAME
```

Replace `YOUR_RESOURCE_GROUP` with the name of the Azure resource group your VM resides in and `YOUR_VM_NAME` with the name of your virtual machine.

### 2. Create a Snapshot of the VM's Disk:

You'll first create a snapshot of the VM's OS disk.

```bash
az snapshot create \
  --resource-group YOUR_RESOURCE_GROUP \
  --name YOUR_SNAPSHOT_NAME \
  --source "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/YOUR_RESOURCE_GROUP/providers/Microsoft.Compute/disks/YOUR_VM_DISK_NAME"
```

Replace placeholders with the appropriate values:
- `YOUR_RESOURCE_GROUP`: The name of your Azure resource group.
- `YOUR_SNAPSHOT_NAME`: The desired name for the snapshot.
- `YOUR_SUBSCRIPTION_ID`: Your Azure subscription ID.
- `YOUR_VM_DISK_NAME`: The name of the VM's OS disk.

### 3. Export the Snapshot:

Now, create a Shared Access Signature (SAS) URL for the snapshot, which will allow you to download it.

```bash
az snapshot grant-access \
  --resource-group YOUR_RESOURCE_GROUP \
  --name YOUR_SNAPSHOT_NAME \
  --duration-in-seconds 3600
```

This command will return a JSON object with a `url` field. Copy this URL.

### 4. Download the Snapshot:

Use `curl` or `wget` to download the snapshot:

```bash
wget -O ubuntu-18.04.vhd "YOUR_SAS_URL"
```

Replace `YOUR_SAS_URL` with the URL you copied in the previous step.

### 5. Convert the VHD to a Raw Disk Image:

Azure provides the disk snapshot in the VHD format. If you need it in raw format, use the `qemu-img` tool:

```bash
qemu-img convert -f vpc -O raw ubuntu-18.04.vhd ubuntu-18.04.img
```

### 6. Clean Up:

Once you've successfully exported and converted the disk image, consider cleaning up the snapshot to avoid incurring additional charges.

```bash
az snapshot delete \
  --resource-group YOUR_RESOURCE_GROUP \
  --name YOUR_SNAPSHOT_NAME
```

---

You should now have a raw disk image (`ubuntu-18.04.img`) of your Azure VM, which you can use for the Mender conversion process.
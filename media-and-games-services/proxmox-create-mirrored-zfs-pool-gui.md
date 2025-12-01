## **🔧 Creating and Connecting a Mirrored ZFS Pool in Proxmox VE Using the GUI**

This guide walks you through the steps of setting up a ZFS pool with mirrored HDDs and connecting this enhanced storage solution to an existing VM in Proxmox VE, all through the intuitive web interface.

### **🛠️ Creating a ZFS Pool with Mirrored HDDs**

1. **🌍 Log Into Proxmox VE Web Interface**
    - Open your preferred web browser and navigate to your Proxmox VE's IP address or hostname. Log in using your credentials.

2. **🏠 Open the Datacenter View**
    - Once logged in, click on "Datacenter" in the tree on the left side to ensure you're viewing the correct area in the Proxmox web interface.

3. **💾 Navigate to Disks Management**
    - Click on the "Storage" tab in the main pane, then find and select the "Disks" option in the left sidebar.

4. **🔎 Identify Disks for ZFS Pool**
    - Within the "Disks" view, pinpoint the two HDDs you plan to use for your ZFS mirrored pool. Make a note of their device identifiers (e.g., `/dev/sda`).

5. **✨ Create ZFS Pool**
    - Navigate to the "ZFS" tab on the left sidebar and hit the "Create ZFS" button.
    - In the creation dialog, set the RAID level to "Mirror".
    - Choose the two previously identified disks for your pool.
    - Name your ZFS pool (something like `storage0`) and tweak additional options as required, though the default settings often work well.
    - Hit "Create" to establish your ZFS pool.

### **💽 Connecting ZFS Storage to an Existing VM**

1. **🖇️ Attach ZFS Volume to VM**
    - Locate your VM in the tree on the left and switch to the "Hardware" tab in the VM's pane.
    - Click "Add" > "Hard Disk".
    - From the "Storage" dropdown, choose the ZFS storage you've just set up.
    - The "Disk size" should automatically fill with your ZVOL's size; adjust if necessary, then click "Add".

### **👩‍💻 Preparing the Disk Inside the VM**

1. **🔑 Access the VM**
    - Open the VM's console by clicking on the "Console" button.

2. **📋 Partition, Format, and Mount the Disk (Linux OS)**
    - Locate the right disk using `lsblk`
    - Partition the new disk using this command:`sudo mkfs.ext4 /dev/sdb`
    - Then make a dir in your server e.g. `sudo mkdir /smb`
    - Mount the filesystem at your preferred mount point (eg, `sudo mount /dev/sdb /smb`).

4. **🔄 Make the Disk Mount Persistent (Optional)**
    - To ensure the disk mounts on reboot, edit the `/etc/fstab` file within the VM and include the new disk.

### **🎉 Conclusion**

Congratulations! You've adeptly created a mirrored ZFS pool and linked a volume from this pool to an existing VM using the Proxmox VE GUI, significantly boosting data resilience and your VM's storage options.

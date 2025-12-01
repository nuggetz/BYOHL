## **🛠️ Setting Up Samba on Ubuntu Server 22.04 LTS**

This guide will help you install Samba to share files across different operating systems seamlessly. We’ll also cover setting up a shared folder and configuring permissions for secure access.

### **📦 Installing Samba**

1. **🌐 Update Your System**
   - Before starting, ensure your system is up to date. Open a terminal and run:
     ```
     sudo apt update && sudo apt upgrade -y
     ```

2. **🔽 Install Samba**
   - Install Samba with the following command:
     ```
     sudo apt install samba -y
     ```

### **🔧 Configuring Samba**

1. **📂 Create a Shared Folder**
   - Decide where you want your shared folder to be located and create it. For example, to create a folder named `sharedfolder` in `/srv/samba/`, run:
     ```
     sudo mkdir -p /smb/data
     ```

2. **🔑 Set Folder Permissions**
   - It’s crucial to set appropriate permissions to ensure only authorized users can access the shared folder. Set the ownership and permissions as needed, for instance:
     ```
     sudo chown -R nobody:nogroup /smb/data
     sudo chmod -R 0775 /smb/data
     ```
   - This setup allows users in the `nogroup` group to read, write, and execute files in the folder, while others can only read and execute.

3. **🛠️ Configure Samba**
   - Back up the original Samba configuration file:
     ```
     sudo cp /etc/samba/smb.conf{,.backup}
     ```
   - Open the Samba configuration file in your preferred text editor:
     ```
     sudo nano /etc/samba/smb.conf
     ```
   - Scroll to the end of the file and add the configuration for your shared folder:
     ```
     [SharedFolder]
     path = /smb/data
     browseable = yes
     read only = no
     writable = yes
     guest ok = yes
     create mask = 0775
     ```
   - This configuration makes the folder accessible and writable for guests, with a create mask ensuring files have the correct permissions.

4. **🔄 Restart Samba Services**
   - Apply the changes by restarting Samba services:
     ```
     sudo systemctl restart smbd
     ```

### **🔐 Adding Samba Users**

1. **👤 Create a Samba User**
   - Samba uses a separate set of usernames and passwords from the system accounts. To access the shared folder, users must be added to Samba:
     ```
     sudo smbpasswd -a username
     ```
   - Replace `username` with the actual username of the system user you wish to grant access. You'll be prompted to set a Samba password for this user.

2. **🚪 Access the Shared Folder**
   - The shared folder is now accessible from other computers on the same network. Access it using the server’s IP address and the share name, like `\\serve-IP\SharedFolder` on Windows or `smb://server-IP/SharedFolder` on macOS.

### **🔒 Conclusion**

You've successfully installed Samba on your Ubuntu 22.04 server and set up a shared folder with the necessary permissions. Your networked devices can now easily access and collaborate using the shared resources. 🎉


# Steps to Install CRI-O:

Download CRI-O v1.30.5: Download the necessary CRI-O tar file:

bash
Copiar código
sudo wget -O /tmp/crio.tar.gz https://storage.googleapis.com/cri-o/artifacts/cri-o.amd64.v1.30.5.tar.gz
Download the Sysext Scripts: These scripts will help create a system extension for CRI-O in Flatcar:

bash
Copiar código
sudo wget https://raw.githubusercontent.com/flatcar/sysext-bakery/main/create_crio_sysext.sh -O create_crio_sysext.sh
sudo chmod +x create_crio_sysext.sh

sudo wget https://raw.githubusercontent.com/flatcar/sysext-bakery/main/bake.sh -O bake.sh
sudo chmod +x bake.sh
Create the System Extension: Run the script to create the system extension (sysext):

bash
Copiar código
sudo ./create_crio_sysext.sh 1.30.5 crio-sysext
Deploy the System Extension: Move the generated .raw system extension file to the required location:

bash
Copiar código
sudo mv crio-sysext.raw /var/lib/extensions/
sudo systemctl enable extensions-crio-sysext.slice
sudo systemctl start extensions-crio-sysext.slice
Verify Installation: Ensure that CRI-O is successfully installed:

bash
Copiar código
systemctl status crio
Install CNI Plugins: Set up the CNI plugins by creating the necessary directories and extracting the files:

bash
Copiar código
sudo mkdir -p /opt/cni/bin
sudo tar -xzf /tmp/crio.tar.gz -C /usr/local/bin/crio/
Install squashfs-tools: Since Flatcar is read-only by default, install squashfs-tools in a container and copy the mksquashfs binary to your Flatcar system:

bash
Copiar código
sudo docker run -it --rm alpine
apk add squashfs-tools
Then, copy the mksquashfs binary from the container to your system:

bash
Copiar código
sudo docker ps
sudo docker cp <container_id>:/usr/bin/mksquashfs /usr/local/bin/mksquashfs
sudo chmod +x /usr/local/bin/mksquashfs
Mount Filesystem as Read/Write (if needed): If you encounter issues with the file system being read-only, remount it:

bash
Copiar código
sudo mount -o remount,rw /
By following these steps, CRI-O should be successfully installed and ready for use on your Flatcar Container Linux. The sysext method is the recommended approach for handling CRI-O installations on this read-only OS​(
ItSufficient
)​(
GitHub
).

___

Steps to Install mksquashfs and Continue the Process:
Use Docker to Install squashfs-tools: Since Flatcar Linux is often used as a minimal, immutable system, you can utilize Docker to install the squashfs-tools and copy the mksquashfs binary to your Flatcar system.

First, run an Alpine container:

bash
Copiar código
sudo docker run -it --rm alpine
Inside the container, install squashfs-tools:

bash
Copiar código
apk add squashfs-tools
After the installation, find the mksquashfs binary location:

bash
Copiar código
find / -name mksquashfs
Exit the container, then copy the binary from the running container to the Flatcar system:

bash
Copiar código
sudo docker ps
sudo docker cp <container_id>:/usr/bin/mksquashfs /usr/local/bin/mksquashfs
Make sure the binary is executable:

bash
Copiar código
sudo chmod +x /usr/local/bin/mksquashfs
Retry the System Extension Creation: Once mksquashfs is available on your system, rerun the create_crio_sysext.sh script:

bash
Copiar código
sudo ./create_crio_sysext.sh 1.30.5 crio-sysext
Handle File System Issues: If the system remains in read-only mode and prevents copying files or making changes, remount it in read-write mode:

bash
Copiar código
sudo mount -o remount,rw /
These steps should help resolve the issue with mksquashfs and enable you to complete the CRI-O installation on Flatcar Linux. Let me know if you encounter any further problems!








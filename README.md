# 💾 Setting Up Logical Volumes with Multiple EBS Volumes on an AWS Instance

This guide walks you through attaching **three EBS volumes** to an EC2 instance, creating a Volume Group (VG), and mounting a Logical Volume (LV) to a directory. By the end of this tutorial, your new storage will be ready for use on your instance!

---

## **🛠 Prerequisites**
- An **AWS EC2 instance** (Amazon Linux 2).
- **Three EBS volumes** attached to your instance (e.g., 10GB each).
- Basic knowledge of Linux commands.

---

## **Step 1: Attach EBS Volumes to the Instance**
1. Go to the AWS Management Console → EC2 → Volumes.
2. Attach three EBS volumes to your instance.
   - Example: `/dev/xvdf`, `/dev/xvdg`, and `/dev/xvdh`.

3. SSH into the EC2 instance:
   ```bash
   ssh -i your-key.pem ec2-user@your-instance-ip
   ```

4. Verify the attached volumes:
   ```bash
   lsblk
   ```
   You should see devices like `/dev/xvdf`, `/dev/xvdg`, and `/dev/xvdh`.

---

## **Step 2: Create Physical Volumes**
Initialize the raw volumes as physical volumes for LVM:

```bash
sudo pvcreate /dev/xvdf /dev/xvdg /dev/xvdh
```

Check the status of physical volumes:
```bash
sudo pvdisplay
```

---

## **Step 3: Create a Volume Group**
Combine the physical volumes into a single Volume Group (VG):

```bash
sudo vgcreate raptors_vg /dev/xvdf /dev/xvdg /dev/xvdh
```

Verify the Volume Group:
```bash
sudo vgdisplay
```

---

## **Step 4: Create a Logical Volume**
Allocate space from the Volume Group to create a Logical Volume (LV):

```bash
sudo lvcreate -L 25G -n raptors_lv raptors_vg
```

Here:
- `-L 25G`: Specifies the size (adjust as needed).
- `-n raptors_lv`: Logical volume name.
- `raptors_vg`: Volume group name.

Verify the Logical Volume:
```bash
sudo lvdisplay
```

---

## **Step 5: Format the Logical Volume**
Create a filesystem on the Logical Volume:
```bash
sudo mkfs.ext4 /dev/raptors_vg/raptors_lv
```

---

## **Step 6: Mount the Logical Volume**
1. Create a directory to serve as the mount point:
   ```bash
   sudo mkdir -p /test/dir
   ```

2. Mount the Logical Volume:
   ```bash
   sudo mount /dev/raptors_vg/raptors_lv /test/dir
   ```

3. Verify the mount:
   ```bash
   df -hT
   ```

---

## **Step 7: Persist the Mount Across Reboots**
1. Add the Logical Volume to `/etc/fstab`:
   ```bash
   echo "/dev/raptors_vg/raptors_lv /test/dir ext4 defaults 0 0" | sudo tee -a /etc/fstab
   ```

2. Test the `/etc/fstab` configuration:
   ```bash
   sudo mount -a
   ```

---

## **Final Verification**
Check everything is working:
1. Verify the mounted filesystem:
   ```bash
   df -h
   ```
2. Confirm directory access:
   ```bash
   ls -l /test/dir
   ```

---

## **🎉 Success!**
You've successfully configured your Logical Volume using three EBS volumes. The storage is now mounted at `/test/dir` and ready for use.

---

### **🚀 Next Steps**
- Use `/test/dir` to store your data.
- Experiment with resizing your Logical Volume using LVM.

For any questions, feel free to raise an issue! 😊

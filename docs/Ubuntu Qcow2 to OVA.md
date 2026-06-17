# Overview
This guide talks about how to convert the Ubuntu qcow2 image to ova format

# Prerequisite
- The latest RHEL/CentOS/Ubuntu ppc64le machine(virtual/baremetal) with enough diskspace with root access
- Packages:
    - qemu-img
    - cloud-utils-growpart
- pvsadm tool

# Steps
## Step 1: Download the Ubuntu cloud image

Ubuntu cloud images for ppc64le architecture are available at: https://cloud-images.ubuntu.com/

### Popular Ubuntu versions:
- Ubuntu 24.04 LTS (Noble Numbat): https://cloud-images.ubuntu.com/noble/current/
- Ubuntu 22.04 LTS (Jammy Jellyfish): https://cloud-images.ubuntu.com/jammy/current/
- Ubuntu 20.04 LTS (Focal Fossa): https://cloud-images.ubuntu.com/focal/current/

Download the required cloud image for ppc64el architecture. Look for files with pattern: `ubuntu-*-server-cloudimg-ppc64el.img`

Example:
```shell
# Download Ubuntu 24.04 LTS image
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-ppc64el.img

# Or Ubuntu 22.04 LTS image
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-ppc64el.img

# Or Ubuntu 20.04 LTS image
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-ppc64el.img
```

## Step 2: Convert the qcow2/img to ova

```shell
# Convert the Ubuntu 24.04 image to ova format with installing all the prerequisites required for image to work in the IBM Power Systems Virtual Server
$ pvsadm image qcow2ova --image-name ubuntu-2404 --image-dist ubuntu --image-url ./noble-server-cloudimg-ppc64el.img

# Convert the Ubuntu 22.04 image to ova format
$ pvsadm image qcow2ova --image-name ubuntu-2204 --image-dist ubuntu --image-url ./jammy-server-cloudimg-ppc64el.img

# Convert the Ubuntu 20.04 image to ova format
$ pvsadm image qcow2ova --image-name ubuntu-2004 --image-dist ubuntu --image-url ./focal-server-cloudimg-ppc64el.img
```

## Step 3: Upload and Import to PowerVS

After conversion, follow these guides:
- [How to upload image to COS bucket](How%20to%20Upload%20Image%20to%20COS.md)
- [How to import image to PowerVS workspace](How%20to%20Import%20Image%20to%20PowerVS%20Instance.md)

# Notes

- Ubuntu cloud images come in `.img` format which is essentially a raw disk image. The tool handles this format automatically.
- The conversion process will:
  - Install cloud-init and multipath-tools
  - Configure multipath for rootfs
  - Install PowerVM required packages (powerpc-utils, librtas)
  - Set up proper GRUB configuration
  - Configure cloud-init for PowerVS environment
  - Set root password (auto-generated and saved to `password.txt`)
- Default user created by cloud-init: `ubuntu` (for Ubuntu distributions)
- You can customize the image preparation using `--prep-template` option
- You can customize cloud-init configuration using `--cloud-config` option

# Advanced Options

## Custom OS Password
```shell
# Set a custom root password
$ pvsadm image qcow2ova --image-name ubuntu-2404 --image-dist ubuntu --image-url ./noble-server-cloudimg-ppc64el.img --os-password MySecurePassword123

# Skip setting root password
$ pvsadm image qcow2ova --image-name ubuntu-2404 --image-dist ubuntu --image-url ./noble-server-cloudimg-ppc64el.img --skip-os-password
```

## Custom Image Size
```shell
# Create OVA with 50GB image size (default is 11GB)
$ pvsadm image qcow2ova --image-name ubuntu-2404 --image-dist ubuntu --image-url ./noble-server-cloudimg-ppc64el.img --image-size 50
```

## Custom Preparation Template
```shell
# Step 1: Dump the default template
$ pvsadm image qcow2ova --prep-template-default > ubuntu-prep.template

# Step 2: Modify the template as needed

# Step 3: Use the custom template
$ pvsadm image qcow2ova --image-name ubuntu-2404 --image-dist ubuntu --image-url ./noble-server-cloudimg-ppc64el.img --prep-template ubuntu-prep.template
```

## Custom Cloud Config
```shell
# Step 1: Dump the default cloud config
$ pvsadm image qcow2ova --cloud-config-default > ubuntu-cloud.config

# Step 2: Modify the cloud config as needed

# Step 3: Use the custom cloud config
$ pvsadm image qcow2ova --image-name ubuntu-2404 --image-dist ubuntu --image-url ./noble-server-cloudimg-ppc64el.img --cloud-config ubuntu-cloud.config
```

# Troubleshooting

## Image Download Issues
If you encounter issues downloading from cloud-images.ubuntu.com, try:
- Using a mirror closer to your location
- Downloading via browser and then using local file path

## Conversion Failures
- Ensure you have sufficient disk space (at least 3x the image size)
- Verify all prerequisite packages are installed
- Check system logs for detailed error messages

## PowerVS Import Issues
- Ensure the OVA file is properly uploaded to COS bucket
- Verify bucket permissions and access policies
- Check PowerVS workspace region matches COS bucket region
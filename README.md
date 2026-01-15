# pve_media_upload

Ansible role for uploading various installation media, scripts and files to Proxmox VE.

## Requirements

rsync and sshpass has to be installed on the proxmox hosts
proxmoxer, requests_toolbelt, requests python package need to be installed on the controller
community.proxmox collection version 1.3.0 or newer needs to be installed

## Role variables examples

```yaml
pve_media_upload_iso_download_path: "/var/lib/vz/template/iso"
pve_media_upload_template_download_path: "/var/lib/vz/template/cache"
pve_media_upload_snippets_folder_path: "/var/lib/vz/snippets"
```

```yaml
pve_media_upload_native:
  - url: "https://somesite.com/Rocky-10.0-x86_64-minimal.iso"
    template: "Rocky-10.0-x86_64-minimal.iso"
    checksum: "de75c2f7cc566ea964017a1e94883913f066c4ebeb1d356964e398ed76cadd12"
    checksum_algorithm: sha256
    content_type: iso
  - url: "https://somesite.com/lxc/lxc-rockylinux9.tar.xz"
    template: "lxc-rockylinux9.tar.xz"
    checksum: "157259825f481a7e21202fe4c110a8a0542dc1ca83eccfb42b52f85db10426b4"
    checksum_algorithm: sha256
```

```yaml
pve_media_upload_local_dirs:
  - "/home/user/fulldir"
```

```yaml
pve_media_upload_smb_shares:
  - mount_point: "/mnt/smb_share"
    smb_share: "//10.1.1.69/files/ISO/proxmox"
    smb_user: "hghjg"
    smb_password: "bferfbf"
```

```yaml
pve_media_upload_snippets:
  - src: redhathook.sh.j2
    dest: redhathook.sh
  - src: ubuntuhook.sh.j2
    dest: ubuntuhook.sh
  - src: almahook.sh.j2
    dest: almahook.sh
```

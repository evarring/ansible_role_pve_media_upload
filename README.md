# ansible_role_pve_media_upload

Ansible role to upload and manage installation media, LXC templates and snippet scripts on Proxmox VE hosts.

## Features

- Upload ISOs and template archives from local folders, SMB shares, or remote URLs.
- Render and install snippet scripts into a configurable snippets folder.
- Use `community.proxmox.proxmox_template` for direct Proxmox uploads when requested.

## Requirements

- Ansible 2.18 or later.
- Collections: `community.proxmox`, `ansible.posix` (for `synchronize` and `mount`).
- Controller requirements (when using `proxmox_template`): Python packages such as `proxmoxer`, `requests`, and `requests_toolbelt` installed on the controller node (the host that runs the delegated `localhost` tasks).
- `rsync` installed on controller and/or target nodes for `ansible.posix.synchronize` to work.

## Role Variables

All variables have sensible defaults in `defaults/main.yml`. Important ones:

- `pve_media_upload_iso_download_path` (default: `/var/lib/vz/template/iso`) — destination for ISO uploads on the Proxmox node.
- `pve_media_upload_template_download_path` (default: `/var/lib/vz/template/cache`) — destination for LXC/template uploads.
- `pve_media_upload_snippets_folder_path` (default: `/var/lib/vz/snippets`) — where rendered snippet scripts are written.

Upload sources (one or more of the following can be defined):

- `pve_media_upload_native`: list of maps describing remote URLs or controller-local `src` files to upload via `community.proxmox.proxmox_template`. Each item may include `url`, `src`, `template`, `checksum`, `checksum_algorithm`, `content_type` and `storage`.
- `pve_media_upload_local_dirs`: list of local controller directories to `synchronize` (push) content from. The role filters by filename extensions.
- `pve_media_upload_smb_shares`: list of SMB share maps with `mount_point`, `smb_share`, `smb_user`, `smb_password` to mount, pull content, then unmount.
- `pve_media_upload_snippets`: list of `{ src: template.j2, dest: output-file }` items to be rendered into the snippets folder.

See `defaults/main.yml` for commented examples and additional options.

## Example Playbooks

Upload a remote ISO/template via the Proxmox API (controller delegates to `localhost`):

```yaml
- hosts: proxmox_nodes
  gather_facts: false
  vars:
    pve_media_upload_native:
      - url: "https://download.example.org/AlmaLinux-10.0-x86_64-minimal.iso"
        template: "AlmaLinux-10.0-x86_64-minimal.iso"
        checksum: "<sha256sum>"
        checksum_algorithm: sha256
        content_type: iso
        storage: local
  roles:
    - role: ansible_role_pve_media_upload
```

Synchronize ISOs from a local controller directory to all Proxmox nodes:

```yaml
- hosts: proxmox_nodes
  vars:
    pve_media_upload_local_dirs:
      - "/srv/isos"
  roles:
    - role: ansible_role_pve_media_upload
```

Pull files from an SMB share and upload the ISOs:

```yaml
- hosts: proxmox_nodes
  vars:
    pve_media_upload_smb_shares:
      - mount_point: "/tmp/mnt_smb"
        smb_share: "//10.0.0.5/ISO/proxmox"
        smb_user: "myuser"
        smb_password: "{{ vault_proxmox_smb_password }}"
  roles:
    - role: ansible_role_pve_media_upload
```

Render snippets into the proxmox snippets folder:

```yaml
  vars:
    pve_media_upload_snippets:
      - src: redhathook.sh.j2
        dest: redhathook.sh
  roles:
    - role: ansible_role_pve_media_upload
```

## Dependencies

None required by the role itself, but the role expects the `community.proxmox` collection and `ansible.posix` collection to be available when using remote upload or synchronization features.

## Example inventory

Use an inventory group like `proxmox_nodes` with connection variables appropriate for your environment. When `proxmox_template` is used the upload tasks are delegated to `localhost` and require API credentials (see `community.proxmox` docs).

## Security and secrets

- Store SMB credentials, Proxmox API credentials and any sensitive values in Ansible Vault or your secret management solution.
- The role uses `no_log` for tasks that mount SMB shares and synchronize those shares; nevertheless, avoid putting plaintext passwords in inventories.

## Testing

- The `tests/` folder contains a minimal `test.yml` for quick smoke runs against `localhost`.
- For full verification run the role against a test Proxmox environment and verify that files appear under configured download paths and that `community.proxmox.proxmox_template` uploads succeed.

## Troubleshooting

- If `synchronize` fails, ensure `rsync` is installed on the controller and (for pull) on the remote host.
- If `proxmox_template` uploads fail, verify the Python dependencies on the controller and that the API credentials work.
- SMB mounts require `cifs-utils` on the Proxmox node.

## License

MIT

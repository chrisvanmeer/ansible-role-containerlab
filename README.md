# Ansible Role: containerlab

This Ansible role installs [containerlab](https://github.com/srl-labs/containerlab) from a GitHub release.

It supports:

- Debian and RedHat OS families
- Installing the **latest release** or a **specific version**
- Two installation methods:
  1. **Direct from host via apt URL**
  2. **Download on controller, copy to host, install via apt**
- Optional **Docker installation**
- Optional **sudo-less** permissions
- Optional **SELinux** fix
- Support for **proxy environments** (for both host and controller)
- Fully **idempotent**: re-running the role will not reinstall if the correct version is already present

## Requirements

- Supported OS:
  - Debian (tested on **bookworm** and **trixie**)
  - RedHat Enterprise Linux / AlmaLinux / Rocky Linux (**9** and **10**)
- Ansible ≥ 2.13

## Role Variables

| Variable                               | Default               | Description                                                                                                                                                |
| -------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `containerlab_version`                 | `"latest"`            | Version to install. Use `"latest"` or a specific tag like `"v0.69.3"`.                                                                                     |
| `containerlab_install_docker`          | `false`               | Install Docker along with containerlab                                                                                                                     |
| `containerlab_docker_users`            | `[]`                  | List of users to add to the `docker` group. If empty, no users are modified.                                                                               |
| `containerlab_install_method`          | `"url"`               | Installation method. Options: `url` (host installs directly) or `controller_copy` (controller downloads `.deb` and copies to host).                        |
| `containerlab_cleanup`                 | `false`               | Whether to remove temporary `.deb`/`.rpm` files after install. If `true`, files are always re-downloaded; if `false`, files remain cached for idempotency. |
| `containerlab_github_owner`            | `"srl-labs"`          | GitHub owner/org of the repository                                                                                                                         |
| `containerlab_validate_certs`          | `true`                | Validate TLS certificates during downloads                                                                                                                 |
| `containerlab_proxy_env_host`          | `{}`                  | Proxy environment variables for the **host** when using `url`                                                                                              |
| `containerlab_proxy_env_controller`    | `{}`                  | Proxy environment variables for the **controller** when using `controller_copy`                                                                            |
| `containerlab_controller_download_dir` | `"/tmp/containerlab"` | Temporary download path on the controller for `.deb`                                                                                                       |
| `containerlab_controller_become`       | `false`               | Whether to use elevated privileges (sudo) when downloading, copying, or cleaning up containerlab packages on the Ansible controller (localhost).           |
| `containerlab_host_pkg_path`           | `"/tmp"`              | Temporary path on the target host for `.deb`                                                                                                               |
| `containerlab_github_token`            | `""`                  | GitHub token (optional, avoids API rate limits when resolving latest release)                                                                              |
| `containerlab_enable_sudo_less`        | `false`               | Enable sudo-less operation for containerlab by setting `SUID` and group membership.                                                                        |
| `containerlab_sudo_group`              | `clab_admins`         | Name of the Unix group used for sudo-less containerlab operation.                                                                                          |
| `containerlab_sudo_users`              | `[]`                  | List of users to add to the sudo-less group. Defaults to the `ansible_user_id` if empty.                                                                   |

## Example Playbooks

### Install latest containerlab release (direct from host) with Docker and sudoless permissions

```yaml
- name: Ensure Containerlab
  hosts: labnodes
  become: true

  roles:
    - role: chrisvanmeer.containerlab
      vars:
        containerlab_install_docker: true
        containerlab_docker_users:
          - chris
        containerlab_install_method: "url"
        containerlab_version: "latest"
        containerlab_enable_sudo_less: true
        containerlab_sudo_users:
          - chris
```

## Install a specific version

```yaml
- name: Ensure Containerlab
  hosts: labnodes
  become: true

  roles:
    - role: chrisvanmeer.containerlab
      vars:
        containerlab_install_method: "url"
        containerlab_version: "v0.69.2"
```

## Install via controller (useful behind firewalls) and cleanup temp files

```yaml
- name: Ensure Containerlab
  hosts: labnodes
  become: true

  roles:
    - role: chrisvanmeer.containerlab
      vars:
        containerlab_install_method: "controller_copy"
        containerlab_cleanup: true
        containerlab_proxy_env_controller:
          http_proxy: "http://proxy.example:3128"
          https_proxy: "http://proxy.example:3128"
          no_proxy: ".example.com,localhost,127.0.0.1"
```

## Idempotency

- The role dynamically builds the package filename and download URL, ensuring no trailing spaces or newline characters.
- The installed version is checked using `dpkg-query` (Debian) or `rpm -q` (RedHat).
- If the desired version is already installed, no action is taken.
- If a different version or no installation is found, the appropriate package (`.deb` or `.rpm`) is installed or upgraded.
- The package is first downloaded on the controller (with optional elevated rights) and then copied to the host.
- After installation, the package file can optionally be cleaned up on both the host and the controller.

## License

MIT

## Author Information

- Chris van Meer <chris@atcomputing.nl>

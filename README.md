# Ansible Role: SCM


Software Code Management role. Currently installs and manages [Gitea](https://about.gitea.com/) on Debian and Ubuntu systems. The role downloads a versioned upstream binary, keeps a previous version for quick rollback via a symlink, creates a dedicated system user, writes `app.ini` fully from variables, and manages the systemd unit.

Requirements
------------

This role requires Ansible 2.12 or higher. The target system should be Debian or Ubuntu.

Role Variables
--------------

The following variables are defined in `defaults/main.yml`:

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `gitea_name` | Service name (used for binary, unit, paths) | `gitea` |
| `gitea_version` | Gitea version to install (no leading `v`) | `1.25.5` |
| `gitea_arch` | Architecture suffix of the upstream release | `amd64` |
| `gitea_opt` | Install directory (holds versioned binaries + symlink) | `/opt/{{ gitea_name }}` |
| `gitea_etc` | Config directory (`app.ini` lives here) | `/etc/{{ gitea_name }}` |
| `gitea_home` | Data directory / `WorkingDirectory` | `/var/lib/{{ gitea_name }}` |
| `gitea_url` | Full download URL of the `linux-<arch>` binary | upstream GitHub release URL |
| `gitea_keep_versions` | Previous versioned binaries to keep for rollback | `1` |
| `gitea_user_create` | Whether this role should create the system user/group | `true` |
| `gitea_user` / `gitea_group` | Service user and group | `git` / `git` |
| `gitea_uid` / `gitea_gid` | Optional fixed uid/gid | unset (system-assigned) |
| `gitea_user_home` | Home directory for the service user | `/home/{{ gitea_user }}` |
| `gitea_user_shell` | Login shell for the service user | `/bin/bash` |
| `gitea_app_ini` | Dict rendered verbatim into `app.ini` | minimal sqlite3 defaults |

### About `gitea_app_ini`

`app.ini` is fully driven from this dictionary. Keys become INI sections; the reserved key `DEFAULT` is rendered at the top of the file **without** a section header (matching Gitea's convention). Section names with dots (e.g. `cron.update_checker`, `repository.signing`) are preserved verbatim. Override this dict in your playbook to inject any setting Gitea supports.

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: gitea_servers
  roles:
    - role: ansible_role_scm
      vars:
        gitea_version: '1.25.5'
        gitea_user: 'git'
        gitea_app_ini:
          DEFAULT:
            APP_NAME: 'My Gitea'
            RUN_USER: 'git'
            WORK_PATH: '/var/lib/gitea'
            RUN_MODE: 'prod'
          server:
            DOMAIN: 'git.example.com'
            HTTP_PORT: 3000
            ROOT_URL: 'https://git.example.com/'
            DISABLE_SSH: true
          database:
            DB_TYPE: 'postgres'
            HOST: '127.0.0.1:5432'
            NAME: 'giteadb'
            USER: 'gitea'
            PASSWD: '{{ vault_gitea_db_password }}'
            SSL_MODE: 'disable'
          security:
            INSTALL_LOCK: true
            INTERNAL_TOKEN: '{{ vault_gitea_internal_token }}'
```

License
-------

GPL-3.0-only

Author Information
------------------

+ Luciano Giacchetta
+ Giacchetta Networks LLC
+ https://gianet.us/engineering/ansible_role_scm

# ludus_gitlab_ce

An Ansible role for [Ludus](https://docs.ludus.cloud) that installs and configures GitLab CE on an Ubuntu target VM, creates groups, and initializes projects with source code from the Ludus host.

## Role Variables

| Variable | Required | Description |
|---|---|---|
| `gitlab_url` | Yes | Full URL GitLab will be reachable at (e.g. `http://gitlab.dev.sbg.corp`) |
| `gitlab_root_password` | Yes | Password for the root GitLab admin account |
| `gitlab_groups` | No | List of group names to create |
| `gitlab_projects` | No | List of project definitions (see below) |

### `gitlab_projects` Structure

```yaml
gitlab_projects:
  - name: my-project
    visibility: private      # 'private' or 'public'
    source_path: /var/lib/ludus/web-apps/my-project  # Absolute path on Ludus host
```

## Important Notes

- `source_path` must be an **absolute path** accessible by the Ludus daemon. Paths under `/tmp` will **not** work due to `PrivateTmp=yes` in the Ludus systemd unit — use `/var/lib/ludus/` or another persistent directory.
- The role uses `git branch -M main` to ensure the default branch is `main` regardless of the system Git version.
- GitLab is configured at the URL specified in `gitlab_url` and must be resolvable via DNS rewrite from the range.

## Example Configuration (ludus.yml)

```yaml
- vm_name: '{{ range_id }}-GITLAB-DEV'
  hostname: GITLAB-DEV
  template: ubuntu-24.04-x64-server-template
  vlan: 20
  ip_last_octet: 15
  ram_gb: 4
  cpus: 2
  dns_rewrites:
    - gitlab.test
  linux: true
  roles:
    - ludus_gitlab_ce
  role_vars:
    gitlab_url: https://gitlab.test
    gitlab_root_password: password!
    gitlab_groups:
      - dev
    gitlab_projects:
      - name: my-project
        visibility: private
        source_path: /var/lib/ludus/web-apps/my-project
```

## License

BSD 2-Clause

## Author

WhisperGate

# Ansible Role: chronyd

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-chronyd)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-chronyd)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-chronyd)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-chronyd/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-chronyd/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-chronyd/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-chronyd/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for managing time synchronization with chronyd.

## Purpose

Install chrony, validate and manage its configuration, and enable and start the
time service. The default configuration synchronizes from NTP pools without
serving NTP or exposing the UDP command port.

## Scope

### Managed

- Chrony packages, configuration, service enablement, and running state.
- NTP sources, server bindings, access rules, RTC settings, and rate limits.
- Native daemon start options, including IPv4-only and IPv6-only operation.
- Removal of conflicting ntp packages and masking of platform-specific time
  services.

### Not Managed

- Firewall rules, authentication keys, and Samba socket permissions.
- Creation and permissions of custom drift-file and measurement-history
  directories.

## Requirements

- Gather Ansible facts before applying the role and run with root privileges.
- Upstream NTP sources must be reachable for actual clock synchronization.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.posix
    version: '>=2.0.0'
```

## Role Variables

### `chronyd_options_mode`

Type: `str`. Required: `false`.

Policy for combining package defaults with chronyd_options.

Default:

```yaml
chronyd_options_mode: append
```

### `chronyd_options`

Type: `list`. Required: `false`.

Additional or replacement chronyd command-line argument tokens.
Use one whitespace-free token per entry; IPv4-only (-4) and IPv6-only (-6) are
mutually exclusive.

Default:

```yaml
chronyd_options: []
```

### `chronyd_directives`

Type: `list`. Required: `false`.

Ordered access directives; explicit deny entries below are applied afterwards.

Default:

```yaml
chronyd_directives: []
```

### `chronyd_port`

Type: `int`. Required: `false`.

NTP server port; zero disables serving NTP requests.

Default:

```yaml
chronyd_port: 0
```

### `chronyd_bindaddress`

Type: `str`. Required: `false`.

Local address for NTP requests; an empty string uses the daemon default.

Default:

```yaml
chronyd_bindaddress: ''
```

### `chronyd_bindcmdaddress`

Type: `str`. Required: `false`.

Local address for the optional chronyc UDP command socket.

Default:

```yaml
chronyd_bindcmdaddress: 127.0.0.1
```

### `chronyd_cmdport`

Type: `int`. Required: `false`.

Command port; zero restricts chronyc access to the local Unix socket.

Default:

```yaml
chronyd_cmdport: 0
```

### `chronyd_deny`

Type: `list`. Required: `false`.

Subnets denied NTP access after processing the ordered directives.

Default:

```yaml
chronyd_deny: []
```

### `chronyd_driftfile`

Type: `str`. Required: `false`.

Drift file path; an empty string disables saving the estimated clock drift.

Default:

```yaml
chronyd_driftfile: /var/lib/chrony/chrony.drift
```

### `chronyd_dumponexit`

Type: `bool`. Required: `false`.

Save measurement histories when the daemon exits.

Default:

```yaml
chronyd_dumponexit: true
```

### `chronyd_dumpdir`

Type: `str`. Required: `false`.

Measurement history directory; an empty string omits the dumpdir directive.

Default:

```yaml
chronyd_dumpdir: /var/lib/chrony
```

### `chronyd_hwtimestamp_interfaces`

Type: `list`. Required: `false`.

Network interfaces supporting hardware timestamping.

Default:

```yaml
chronyd_hwtimestamp_interfaces: []
```

### `chronyd_leapsectz`

Type: `str`. Required: `false`.

Timezone containing leap-second information; an empty string omits the
directive.

Default:

```yaml
chronyd_leapsectz: right/UTC
```

### `chronyd_makestep_secs`

Type: `float`. Required: `false`.

Clock offset threshold in seconds above which an initial update may step time.

Default:

```yaml
chronyd_makestep_secs: 1.0
```

### `chronyd_makestep_nums`

Type: `int`. Required: `false`.

Number of initial updates allowed to step time; negative values remove the
limit.

Default:

```yaml
chronyd_makestep_nums: 3
```

### `chronyd_ntp_pools`

Type: `list`. Required: `false`.

NTP pools with a required address and list of chrony source options.

Default:

```yaml
chronyd_ntp_pools:
  - address: 0.de.pool.ntp.org
    options:
      - iburst
  - address: 1.de.pool.ntp.org
    options:
      - iburst
  - address: 2.de.pool.ntp.org
    options:
      - iburst
  - address: 3.de.pool.ntp.org
    options:
      - iburst
```

### `chronyd_ntp_servers`

Type: `list`. Required: `false`.

Individual NTP servers with a required address and list of source options.

Default:

```yaml
chronyd_ntp_servers: []
```

### `chronyd_ntpsigndsocket`

Type: `str`. Required: `false`.

Samba NTP signing socket directory; an empty string disables integration.

Default:

```yaml
chronyd_ntpsigndsocket: ''
```

### `chronyd_ratelimit`

Type: `str`. Required: `false`.

NTP response rate-limit options; an empty string omits the directive.

Default:

```yaml
chronyd_ratelimit: ''
```

### `chronyd_rtconutc`

Type: `bool`. Required: `false`.

Interpret the hardware real-time clock as UTC.

Default:

```yaml
chronyd_rtconutc: true
```

### `chronyd_rtcsync`

Type: `bool`. Required: `false`.

Enable periodic kernel synchronization of the hardware real-time clock.

Default:

```yaml
chronyd_rtcsync: true
```

## Managed Files

- `/etc/chrony/chrony.conf on Debian-family systems.`
- `/etc/chrony.conf on Red Hat and SUSE systems.`
- `OPTIONS in /etc/sysconfig/chronyd on Red Hat and SUSE systems.`
- `DAEMON_OPTS in /etc/default/chrony on Debian-family systems.`

## Check Mode

Supports check mode on provisioned hosts through native module behavior.

- A first installation in check mode cannot validate configuration before
  chronyd is installed.

## Service Behavior

Every run enables and starts the service. Configuration changes notify a
restart, applied before the role returns. Repeated runs with unchanged inputs
and service state are idempotent.

### Handlers

- Restart the chrony service after configuration or start options change.

## Security Notes

- NTP serving and UDP command access are disabled by default.
- Configuration is owned by root with mode 0644 and uses module-provided
  backups.
- Explicit chronyd_deny entries are rendered after chronyd_directives; directive
  order is preserved.

## Operational Notes

- Rename legacy `chrony_*` inventory variables to `chronyd_*`; the role is now
  jomrr.chronyd.
- Pool and server entries require address and options; use an empty options list
  when no options are needed.
- Empty optional strings omit their corresponding configuration directives.
- chronyd_options_mode defaults to append, combining known package defaults with
  chronyd_options. The package base is -F 2 on Red Hat, -F 1 on Debian and
  Ubuntu, and empty on SUSE.
- Use chronyd_options_mode replace to replace the native options variable
  completely; an empty list clears it. An empty list with append restores the
  package defaults. Fixed service arguments and distribution wrappers remain in
  effect.
- The role owns the complete options assignment and replaces existing local
  changes. Each chronyd_options entry is one whitespace-free argument token; use
  ["-F", "1", "-6"] for options with values. Shell quoting inside tokens is not
  supported.
- IPv4-only (-4) and IPv6-only (-6) are mutually exclusive and require time
  sources reachable over the selected family.
- Molecule runs chronyd with -x through a test-only service override and does
  not adjust the host clock.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### NTP client

Install the default client configuration.

```yaml
- name: Configure time synchronization
  hosts: all
  become: true
  gather_facts: true
  roles:
    - role: jomrr.chronyd
```

### IPv4-only client

Append -4 while retaining the distribution package defaults.

```yaml
- name: Configure IPv4 time synchronization
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.chronyd
      chronyd_options: ["-4"]
```

### Replace start options

Replace the package options with an explicit IPv6-only configuration.

```yaml
- name: Configure IPv6 time synchronization
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.chronyd
      chronyd_options_mode: replace
      chronyd_options: ["-F", "1", "-6"]
```

### NTP server

Serve one network while excluding a subnet.

```yaml
- name: Configure an NTP server
  hosts: all
  become: true
  gather_facts: true
  roles:
    - role: jomrr.chronyd
      chronyd_port: 123
      chronyd_bindaddress: 192.0.2.1
      chronyd_directives:
        - allow 192.0.2.0/24
      chronyd_deny:
        - 192.0.2.128/25
```

## References

- [chrony Documentation](https://chrony-project.org/documentation.html)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2020 Jonas Mauer.

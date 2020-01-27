# Ansible role for the BIRD Internet Routing Daemon

This role installs and configures
[the BIRD Internet Routing Daemon](https://bird.network.cz) on an APT-based system.

## Example Playbook

The following is an example playbook which installs BIRD and configures `bird.service`
(resp. `bird6.service`) via the `bird_ipv4_config` (resp. `bird_ipv6_config`) role variable.

```yml
- hosts: router01
  become: true
  roles:
    - role: bird
      bird_ipv4_config: |
        # Content of /etc/bird/bird.conf goes here
        #
        # ...
      bird_ipv4_config: |
        # Content of /etc/bird/bird6.conf goes here
        #
        # ...
```

Hereby you have to make yourself familiar with
[how to configure BIRD](https://bird.network.cz/?get_doc&v=16&f=bird-3.html).

## Services

If `bird_ipv4_config` is set to `[]`, then `bird.service` is disabled and stopped.
Otherwise `bird.service` is enabled and started.

The same holds for `bird_ipv6_config` and `bird6.service`, respectively.

## Configuration language

In the above [example](#example-playbook) the `bird_ipv4_config` and `bird_ipv6_config` role
variables were set to strings.
With this role you can alternatively use YAML so as to write your BIRD config.
For example, the following two settings are equivalent.

```yml
bird_ipv4_config: |
  log syslog all;
  debug protocols all;
  router id 192.168.100.3;
  protocol device {
      scan time 10;
  };
  protocol kernel {
      scan time 10;
      import none;
      export all;
  };
  protocol ospf {
      export all;
      area 0 {
          stubnet 0.0.0.0/0;
          interface "eth0" {
              type broadcast;
              ttl security on;
          };
          interface "eth1" {
              stub;
          };
      };
  };
```

```yml
bird_ipv4_config:
  - log syslog all
  - debug protocols all
  - router id 192.168.100.3
  - protocol device:
      - scan time 10
  - protocol kernel:
      - scan time 10
      - import none
      - export all
  - protocol ospf:
      - export all
      - area 0:
          - stubnet 0.0.0.0/0
          - interface "eth0":
              - type broadcast
              - ttl security on
          - interface "eth1":
              - stub
```

The general policy is as follows:

* A `string` is written into the config as is.
* On a `list` the policy is applied recursively on the elements and a semicolon is added after each
  element.
* On a `dict` each key-value pair turns into `key: { value }` and the policy is applied recursively
  on the values.

## Role variable defaults

| Name               | Default | Description                                        |
| :----------------- | :------ | :------------------------------------------------- |
| `bird_ipv4_config` | `[]`    | [#configuration-language](#configuration-language) |
| `bird_ipv6_config` | `[]`    | [#configuration-language](#configuration-language) |

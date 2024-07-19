# Ansible role [haproxy](https://galaxy.ansible.com/ui/standalone/roles/buluma/haproxy/documentation)

Install and configure haproxy on your system.

|GitHub|Version|Issues|Pull Requests|Downloads|
|------|-------|------|-------------|---------|
|[![github](https://github.com/buluma/ansible-role-haproxy/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-haproxy/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-haproxy.svg)](https://github.com/buluma/ansible-role-haproxy/releases/)|[![Issues](https://img.shields.io/github/issues/buluma/ansible-role-haproxy.svg)](https://github.com/buluma/ansible-role-haproxy/issues/)|[![PullRequests](https://img.shields.io/github/issues-pr-closed-raw/buluma/ansible-role-haproxy.svg)](https://github.com/buluma/ansible-role-haproxy/pulls/)|[![Ansible Role](https://img.shields.io/ansible/role/d/buluma/haproxy)](https://galaxy.ansible.com/ui/standalone/roles/buluma/haproxy/documentation)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/buluma/ansible-role-haproxy/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true

  roles:
    - role: buluma.haproxy
      haproxy_frontends:
        - name: http
          address: "*"
          port: 80
          default_backend: backend
        - name: https
          address: "*"
          port: 443
          default_backend: backend
          ssl: true
          crts:
            - /tmp/haproxy.keycrt
        - name: smtp
          address: "*"
          port: 25
          default_backend: smtp
          mode: tcp
      haproxy_backend_default_balance: roundrobin
      haproxy_backends:
        - name: backend
          httpcheck: true
          # You can tell how the health check must be done.
          # This requires haproxy version 2
          # http_check:
          #   send:
          #     method: GET
          #     uri: /health.html
          #   expect: status 200
          balance: roundrobin
          # You can refer to hosts in an Ansible group.
          # The `ansible_default_ipv4` will be used as an address to connect to.
          servers: "{{ groups['all'] }}"
          port: 8080
          options:
            - check
        - name: smtp
          balance: leastconn
          mode: tcp
          # You can also refer to a list of servers.
          servers:
            - name: first
              address: "127.0.0.1"
              port: 25
            - name: second
              address: "127.0.0.2"
              port: 25
          port: 25
        - name: vault
          mode: tcp
          httpcheck: GET /v1/sys/health HTTP/1.1
          servers: "{{ groups['all'] }}"
          http_send_name_header: Host
          port: 8200
          options:
            - check
            - check-ssl
            - ssl verify none

      haproxy_listen_default_balance: roundrobin
      haproxy_listens:
        - name: listen
          address: "*"
          httpcheck: true
          listen_port: 8081
          balance: roundrobin
          # You can refer to hosts in an Ansible group.
          # The `ansible_default_ipv4` will be used as an address to connect to.
          servers: "{{ groups['all'] }}"
          port: 8080
          options:
            - maxconn 100000
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/buluma/ansible-role-haproxy/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: false

  roles:
    - role: buluma.bootstrap
    - role: buluma.core_dependencies
    - role: buluma.epel
    - role: buluma.buildtools
    - role: buluma.python_pip
    - role: buluma.openssl
      openssl_key_directory: /tmp
      openssl_items:
        - name: haproxy
          common_name: "{{ ansible_fqdn }}"
    # This role is applied to serve as a mock "backend" server. See `molecule/default/verify.yml`.
    - role: buluma.httpd
      httpd_port: 8080

  vars:
    ansible_python_interpreter: /usr/bin/python3
    _httpd_data_directory:
      default: /var/www/html
      Alpine: /var/www/localhost/htdocs
      Suse: /srv/www/htdocs

    httpd_data_directory: "{{ _httpd_data_directory[ansible_os_family] | default(_httpd_data_directory['default'] ) }}"
  post_tasks:
    - name: Place health check
      ansible.builtin.copy:
        content: 'ok'
        dest: "{{ httpd_data_directory }}/health.html"

    - name: Place sample page
      ansible.builtin.copy:
        content: 'Hello world!'
        dest: "{{ httpd_data_directory }}/index.html"
```

Also see a [full explanation and example](https://buluma.github.io/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/buluma/ansible-role-haproxy/blob/master/defaults/main.yml):

```yaml
---
# defaults file for haproxy

# Configure stats in HAProxy?
haproxy_stats: true
haproxy_stats_port: 1936
haproxy_stats_bind_addr: "0.0.0.0"

# Default setttings for HAProxy.
haproxy_retries: 3
haproxy_timeout_http_request: 10s
haproxy_timeout_connect: 10s
haproxy_timeout_client: 1m
haproxy_timeout_server: 1m
haproxy_timeout_http_keep_alive: 10s
haproxy_timeout_check: 10s
haproxy_maxconn: 3000

# A list of frontends. See `molecule/
haproxy_frontends: []
haproxy_backend_default_balance: roundrobin
haproxy_backends: []

# For the listening lists:
haproxy_listen_default_balance: roundrobin
haproxy_listens: []
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/buluma/ansible-role-haproxy/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | Version |
|-------------|--------|--------|
|[buluma.bootstrap](https://galaxy.ansible.com/buluma/bootstrap)|[![Ansible Molecule](https://github.com/buluma/ansible-role-bootstrap/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-bootstrap/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-bootstrap.svg)](https://github.com/shadowwalker/ansible-role-bootstrap)|
|[buluma.buildtools](https://galaxy.ansible.com/buluma/buildtools)|[![Ansible Molecule](https://github.com/buluma/ansible-role-buildtools/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-buildtools/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-buildtools.svg)](https://github.com/shadowwalker/ansible-role-buildtools)|
|[buluma.core_dependencies](https://galaxy.ansible.com/buluma/core_dependencies)|[![Ansible Molecule](https://github.com/buluma/ansible-role-core_dependencies/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-core_dependencies/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-core_dependencies.svg)](https://github.com/shadowwalker/ansible-role-core_dependencies)|
|[buluma.epel](https://galaxy.ansible.com/buluma/epel)|[![Ansible Molecule](https://github.com/buluma/ansible-role-epel/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-epel/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-epel.svg)](https://github.com/shadowwalker/ansible-role-epel)|
|[buluma.httpd](https://galaxy.ansible.com/buluma/httpd)|[![Ansible Molecule](https://github.com/buluma/ansible-role-httpd/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-httpd/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-httpd.svg)](https://github.com/shadowwalker/ansible-role-httpd)|
|[buluma.openssl](https://galaxy.ansible.com/buluma/openssl)|[![Ansible Molecule](https://github.com/buluma/ansible-role-openssl/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-openssl/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-openssl.svg)](https://github.com/shadowwalker/ansible-role-openssl)|
|[buluma.python_pip](https://galaxy.ansible.com/buluma/python_pip)|[![Ansible Molecule](https://github.com/buluma/ansible-role-python_pip/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-python_pip/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-python_pip.svg)](https://github.com/shadowwalker/ansible-role-python_pip)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://buluma.github.io/) for further information.

Here is an overview of related roles:

![dependencies](https://raw.githubusercontent.com/buluma/ansible-role-haproxy/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/buluma):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/r/buluma/enterpriselinux)|8, 9|
|[Debian](https://hub.docker.com/r/buluma/debian)|all|
|[Fedora](https://hub.docker.com/r/buluma/fedora)|38, 39, 40|
|[opensuse](https://hub.docker.com/r/buluma/opensuse)|all|
|[Ubuntu](https://hub.docker.com/r/buluma/ubuntu)|jammy, focal, bionic, noble|

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/buluma/ansible-role-haproxy/issues)

## [Changelog](#changelog)

[Role History](https://github.com/buluma/ansible-role-haproxy/blob/master/CHANGELOG.md)

## [License](#license)

[Apache-2.0](https://github.com/buluma/ansible-role-haproxy/blob/master/LICENSE)

## [Author Information](#author-information)

[Shadow Walker](https://buluma.github.io/)

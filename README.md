Ansible Role: Cerbot DNS (for Let's Encrypt)  
=========
[![CI](https://github.com/brunobenchimol/ansible-role-certbot-dns/actions/workflows/ci.yml/badge.svg)](https://github.com/brunobenchimol/ansible-role-certbot-dns/actions/workflows/ci.yml)    

Installs and configures Certbot (for Let's Encrypt) using DNS challenge using [DNS Plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins).  

Requirements
------------

This role uses `geerlingguy.certbot` role so it inherits all recommendations and pre-requisites. Some recommendations follows:
* Only use `source` installation from git if you're using an older OS release.  
* Preferred method of installation is `snap` . Otherwise use `package` method.  
* If you're using RedHat/CentOS you will require EPEL. Its recommended to use `geerlingguy.repo-epel` role before.

Role Variables
--------------

The following table shows every new variable created that differs from `geerlingguy.certbot`:  

|    Variable      |     Description     |
| ---------------- | ------------------- |
| `certbot_delete_certificate`      |  (true/`false`) Enable/Disable Certificate delete (using certbot delete). | 
| `certbot_auto_renew`              |  If you set to `false` it will remove the cronjob if it was previously installed.  |
| `certbot_create_reload_services`  | List of services to reload after each successfully issued certificate.  |
| `certbot_dns_plugin`              | Certbot [DNS Plugin](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins) to use. There is no default. Mandatory. |
| `certbot_dns_credentials_custom_file` | DNS Credentials File Path. Useful when using unsupported plugin by this role. |
| `certbot_dns_*`                    | Options to choose depending on each plugin, refer to DNS Plugins Variables below. |


### DNS Plugins

Currently we support built-in methods for **certbot_dns_plugins**: `rfc2136`, `luadns`, `cloudflare` and `digitalocean`. Other plugins may be added in the future.     

**Plugin**: `digitalocean`, `cloudflare` and `luadns`   
|  Variable | Description |
| --------- | ----------- |
| `certbot_dns_api_email` | E-mail/Login to Access API |
| `certbot_dns_api_token` | API Token/Credential |

*Most DNS Providers require only an API token and does not require an email. Only fill required fields*

**Plugin**: `rfc2136`   
|  Variable | Description |
| --------- | ----------- |
| `certbot_dns_target_server` | Target DNS server (IPv4 or IPv6 address, not a hostname) |
| `certbot_dns_target_server_port` | Target DNS port |
| `certbot_dns_tsig_keyname` | TSIG key name |
| `certbot_dns_key_secret` | TSIG key secret |
| `certbot_dns_key_algorithm` | TSIG key algorithm |

*Considering RFC2136 is the mostly used on BIND nameservers and will require effort on server side. Example is provided below but that's outside of scope of this document how to configure BIND.*

### Sample Bind Configuration

*Generate TSIG Key*
```
dnssec-keygen -a HMAC-SHA512 -b 512 -n HOST certbot.
```

*Sample BIND configuration (named.conf)*
```
key "certbot." {
  algorithm hmac-sha512;
  secret "4q4wM/2I180UXoMyN4INVhJNi8V9BCV+jMw2mXgZw/CSuxUT8C7NKKFs AmKd7ak51vWKgSl12ib86oQRPkpDjg==";
};

zone "example.com." IN {
  type master;
  file "named.example.com";
  update-policy {
    grant certbot. name _acme-challenge.example.com. txt;
  };
};
```

**Note**: This configuration limits the scope of the TSIG key to just be able to add and remove TXT records for one specific host for the purpose of completing the `dns-01` challenge. If your version of BIND doesn’t support the `update-policy` directive, then you can use the less-secure `allow-update` directive instead. [See the BIND documentation](https://bind9.readthedocs.io/en/latest/reference.html#dynamic-update-policies) for details.

```
zone "example.com." IN {
  type master;
  allow-update {
    key certbot.;
  };
};
```

### Automatic Certificate Generation

Currently only `dns` method is supported for generating new certificates using this role. If you need to use `standalone` or `webroot` you should use `geerlingguy.certbot` role instead.   

```
certbot_create_if_missing: false
``` 
Set `certbot_create_if_missing` to `true` to let this role generate certs.  

```
certbot_testmode: false
``` 
Enable test mode to only run a test request without actually creating certificates.  

**Note**: This role defaults `certbot_testmode` to `false`. Using of Let's Encrypt [Staging Enviroment](https://letsencrypt.org/docs/staging-environment/) is recommended when setting DNS and Certbot for first time. If thats the case set `certbot_testmode` to `true`.  

If you feel like to change how certbot run, just set `certbot_create_command` to suit your needs. You can virtually use any additional plugins (besides automatic installation) using `certbot_dns_credentials_custom_file` and `certbot_create_command`.  

### Certbot Deploy Hook (Automatic Service Reload)

Since `dns` mode does not require you to stop services like `standalone` mode, which runs it's own builtin server, you may safely reload a list of services to pickup the renewed certificate.    

```
certbot_create_reload_services:
  - haproxy
  # - nginx
  # - apache
  # - httpd
```

Considering HAProxy is a fairly common load balancer and it requires to use a single file which contains full chain, certificates and private key, this role will assemble this pem file automatically for you if `haproxy` is listed as a service. Which is very helpful considering certbot knows when its going to renew or not (benefits from --deploy-hook).   

**Note**: It will only reload configured services after a sucessfully rewned (using --deploy-hook).   

### Certificate Installation

This role currently does not support or handles installating certificates on Apache, Ngnix, HAProxy or Third-party plugins supported by certbot.   

### Wildcard Certificates

This role fits perfectly for Wildcards since they require `dns-01` instead of the usual `http-01` challenge.

*Sample wildcard*
```
- certbot_certs:
    - domains:
        - *.example.com
```

Dependencies
------------

This role depends on [geerlingguy.certbot](https://galaxy.ansible.com/geerlingguy/certbot). Minimum Role Version 5.0.0.  

Example Playbook
----------------

**For a complete example**: see the fully functional test playbook in molecule/default/playbook-snap-haproxy-rhel8.yml(https://github.com/brunobenchimol/ansible-role-certbot-dns/blob/main/molecule/default/playbook-snap-haproxy-rhel8.yml).  

*Example Playbook*
```
---
- hosts: haproxy
  become: true
  gather_facts: true

  vars:
    certbot_install_method: snap
    certbot_admin_email: admin@example.com
    certbot_auto_renew: true
    certbot_auto_renew_user: "{{ ansible_user | default(lookup('env', 'USER')) }}"
    certbot_auto_renew_hour: "5"
    certbot_auto_renew_minute: "{{ 59 |random(seed=inventory_hostname) }}"  # random number between 0-59
    certbot_auto_renew_options: "--quiet --no-self-upgrade"
    certbot_create_reload_services:
      - haproxy
    certbot_create_if_missing: true
    certbot_delete_certificate: false
    certbot_certs:
      - domains:
          - www.example.com
          - certbotdnstest.example.com
    certbot_testmode: false
    certbot_dns_plugin: luadns
    certbot_dns_api_token: "<TOKEN>"
    certbot_dns_api_email: "<TOKEN_EMAIL>"

  roles:
   - brunobenchimol.certbot_dns
```

Automatic Testing (Ansible Molecule)
------------------------------------

Tests are done with 'package' installation method. Snap is currently *unsupported* to run on docker (at least all testing failed when installing certbot from snapd). Source usually fails on modern OS. Its too dificult to use DNS plugins on older OS because they lack DNS plugins when using Package Management tools.    

Snap installation method was tested on a VM running RHEL 8.  

**Note**: Slightly modified version from [geerlingguy.certbot](https://galaxy.ansible.com/geerlingguy/certbot).  

License
-------

MIT / BSD

References
----------

1. https://galaxy.ansible.com/geerlingguy/certbot
2. https://galaxy.ansible.com/michaelpporter/certbot_cloudflare
3. https://eff-certbot.readthedocs.io/en/stable/using.html

Author Information
------------------

This role was created in 2021 by [Bruno Benchimol](https://www.linkedin.com/in/brunobenchimol/).     

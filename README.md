# ansible-role-certbot-dns
Ansible Role - Certbot with DNS challenge


CLEANUP and write README

--------- TODO: REMOVE _*.yml files they are not needed anymore


Role Name
=========

A brief description of the role goes here.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).



      vars:
        - certbot_install_method: snap
        # - certbot_install_method: package
        - certbot_admin_email: test@brunobenchimol.ga
        - certbot_auto_renew: true
        - certbot_auto_renew_user: "{{ ansible_user | default(lookup('env', 'USER')) }}"
        - certbot_auto_renew_hour: "5"
        - certbot_auto_renew_minute: "{{ 59 |random(seed=inventory_hostname) }}"  # random number between 0-59
        - certbot_auto_renew_options: "--quiet --no-self-upgrade"
        - certbot_create_reload_services:
            - haproxy
        - certbot_create_if_missing: true
        # - certbot_delete_certificate: true
        - certbot_delete_certificate: false
        - certbot_certs:
            - domains:
                - www.domain.com
                - certbotdnstest.domain.com
        - certbot_use_staging_server: true
        - certbot_dns_plugin: luadns
        - certbot_dns_api_token: "tokenkey"
        - certbot_dns_api_email: "email@example.com"

certbot_dns_plugin

        - certbot_dns_plugin: rfc2136
        - certbot_dns_target_server: 177.74.3.220
        - certbot_dns_target_server_port: 53
        - certbot_dns_tsig_keyname: "certbot-sha256."
        - certbot_dns_key_secret: "xwOGjKnek3ec15X3pEsFF9Y7or8p3u1bO0FyrXw49TU="
        - certbot_dns_key_algorithm: "HMAC-SHA256"


References:

https://galaxy.ansible.com/geerlingguy/certbot
https://galaxy.ansible.com/michaelpporter/certbot_cloudflare
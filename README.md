[![ansible-lint](https://github.com/sscheib/ansible-role-register_to_satellite/actions/workflows/ansible-lint.yml/badge.svg?branch=main)](https://github.com/sscheib/ansible-role-register_to_satellite/actions/workflows/ansible-lint.yml) [![Publish latest release to Ansible Galaxy](https://github.com/sscheib/ansible-role-register_to_satellite/actions/workflows/ansible-galaxy.yml/badge.svg)](https://github.com/sscheib/ansible-role-register_to_satellite/actions/workflows/ansible-galaxy.yml)

register_to_satellite
=========

Registers a system to a Satellite instance using the [`katello-ca-consumer`](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.11/html/managing_hosts/registering_hosts_to_server_managing-hosts#registration-methods_managing-hosts)-method. 

This role is only meant for people that have not yet migrated off the **deprecated** registration using the `katello-ca-consumer` RPM. This has been deprecated since
[Red Hat Satellite 6.11](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.11/html/release_notes/assembly_introducing-red-hat-satellite_sat6-release-notes#ref_deprecated-functionality_assembly_introducing-red-hat-satellite) but is still a valid method to use with [Red Hat Satellite 6.14](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.14/html/release_notes/deprecated-functionality), although deprecated.

This role tries to be idempotent. It is assumed that a system is registered to another Satellite, Capsule or the Red Hat Customer Portal if the key `hostname` in
`/etc/rhsm/rhsm.conf` is different than the specified `rts_satellite_hostname`.

To force the registration always, please use `rts_force_registration`. This, however, is not idempotent.


Role Variables
--------------
| variable                                                  | default                      | required | description                                                                    |
| :-------------------------------------------------------- | :--------------------------- | :------- | :----------------------------------------------------------------------------- |
| `rts_satellite_hostname`                                  | unset                        | true     | Hostname of the Satellite/Capsule to register to and download the Katello RPM  |
| `rts_satellite_activation_key`                            | unset                        | true     | Activation key to use to register with the Satellite/Capsule                   |
| `rts_satellite_organization_name`                         | unset                        | true     | Satellite Organization name to use for registering with the Satellite/Capsule  |
| `rts_force_registration`                                  | `false`                      | false    | Whether to force system registration                                           |
| `rts_katello_ca_consumer_regexp`                          | `^katello-ca-consumer-`      | false    | Regular expression to find existing katello-ca-consumer packages               |
| `rts_quiet_assert`                                        | `false`                      | false    | Whether to quiet assert statements                                             |
| `rts_rhsm_conf_path`                                      | `/etc/rhsm/rhsm.conf`        | false    | Path to the RHSM configuration file                                            |
| `rts_satellite_katello_ca_consumer_package_name`          | Check `vars/main.yml`        | false    | Package name of katello-ca-consumer required for retrieving the RPM            |
| `rts_satellite_katello_ca_consumer_port`                  | `80`                         | false    | Port on which the Satellite/Capsule serves the katello-ca-consumer RPM         |
| `rts_satellite_katello_ca_consumer_published_path`        | `pub`                        | false    | Published path (after Satellite/Capsule hostname) of the katello-ca-consumer   |
| `rts_satellite_katello_ca_consumer_scheme`                | `http`                       | false    | Scheme (http/https) to use when downloading the katello-ca-consumer            |
| `rts_satellite_katello_ca_consumer_validate_certificates` | `false`                      | false    | Whether to verify the HTTPS certificates when downloading the Katello RPM      |
| `rts_satellite_port`                                      | `443`                        | false    | Port on which the Satellite is reachable for subscription-manager registration |
| `rts_satellite_validate_certificates`                     | `true`                       | false    | Whether to validate certificates during subscription-manager registration      |

## Notes

This role makes use of a custom filter named `from_ini`, which is used to read the values of `rts_rhsm_conf_path`. It is meant to be part of
[`community.general`](https://github.com/ansible-collections/community.general) in the future ([Pull Request](https://github.com/ansible-collections/community.general/pull/7743)).
For the time being, I provide the filter as custom filter via `filter_plugins`. Once the Pull Request is merged, I'll update this role, of course.

The role's variables are designed in such a way, that you can set the 'general' variables for (possibly) all hosts and only set certain variables on a host
level (e.g. `rts_satellite_activation_key`, possibly `rts_satellite_hostname` if you make use of Capsules).

Example Playbook
----------------

### Example with only the mandatory variables set
```
---
- hosts: 'all'
  gather_facts: false
  roles:
    - role: 'register_to_satellite'
  vars:
    # mandatory variables
    rts_satellite_hostname: 'satellite.example.com'
    rts_satellite_activation_key: 'ak-default-rhel-8-prod'
    rts_satellite_organization_name: 'org-core'
```

### Example with all variables set
```
---
- hosts: 'all'
  gather_facts: false
  roles:
    - role: 'register_to_satellite'
  vars:
    # mandatory variables
    rts_satellite_hostname: 'satellite.example.com'
    rts_satellite_activation_key: 'ak-default-rhel-8-prod'
    rts_satellite_organization_name: 'org-core'

    # optional variables
    rts_force_registration: false
    rts_rhsm_conf_path: '/etc/rhsm/rhsm.conf'
    rts_katello_ca_consumer_regexp: '^katello-ca-consumer-'
    rts_satellite_katello_ca_consumer_package_name: 'katello-ca-consumer-latest.noarch.rpm'
    rts_satellite_katello_ca_consumer_port: 80
    rts_satellite_katello_ca_consumer_published_path: 'pub'
    rts_satellite_katello_ca_consumer_scheme: 'http'
    rts_satellite_katello_ca_consumer_validate_certificates: false
    rts_satellite_validate_certificates: true
    rts_satellite_port: 443
```

License
-------

GPL-2.0-or-later

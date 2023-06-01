# Ansible role for StrongSwan

[![Build Status](https://travis-ci.org/torian/ansible-role-strongswan.svg)](https://travis-ci.org/torian/ansible-role-strongswan)

An Ansible Role that installs and configures StrongSwan on Red Hat/CentOS or Debian/Ubuntu.

## Tested On

  * EL / Centos (7 / 6)
  * Ubuntu (Focal / Xenial / Trusty / Precise)


### Debian Strongswan Versions
When installing Strongswan on the Debian-based distros, a `strongswan_version` var may be used. 

Available versions: https://download.strongswan.org/.
> **5.8.2** used as default in this role. 



## Role Variables

Defaults defined in `defaults/main.yml`:

```
strongswan_config_file:  {{strongswan_prefix}}/ipsec.conf
strongswan_secrets_file: {{strongswan_prefix}}/ipsec.secrets
```

The var `strongswan_prefix` is defined based on `ansible_os_family`.

The `ipsec.conf` file is configured through the following vars (and their default values):

```
strongswan_config_setup:
  charondebug: 'dmn 2, mgr 2, ike 2, chd 2, job 2, cfg 2, knl 2, net 2, enc 2 lib 2'
  uniqueids: 'yes'
  strictcrlpolicy: 'no'

strongswan_conn_default: {}

strongswan_conns: {}
```

Secrets are specified using `strongswan_secrets`. It is a list where each
element might contain the following attributes:
```
  left:       Optional - Any valid ID selector
  right:      Optional - Any valid ID selector
  type:       Optional (defaults to PSK) - any valid secret type
  credential: Required - Connection's credentials
```

An updown script can be used to specify interface parameters of the VTI interface:
```
   strongswan_updown:
     out_int: 'eth0'                # public interface
     vti_int: 'vti100'              # Virtual Tunnel Interface
     vti_local: '192.168.0.5/31'    # leftsourceip/mask
     vti_nexthop: '192.168.0.4'     # rightsourceip
```
Script path is `{{ strongswan_prefix }}`/strongswan/updown.sh. Should be referenced under `strongswan_conns`:
> leftupdown=/etc/strongswan/updown.sh

Disabled by default.


#### Logging
Default logfile is */var/log/charon.log*.

Can be changed by overwriting `charon_filelog` value. 


## Usage

The vars used for the `config setup` and `conn %default` sections are a hash that accept
any valid configuration option for strongswan. An example:

```
strongswan_conn_default:
  type: tunnel
  ikelifetime: 1h
  lifetime: 30m
  left: 1.2.3.4
```

Connections are configured with `strongswan_conns` hash:

```
strongswan_conns:
  conn1:
    right: 2.3.4.5
    rightsubnet: 2.3.4.0/24
    ike: aes256-sha1-modp1024
    esp: aes256-sha1-modp1024
    auto: start

  conn2:
    right: 3.4.5.6
    rightsubnet: 3.4.5.0/24
    auto: route
```

Setting up secrets can be done in the following way:

```
strongswan_secrets:
  - left:  1.2.3.4
    right: 2.3.4.5
    type:  PSK
    credentials: '"super wooper passw0rd"'

  - right: 3.4.5.6
    type: RSA
    credentials: cert.pem
```

The double quotes inside the simple ones is meant to escape any special chars. 

`RSA` private keys (or any secret type that requires a key file) might be specified 
through `strongswan_private_keys` (TODO):

```
strongswan_private_keys:
  - filename: cert.pem
    key: |
      --- begin your key ---
      --- end your key ---
  - filename: more_secure.pem
    key: "{{var_from_a_vault}}"
```

## License

See [License](LICENSE)


## Author Information

This role was created in 2017 by Emiliano Castagnari.


# Puppet Keepalived

## Requirements

* [concat module](https://github.com/ripienaar/puppet-concat)

## Tested on...

* Debian 6 (Squeeze)

## Example usage

### Basic IP-based VRRP failover

This configuration will fail-over when:

a. Master node is unavailable

    node /node01/ {
      include keepalived

      keepalived::vrrp::instance { 'VI_50':
        interface         => 'eth1',
        state             => 'MASTER',
        virtual_router_id => '50',
        priority          => '101',
        auth_type         => 'PASS',
        auth_pass         => 'secret',
        virtual_ipaddress => '10.0.0.1/29',
      }
    }

    node /node02/ {
      include keepalived

      keepalived::vrrp::instance { 'VI_50':
        interface         => 'eth1',
        state             => 'BACKUP',
        virtual_router_id => '50',
        priority          => '100',
        auth_type         => 'PASS',
        auth_pass         => 'secret',
        virtual_ipaddress => '10.0.0.1/29',
      }
    }

### Detect application level failure

This configuration will fail-over when:

a. NGinX daemon is not running<br>
b. Master node is unavailable

    node /node01/ {
      include keepalived

      keepalived::vrrp::script { 'check_nginx':
        script => '/usr/bin/killall -0 nginx',
      }

      keepalived::vrrp::instance { 'VI_50':
        interface         => 'eth1',
        state             => 'MASTER',
        virtual_router_id => '50',
        priority          => '101',
        auth_type         => 'PASS',
        auth_pass         => 'secret',
        virtual_ipaddress => '10.0.0.1/29',
        track_script      => 'check_nginx',
      }
    }

    node /node02/ {
      include keepalived

      keepalived::vrrp::script { 'check_nginx':
        script => '/usr/bin/killall -0 nginx',
      }

      keepalived::vrrp::instance { 'VI_50':
        interface         => 'eth1',
        state             => 'BACKUP',
        virtual_router_id => '50',
        priority          => '100',
        auth_type         => 'PASS',
        auth_pass         => 'secret',
        virtual_ipaddress => '10.0.0.1/29',
        track_script      => 'check_nginx',
      }
    }

### IP-based VRRP failover with 20+ IPs

```virtual_ipaddress``` has a 20 IP limit, and so to use more than 20 IPs, we need to use ```virtual_ipaddress_exclude```

```virtual_ipaddress_excluded``` contains a list of IP addresses that keepalived will bring up and down on the server, however they are not included in the VRRP packet itself so they don't count towards the 20 IP address limit.

In the ```virtual_ipaddress_exclude``` array, you specify the starting address, and the ending address. In the below example, it will then create the range 10.0.0.101 to 10.0.0.200

In my (@adamstrawson) configurations I like to allocate an IP specifically for ```virtual_ipaddress```. i.e. the one that is included in the VRRP packets and put everything else in ```virtual_ipaddress_excluded```.

As with the above example, this configuration will fail-over when:

a. Master node is unavailable

    node /node01/ {
      include keepalived

      keepalived::vrrp::instance { 'VI_50':
        interface         => 'eth1',
        state             => 'MASTER',
        virtual_router_id => '50',
        priority          => '101',
        auth_type         => 'PASS',
        auth_pass         => 'secret',
        virtual_ipaddress => '10.0.0.1',
        virtual_ipaddress_exclude => [ '10.0.0.101', '10.0.0.200' ],
      }
    }

    node /node02/ {
      include keepalived

      keepalived::vrrp::instance { 'VI_50':
        interface         => 'eth1',
        state             => 'BACKUP',
        virtual_router_id => '50',
        priority          => '100',
        auth_type         => 'PASS',
        auth_pass         => 'secret',
        virtual_ipaddress => '10.0.0.1',
        virtual_ipaddress_exclude => [ '10.0.0.101', '10.0.0.200' ],
      }
    }


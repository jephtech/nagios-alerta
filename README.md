Naemon-to-Alerta Gateway
========================

[![Build Status](https://travis-ci.org/alerta/nagios-alerta.png)](https://travis-ci.org/alerta/nagios-alerta)

Consolidate Nagios/Naemon alerts from across multiple sites into a single
"at-a-glance" console. Nagios 3 and Nagios 4 are now supported.

Transform this ...

![nagios](/docs/images/nagios3-v3.png?raw=true)

Into this ...

![alerta](/docs/images/nagios3-alerta-v3.png?raw=true)

System Requirements
------------

You'll need the following system packages to Install nagios-alerta:

In RedHat/CentOS/Fedora:
```
yum install -y git curl gcc make libcurl-devel
```

In Debian/Ubuntu:
```
apt-get install -y git curl gcc make libcurl-dev
```

Installation (Naemon)
------------

    $ git clone https://github.com/jephtech/naemon-alerta.git
    $ cd naemon-alerta
    $ make naemon
    $ sudo make install
    $ sudo service nagios restart

Alerts
------

To forward host and service check results to Alerta,
modify `/etc/naemon/naemon.cfg` as follows:
```
broker_module=/usr/lib/nagios/alerta-neb.o http://localhost:8080
```

To specify the environment name:
```
broker_module=/usr/lib/nagios/alerta-neb.o http://localhost:8080 env=ENV_NAME_HERE
```

To provide the API key if authentication is enabled on the alerta server:
```
broker_module=/usr/lib/nagios/alerta-neb.o http://localhost:8080 key=INSERT_API_KEY_HERE
```

To forward check results in Hard state only:
```
broker_module=/usr/lib/nagios/alerta-neb.o http://localhost:8080 hard_only=1
```

And to enable debug mode:
```
broker_module=/usr/lib/nagios/alerta-neb.o http://localhost:8080 debug=1
```

**Note:** The default `environment` is `Production` and the default `service` is `Platform`.

Setting Environment & Service Per-Check
---------------------------------------

Use [custom object variables](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/customobjectvars.html)
`_Environment` and `_Service` to set environment and service on a
per-check basis:

```
define host{
        use                     generic-host            ; Name of host template to use
        host_name               localhost
        alias                   localhost
        address                 127.0.0.1
        _Environment            Development
        _Service                Network
        }
```

```
define service{
        use                             generic-service         ; Name of service template to use
        host_name                       localhost
        service_description             Total Processes
        _Environment                    Production
        _Service                        Web
        check_command                   check_procs!250!400
        }
```

Heartbeats
----------

To configure the Nagios server to send regular heartbeats to Alerta to
ensure that Nagios and the event broker are still forwarding alerts
configure a dummy service check as follows:

1. Define a heartbeat command and add it to `/etc/nagios/commands.cfg`:
```
define command{
        command_name    check_heartbeat
        command_line    /usr/lib/nagios/plugins/check_dummy 0
}
```

2. Define a hostgroup for the Nagios servers that have the Alerta event
broker installed and add it to `/etc/nagios3/conf.d/hostgroups_nagios2.cfg`:
```
define hostgroup {
        hostgroup_name  nagios-servers
        alias           Nagios servers
        members         localhost
}
```

3. Define a Heartbeat service check to execute every minute and add it
to `/etc/naemon/conf.d/services_nagios2.cfg`:
```
define service {
        hostgroup_name                  nagios-servers
        service_description             Heartbeat
        check_command                   check_heartbeat
        use                             generic-service
        notification_interval           0 ; set > 0 if you want to be renotified
        normal_check_interval           1
}
```

License
-------

Copyright (c) 2013-2017 Nick Satterly. Available under the MIT License.

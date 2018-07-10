The omtd-standalone-setup script builds the OpenMinTeD (https://openminted.eu) workflow execution backend for a single host. This script facilitates the local installation and execution of the OpenMinTeD software stack and on a user's own infrastructure.


## Requirements

* python 2.7
* python-virtualenv
* ansible 2.7 or later

If you have trouble setting up ansible on your system, try virtualenv and run the installation script from there.

## Setup ansible

```code=bash
$ git clone https://github.com/openminted/omtd-standalone-setup.git
$ cd omtd-standalone-setup
```
If you target the local host, there is no need to edit the `hosts` file. If you target a different host for the installation, update `hosts` file to target the IP of this host.

Now, you can edit the `group_vars/all` file, although it could work as is with the default settings.

### Changes in `group_vars/all`

By default, the system will pull docker images from docker hub. If you have a private docker registry, you must uncommend the following lines. The username and password are http passwords and are optional.
```code=yaml
# registry_url: https://registry.example.com
# registry_username: omtduser
# registry_password: omtdpass
```

By default, the system will setup a postgres database with galaxy as username and password. You can change the credentials here
```code=yaml,name=group_varss/all
galaxy_db_name: galaxy
galaxy_db_user: galaxy
```

## Run the script
It is assumed that the scripts run with superuser priviledges. If this is not the case, set add a line in `group_vars/all`:

```code=yaml
galaxy_directory: /my/locally/accessible/location/galaxy
``` 

So, we are ready to fire:
```code=bash
$ ansible-galaxy install -r requirements.yaml
$ ansible-playbook -i hosts site.yaml
```

If the script completes without errors, you must have a galaxy installation on `/srv/galaxy`. Check `service galaxy status` to see if galaxy runs. You can even run it explicitely with `/srv/galaxy/run.sh`.

## Make your Galaxy accessible
Galaxy will be reachable through localhost:8080 but it is common practice to run it behind a reverse proxy. In order to do this, on the machine running galaxy e.g., for apache2:
```
$ sudo apt install apache2
$ sudo a2enmod proxy proxy_http rewrite
```

and modify `/etc/apache2/000-default.conf` to contain the following lines:
```
<VirtualHost *:80>
  ...
  ServerName <host ip>

  RewriteEngine on
  RewriteRule ^(.*) http://localhost:8080$1 [P]
  ...
</VirtualHost>
```

# Custom Resources <!-- omit in toc -->

- [Apache Resource](#apache-resource)
  - [Summary](#summary)
  - [Properties](#properties)
  - [Actions](#actions)

## Apache Resource
### Summary
**Location:** `app/cookbooks/apache/resources/apache.rb`

**Purpose:** The apache resource handles the installation and configuration of the apache webserver as part of the apache cookbook.

### Properties
#### :name
- Purpose: The resource name which can be used in its actions.
- Ruby Type(s): `String`
- Options:
  - `name_property: true`

#### :user
- Purpose: The apache user.
- Ruby Type(s): `String`
- Default Value: `node[:apache][:init][:user]`

#### :group
- Purpose: The apache group.
- Ruby Type(s): `String`
- Default Value: `node[:apache][:init][:user]`

#### :directory_list
- Purpose: An array of directories that need permissions set to the VM user.
- Ruby Type(s): `Array`
- Default Value: `default: node[:apache][:directory_list]`

#### :package_list
- Purpose: An array of the packages used to install apache.
- Ruby Type(s): `Array`
- Default Value: `default: node[:apache][:package_list]`

#### :mod_list
- Purpose: An array of apache modules to be installed.
- Ruby Type(s): `Array`
- Default Value: `node[:apache][:mod_list]`

#### :web_root
- Purpose: The web root location for apache.
- Ruby Type(s): `String`
- Default Value: `node[:apache][:init][:web_root]`

#### :http_port
- Purpose: The apache http port.
- Ruby Type(s): `String`, `Integer`
- Default Value: `node[:apache][:http_port]`

#### :php_version
- Purpose: The PHP version. Used to configure apache's php-fpm.conf.
- Ruby Type(s): `String`
- Default Value: `node[:apache][:php][:version]`

#### :fpm_backend
- Purpose: The address used for the php-fpm service backend.
- Ruby Type(s): `String`
- Default Value: `node[:apache][:php][:fpm_backend]`

#### :fpm_port
- Purpose: The port used for the php-fpm service backend.
- Ruby Type(s): `String`, `Integer`
- Default Value: `node[:apache][:php][:fpm_port]`

#### :demo_structure
- Purpose: A hash of the custom demo structure and urls.  Used to populate virtual hosts.
- Ruby Type(s): `Hash`
- Default Value: `node[:apache][:init][:demo_structure]`

### Actions
####  :clear_sites
Loops through all of the available and enabled apache sites and deletes them so that accurately-configured virtual hosts can be created based on the supplied demo structure in `config.json`.

#### Example
```ruby
apache "Install Apache" do
    action :clear_sites
end
```

#### :configure_apache
Creates `apache2.conf` based on the supplied configuration from `config.json` and other default apache cookbook attributes and then disables the default site.

##### Example
```ruby
apache "Configure Apache" do
  action :configure_apache
  web_root "/var/www/magento"
  user "vagrant"
  group "vagrant"
end
```

#### :configure_envvars
Creates `envvars` with the user and group based on the supplied configuration from `config.json` or the init cookbook default attributes.

##### Example
```ruby
apache "Configure Env Vars" do
  action :configure_envvars
  user "vagrant"
  group "vagrant"
end
```

#### :configure_fpm_conf
Creates `php-fpm.conf` based on the supplied configuration from `config.json` and other default apache cookbook attributes and then disables the default site.

##### Example
```ruby
apache "Configure php-fpm.conf" do
  action :configure_fpm_conf
  user "vagrant"
  group "vagrant"
  php_version "7.4"
  fpm_backend "127.0.0.1"
  fpm_port 9000
end
```

#### :configure_multisite
Creates and enables virtual hosts based on the supplied demo structure in `config.json`.

##### Example
```ruby
apache "Configure multisite" do
  action :configure_multisite
  http_port 80
  fpm_backend "127.0.0.1"
  fpm_port 9000
  server_name "luma.demo"
  web_root "/var/www/magento"
  demo_structure({
    website: {
      base: "luma.demo"
    }
  })
end
```

#### :configure_ports
Creates `ports.conf` based on the supplied configuration from `config.json` or the default apache cookbook attributes.

##### Example
```ruby
apache "Configure ports" do
  action :configure_ports
  http_port 80
end
```

#### :create_web_root
Creates the web root based on the supplied configuration from `config.json` or the default init cookbook `default[:init][:webserver][:web_root]` attribute.  

**Note:** The web root is defined in the init cookbook to abstract it from the apache and nginx cookbooks respectively.  This adheres to DRY and reduces code duplication.

##### Example
```ruby
apache "Create the web root" do
  action :create_web_root
  user "vagrant"
  group "vagrant"
  web_root "/var/www/magento"
end
```

#### :enable
Enables the apache service.

##### Example
```ruby
apache "Enable the Apache service" do
  action :enable
end
```

#### :install
- Loops through the supplied apache packages and installs them
- Manually creates the `libapache2-mod-fastcgi package` source file using `dpkg`
- Enables apache mods as defined in the default apache cookbook attributes
- Updates ownership of the apache logs directory
- Unmasks the apache2 service so that it can be restarted via `systemctl`

##### Example
```ruby
apache "Install Apache" do
    action :install
    user "vagrant"
    group "vagrant"
    package_list ["apache2", "apache2-bin", "apache2-data", "apache2-utils"]
    mod_list ["rewrite", "ssl", "actions", "proxy_fcgi"]
    web_root "/var/www/magento"
    http_port 80
    php_version "7.4"
    fpm_backend "127.0.0.1"
    fpm_port 9000
    demo_structure({
      website: {
        base: "luma.demo"
      }
    })
end
```

#### :restart
Restarts the apache2 service.

##### Example
```ruby
apache "Restart the Apache service" do
  action :restart
end
```

#### :set_permissions
Sets permissions for an array of apache-specific directories.

##### Example
```ruby
apache "Setting permissions on Apache files" do
  action :set_permissions
  user "vagrant"
  group "vagrant"
  directory_list ["/etc/apache2", "/var/lib/apache2/fastcgi", "/var/lock/apache2", "/var/log/apache2"]
end
```

#### :stop
Stops the apache service.

##### Example
```ruby
apache "Stop the Apache service" do
  action :stop
end
```

#### :uninstall
Removes apache and purges apache packages.

##### Example
```ruby
apache "Uninstall the Apache service" do
  action :uninstall
end
```
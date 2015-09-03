# Chef-LEMP

Kitchen to setup an Ubuntu Server ready to roll with Nginx, MySQL and PHP.

## Requirements

* Ubuntu 12.04

## Usage

To cook with this kitchen you must follow four easy steps.

### 1. Prepare your local working copy

```bash
git clone git://github.com:acidlabs/chef-lemp.git
cd chef-lemp
bundle install
bundle exec librarian-chef install
```

### 2. Prepare the servers you want to configure

We need to copy chef-solo to any server we’re going to setup. For each server, execute

```bash
bundle exec knife solo prepare [user]@[host] -p [port]
```

where

* *user* is a user in the server with sudo and an authorized key.
* *host* is the ip or host of the server.
* *port* is the port in which ssh is listening on the server.

### 3. Define the specs for each server

If you take a look at the *nodes* folder, you’re going to see files called [host].json, corresponding to the hosts or ips of the servers we previously prepared, plus a file called *localhost.json.example* which is, as its name suggests, and example.

The specs for each server needs to be defined in those files, and the structure is exactly the same as in the example.

For the very same reason, we’re going to exaplain the example for you to ride on your own pony later on.

```json
{
// This is the list of the recipes that are going to be cooked.
  "run_list": [
    "recipe[sudo]",
    "recipe[apt]",
    "recipe[dpkg_packages]",
    "recipe[timezone-ii]",
    "recipe[build-essential]",
    "recipe[runit]",
    "recipe[git]",
    "recipe[php-fpm]",
    "recipe[nginx::server]",
    "recipe[mysql::server]",
    "recipe[varnish]",
    "recipe[fail2ban]"
  ],

// You must define the sudoers users. You must created them before cooking.
  "authorization": {
    "sudo": {
      "groups"        : ["sudo","admin"],
      "users"         : ["deploy"],
      "passwordless"  : true
    }
  },

// Your MySQL configuration goes here.
  "mysql": {
    "server_debian_password" : "my_database_password",
    "server_root_password"   : "my_database_password",
    "server_repl_password"   : "my_database_password",
    "server": {
      "packages": ["mysql-server", "libmysqld-dev"]
    },

// Here you can set some custom configuration.
    "tunable": {
      "log_error"                       : "/var/log/mysql/error.log",
      "innodb_buffer_pool_size"         : "256M",
      "innodb_additional_mem_pool_size" : "16M",
      "innodb_flush_method"             : "O_DIRECT",
      "max_connections"                 : "500",
      "innodb_use_sys_malloc"           : "0"
    }
  },

// Now we configure Nginx for our deploy user.
  "nginx": {
    "user"                : "deploy",
    "client_max_body_size": "2m",
    "worker_processes"    : "auto",
    "worker_connections"  : 768,
    "repository"          : "ppa",
    "site"                : {
      "listen"         : "8080"
    }
  },
// Varnish server
  "varnish": {
    "listen_port"     : "80",
    "backend_port"    : "8080"
  },

// List all the system packages required by the services and gems you’re using in your apps.
   "dpkg_packages": {
    "pkgs": {
      "tzdata"         : { "action": "upgrade" },
      "libxml2-dev"    : { "action": "install" },
      "sendmail"       : { "action": "install" },
      "vim"            : { "action": "install" },
      "fcgiwrap"       : { "action": "install" },
      "curl"           : { "action": "install" },
      "libcurl3"       : { "action": "install" },
      "libcurl3-dev"   : { "action": "install" },
      "libxslt1-dev"   : { "action": "install" },
      "htop"           : { "action": "install" }
    }
  },

// We specicy user and group of PHP-FPM.
  "php-fpm" : {
    "user"  : "deploy",
    "group" : "deploy"
  },

// Select Timezone you want to configure
  "tz": "America/Santiago",

// Fail2ban configuration to protect our server against SSH attack attempts
  "fail2ban": {
    "bantime" : 600,
    "maxretry": 3,
    "backend" : "auto"
  }
}
```

Take a look to each cookbook `attributes/` files for more configuration parameters.

### 4. Happy cooking

We’re now ready to cook. For each server you want to setup, execute

```bash
knife solo cook [user]@[host] -p [port]
```

### 5. Recommendations

If you're using SSH Credentials to access your server, we recommend to lock your deploy user password.
That way, your user's password is going to expire in very short intervals. You can do this executing:

```bash
sudo passwd <deploy_user> -l
```

Maybe you will experience problems with `fcgiwrap`, because this service has www-data user as owner. If you get
a `Permission Denied` error trying to serve your CGI scripts, change the owner and group of the socket to match
your deploy user and group.

```bash
sudo chown <deploy_user>:<deploy:group> /var/run/fcgiwrap.socket
```

### 6. Testing against a vagrant machine with knife-solo

Initialize the vagrant machine
```bash
vagrant up
```
Then locate the ssh key used by the vagrant machine
```bash
vagrant ssh-config | grep IdentityFile | sed 's/.*IdentityFile//'
```
Finally connect and prepare && cook knife solo
```bash
knife solo prepare vagrant@127.0.0.1 -p 2222 -i /Your/vagrant/private_key
knife solo cook vagrant@127.0.0.1 nodes/vagrant.json.example -p 2222 -i /Your/vagrant/private_key
```
```

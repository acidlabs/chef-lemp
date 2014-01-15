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
bundle exec knife prepare [user]@[host] -p [port]
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
    "recipe[build-essential]",
    "recipe[ohai]",
    "recipe[runit]",
    "recipe[git]",
    "recipe[chef-rails]",
    "recipe[php-fpm]",
    "recipe[nginx]",
    "recipe[nginx::apps]",
    "recipe[mysql::server]",
    "recipe[monit]",
    "recipe[monit::mysql]",
    "recipe[monit::nginx]",
    "recipe[monit::ssh]"
  ],

// You must define the sudoers users. You must created them before cooking.
  "authorization": {
    "sudo": {
      "groups"        : ["deploy"],
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
    "user"          : "deploy",
    "distribution"  : "precise",
    "components"    : ["main"],

// Here we configure Nginx Virtual Hosts
    "apps": {

// This is an example of a PHP application with a posible HTML index.
      "example.com": {
        "listen"     : [80],
        "server_name": "example.com www.example.com xxx.xxx.xxx.xxx",
        "public_path": "/home/deploy/www.example.com",
        "locations": [
          {
            "path": "/",
            "directives": [
              "index index.html index.php",
              "try_files $uri /index.php?q=$uri&$args;"
            ]
          },
          {
            "path": "~ \\.php$",
            "directives": [
            "fastcgi_pass unix:/var/run/php-fpm-www.sock;",
              "fastcgi_index index.php;",
              "include fastcgi_params;"
            ]
          }
        ]
      },

// Another application example, in this case serving a CGI script.
      "appcgi.example.com": {
        "listen"     : [80],
        "server_name": "appcgi.example.com",
        "public_path": "/home/deploy/appcgi.example.com/",
        "locations": [
          {
            "path": "/",
            "directives": [
              "try_files $uri /index.php?q=$uri&$args;"
            ]
          },
          {
            "path": "~ \\.php$",
            "directives": [
            "fastcgi_pass unix:/var/run/php-fpm-www.sock;",
              "fastcgi_index index.php;",
              "include fastcgi_params;"
            ]
          },
          {
            "path": "/cgi-bin",
            "directives": [
              "gzip off;",
              "fastcgi_pass unix:/var/run/fcgiwrap.socket;",
              "include /etc/nginx/fastcgi_params;",
              "fastcgi_param SCRIPT_FILENAME /home/deploy/appcgi.example.com$fastcgi_script_name;"
            ]
          }
        ]
      }
    }
  },

// All the packages you need goes here.
  "chef-rails": {
    "packages": ["libxml2-dev", "libxslt1-dev", "libncurses5-dev", "libncurses5-dev", "sendmail", "vim", "php5-mysql", "php5-curl", "fcgiwrap", "curl", "libcurl3", "libcurl3-dev"]
  },

// We specicy user and group of PHP-FPM.
  "php-fpm" : {
    "user"  : "deploy",
    "group" : "deploy"
  },

// Setting monit
  "monit" : {
    "notify_email"     : "notify@example.com",
    "poll_period"      : "60",
    "poll_start_delay" : "120"
  }
}
```

Take a look to each cookbook `attributes/` files for more configuration parameters.

### 4. Happy cooking

We’re now ready to cook. For each server you want to setup, execute

```bash
knife cook [user]@[host] -p [port]
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
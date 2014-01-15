## v0.101.6:

Erroneous cookbook upload due to timeout.

Version #'s are cheap.

## v0.101.4:

* [COOK-1280] - Improve RHEL family support and fix ohai_plugins
 recipe bug
* [COOK-1194] - allow installation method via attribute
* [COOK-458] - fix duplicate nginx processes

## v0.101.2:

* [COOK-1211] - include the default attributes explicitly so version
is available.

## v0.101.0:

**Attribute Change**: `node['nginx']['url']` -> `node['nginx']['source']['url']`; see the README.md.

* [COOK-1115] - daemonize when using init script
* [COOK-477] - module compilation support in nginx::source

## v0.100.4:

* [COOK-1126] - source version bump to 1.0.14

## v0.100.2:

* [COOK-1053] - Add :url attribute to nginx cookbook

## v0.100.0:

* [COOK-818] - add "application/json" per RFC.
* [COOK-870] - bluepill init style support
* [COOK-957] - Compress application/javascript.
* [COOK-981] - Add reload support to NGINX service

## v0.99.2:

* [COOK-809] - attribute to disable access logging
* [COOK-772] - update nginx download source location

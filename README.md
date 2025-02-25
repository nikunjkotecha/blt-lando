Acquia BLT Lando integration
====

This is an [Acquia BLT](https://github.com/acquia/blt) plugin providing [Lando](https://lando.dev/) integration.

This plugin is **community-created** and **community-supported**. Acquia does not provide any direct support for this software or provide any warranty as to its stability.

## OS Support ##

While Lando itself can be used with any operating system (see their [system requirements here](https://docs.lando.dev/basics/installation.html)) this plugin has been built with a Mac OS host in mind. See my [blog for details on setting up a Macbook for work as a host machine](https://mikemadison.net/blog/2020/7/21/setting-up-a-new-macbook-pro-for-local-development). 

This project can be used on non-Windows hosts, but you will have to remove the ssh-fix tooling and the related ssh code (as it it specific to Mac OS).

## Installation and usage

To use this plugin, you must already have a Drupal project using BLT 11 (or higher).

1. Add this plugin to your project using composer: 

`composer require --dev mikemadison13/blt-lando`

Note: if you are using drupal/recommended-project (or another project template) that sets the minimum-stability to "stable" you may have trouble installing this plugin. You can either change your stability to "dev" or more specifically composer require like:

`composer require mikemadison13/blt-lando:dev-main`

2. Ensure that your blt/blt.yml file has properly set the `project.machine_name` key (as this will be used during template generation by this plugin).

3. Initialize the Gitlab integration by calling `blt recipes:vm:lando`, which is provided by this plugin.

This will copy a template version of the .lando.yml to your project root directory and instantiate a number of other BLT and Drupal settings files required to make Drupal bootstrap properly in a Lando container. Make sure to commit these files (where appropriate, remember that BLT gitignores local settings and local blt files by default, and these "local" specific files should not be committed) as well as your updated composer.json to Git.

Note: the template YAML file assumes standard BLT steps for builds and is based upon the drupal9 community image. The plugin will attempt to customize the file based on your project's machine and host names as defined in your blt.yml file.

4. Carefully review the created .lando.yml file prior to running `lando start`

5. Once Lando has been provisioned, run `lando blt setup` to install Drupal via BLT.

### Accessing Drupal Logs
To access Drupal logs you need to enable the syslog module available in Drupal CORE and 
then execute `lando logs-drupal` command.

It by default tails the logs. For searching inside logs you can check in `/var/logs/drupal.log` file

Note: syslog service is started after you run `lando logs-drupal` post every lando start or restart.

### Using XHPROF
To use XHPROF you just need to turn on the [XHPROF](https://www.drupal.org/project/xhprof) module and 
configure it to use tideways, it should be configured automatically on install.

You can access the reports from the admin screen of XHPROF module inside Drupal itself. No separate GUI 
is configured as of now in the setup.

## Constraining Composer

Now that composer 2 is out, Lando should default to composer 2. HOWEVER if you are running much older versions of Drupal, you may still need composer 1. You can add this into your project manually with the following config key:

```yaml
config:
  composer_version: '1.10.22'
```

## Adding Solr

The .lando.yml file included with this plugin does not include an Apache Solr service by default because not all projects need Solr!

However, if you wish to use Solr, simply add an additional service to the .lando.yml file like the following:

```yaml
services:
  solr:
    type: solr:7.7
    core: drupal
    portforward: true
```

Then rebuild your VM using `lando rebuild`.

Note that for Search API you'll need the following information:

Connector: standard
scheme: http
host: solr
port: 8983
path: /
core: drupal 

notes: 
* the core is configurable so if you want it to be something else, change the core definition in the service definition and update the solr config to be the same!
* the internal / localhost connectivity for the server is NOT the same as the service url that lando will report (and that's ok)
* even though Search API ships a config file, I have not been able to get the solr service to recognize and pull the config file from the appserver (so there is not currently a config path here)

## Debugging with XDebug

Between the Drupal8 recipe for Lando and this config file, XDebug is pre-loaded onto the appserver (web) container. You will need to toggle XDebug on when you wnat to use it, which I have provided simple tooling for:

```yaml
lando xdebug-on
lando xdebug-off
```

No restart of the VM is required, just these commands!

However, simply turning on XDebug isn't enough, as you'll still have to manually configure PHPStorm for each project. At a high level, this is broken down into three steps:

1. Add a server in PHPStorm (or other IDE)
2. Add a PHP Web Page debug config (using the server)
3. Instruct both the IDE and your browser to start debugging / listening 

Here's an [article on configuring PHPStorm](https://docs.lando.dev/guides/lando-phpstorm.html). Note that the Drupal8 recipe comes pre-configured to debug in PHPStorm. Other IDEs will require some additional tweaks.

Finally, it is possible despite fully configuring that XDebug is still not functioning. [This article](https://untoldhq.com/blog/2019/08/02/when-lando-phpstorm-and-xdebug-setup-gets-hairy) walks through additional steps that will ensure that PHPStorm detects that the PHP version running inside the container has XDebug.

# License

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License version 2 as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

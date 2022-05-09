Why LAMPready?
---

A LAMP is an acronym describing a Web server that runs on a Linux platform with
[Apache](https://httpd.apache.org/), [MySQL](https://www.mysql.com/) and
[PHP](https://www.php.net/).  Web pages with [Python](https://www.python.org/)
and [Perl](https://www.perl.org/) can also be served under this environment. A
LAMP stack generally refers to a testing environment where website code can be
tested for production (professional) deployment. 

There are a lot of great LAMP packages out there.  However, I wanted something
I could spin up fast without a lot of manual intervention.  Each website is
unique in the way it is served and maintained.  This script takes away a lot
of the heavy lifting and complexity of setting up a LAMP (or virtual
environment).  This is also referred to VM for short, meaning virtual machine.

This LAMP stack is not too different from an actual working production Web
server.  As a general warning though, this environment is only intended to be
a template of a Web server.  If anything, it is an oversimplified version of
such that provides a stable environment for testing code.

How it Works
---

This script syncs your project's root folder inside a virtual machine.  That
allows you to work on and debug code in a working Web server environment.
Most of these virtual environments do not have mail utilities set up because
dealing with the fallout of development spam is never fun.  Sometimes those
forms and crons need tested.  LAMPready uses
[Postfix](https://www.postfix.org/) to route all outbound mail to a single
convenient inbox that can be accessed with a local mail client.


Built on Red Hat Linux
---

This script is based off
[Scott McCarty's](https://crunchtools.com/moving-linux-services-to-containers)
thoughts on using a single podman container to build a LAMP stack.  If you use
[podman](https://podman.io/), you might want to test this out.  If you aren't
familiar with podman, hopefully a quick scan through this script will demystify
the process.

In this example Podman will need to be logged into registry.access.redhat.com.
However you may find something suitable on one of the non-subscription repositories.
In either case, this script is designed to work with 
[Red Hat Enterprise Linux](https://www.redhat.com) or RHEL for short.

```
podman login registry.access.redhat.com
```

Right now, Red Hat is allowing access to this repository at no charge.  All you
need to do is
[register](https://developers.redhat.com/#assembly-field-sections-7105). Since
CentOS has moved out of the production space, this is a good way to see websites
working in the RHEL environment.  Thankfully, there's not much difference.

*By default a container does not use up one of your deployment slots.  Inside the
container you already have access to the main repository.*

If you plan on using the standard ports 80 and 443, you will need to expose priviliged
ports to users.  Run the following commands under root:

```
echo "net.ipv4.ip_unprivileged_port_start=0" > /etc/sysctl.d/05-expose-privileged.conf
sysctl --system
```

LAMPReady is a rootless container. As such, there are many configuration standards
and standard practices to setting up a web server that have been broken simply
for the sake of productivity.

Don't let this fool you.  The user will still need access to the root account
if they plan on using standard Web server ports as noted above.  Root access will
also be needed to make entries to the firewall and SELinux in order for 
Xdebug to work properly.

How to Install the RHEL Box?
---
Download the box script into your project's root directory.  A pretty rough
video demonstration can be seen
[here](https://bridgesense.com/blog/making-friends-with-podman).

```
curl https://raw.githubusercontent.com/bridgesense/lampready/master/box > box
```

__PLEASE NOTE:__ Be sure to review the notes at the head of the box script.
Xdebug is disabled by default. This box is designed to run container processes
with root permission, which includes Apache.  In a rootless container it is the
only way to preserve the proper permissions of shared files. In order for xdebug
to work as intended, some local network information is exposed when enabling it.

As you will notice further on, this script's commands closely models
[Vagrant](https://www.vagrantup.com/).  The advantage of running a container is
that I no longer need to maintain a fully encapsulated operating system image.

To see a list of commands, just run:

```
bash box
```

* Run the following command to spin up the new virtual environment in minutes

```
bash box up
```

* Log into to shell to check out the virtual environment with:

```
bash box ssh
```

* Access the website via https://127.0.01 or use your custom domain
settings (see below)

* Access outbound mail using the following command:
```
bash box mail 
```


How to Configure this Script for Your Project
---
The very first time you run the "vagrant up" command, the hostname and
"User Defined Settings" will be used to set up Apache, Xdebug, the SSL
certificate, mail relay and any databases without further intervention.


**HOSTNAME**
The hostname should contain your domain name. 

**SUB_DOMAIN**
It is recommended that you choose a unique sub domain for your virtual machine.
This way your public domain is active for comparison.  Next, alter your system's
hosts file to allow the new domain to be accessed from any of your browsers.  On
Linux systems, this host file can be found at /etc/hosts.  You'll need root
access to add the following line or something similar:
```
127.0.0.1   dev.lampready.com
```

**PUBLIC_ROOT_PATH**
This script should be placed in your project's root directory.  If your
root directory contains an index file, you'll need to leave this setting blank.
Otherwise, you'll include the local path to your index file.  The example
default entry in this script assumes the following directory structure:
```
box
user_database.sql
public_html/index.php
```

In another example, let's say a Magento store resides within a subdirectory of
an existing website, you might change the root path to "public_html/store"
assuming your directory structure is as follows
```
box
user_database.sql
public_html/index.php
public_html/store/index.php
public_html/store/app
```

The above examples should demonstrate the fact that this script is
designed to set up one website with one corresponding URL out of the box.
However, you'll notice a new hidden directory formed in your project root
directory  You can add more URLS by adding the appropriate configuration
files to: .container/conf.d


**HTTP_PORT**
The this port generally defaults to 80.  If you have root access, you are
encouraged to use this port for full compatibility of the code that
is now running in a virtual environment.


**SSL_PORT**
This port defaults to 443, but may be changed.  Remember though, if the
port is different, it should be part of the URL entered into the browser:

```
https://dev.lampready.com:443
```

**PHP_VERSION**
This option is intended to provide easier debugging during PHP upgrades.  PHP
versions 7.2, 7.3 and 7.4 are ready to use.  Each version has access to the
most common modules.


**PHP_MAX_EXEC_TIME**
This is the maximum execution time a PHP script runs before it times out.


**PHP_MEM_LIMIT**
This is the maximum amount of memory a single PHP script can use.  Remember,
this is not a virtual environment per se.  This is a rootless container where
memory is not reserved, only used during the course of the Web server's
operations.  This makes for a more efficient environment for testing
software.


**XDEBUG_ENABLE**
Xdebug is not enabled by default. Xdebug must be enabled before the initial
setup process begins.  If Xdebug is not enabled during the first initialization
of this VM will have to be destroyed and recreated.


**XDEBUG_PORT**
This is the default port normally allocated for Xdebug.  You'll want to be sure
to open up your firewall to allow communication between Xdebug and your browser.
Think about the following changes that will need to be made with the root user
in order for Xdebug to function:
```
firewall-cmd --permanent --zone=webserver --add-port=9003/tcp
semanage port -a -t http_port -p tcp 9003
```

**XDEBUG_FORCE_ERROR_DISPLAY**
For sites constructed from many packages with different settings, it is nice to
be able to easily override error suppression.  Set this option to 1 to
override individual error settings throughout the code.


**XDEBUG_SCREAM**
This is another setting that can help ensure all PHP errors are described
in the browser.


**DB1**
The script can automatically set up and install up to three MySQL databases.
By filling out this information, the script will create the database and inject
your data from a sql file you have placed in the project's root directory.

**DB_NAME, DB_USER and DB_PASS**
These settings should mirror the same database name and user credentials
written in the website's database configuration file.  Notice the single quotes
used for the password.  This is an important work-around to allow the bash
script to insert certain special characters correctly.


**DB_PERM**
Assign DB permissions to specific users.


**DB_PREFIX**
If there are any prefixes associated with your database as commonly is the case
with Wordpress, this should be entered here.  An example would be: wp_


**DB_TYPE**
If the database is Magento or Wordpress, you will want to set this option to say
so.  The following arguments are valid: custom, wordpress, magento_1 or
magento_2.  The script will update the appropriate tables to match this
script's HOSTNAME and SUB_DOMAIN settings.  For typical installations no manual
fiddling should be necessary.


**DB_FILENAME**
This option is not neccessary if there is no database to import.  While the
DB_NAME option will create an empty database, this option populates that
database.  The script will look for the sql file in your projects root
directory (one directory below document root).


**DB_CUSTOM_FUNCTIONS**
This option is used for importing an additional script or custom functions.


How to Use this Box?
---
After installing, LAMPready must be started and managed from the terminal:

**to initialize or start the server:**
```
bash box up
```

**stop the server:**
```
bash box halt
```

**delete the server:**
```
bash box destroy
```
**Note:** If you run Vagrant destroy, the box will be built from scratch again
-- which may be a good thing.


**to shell into the server:**
```
bash box ssh
```

**check if you are running any other LAMPReady instances:**
```
vagrant box ps 
```

**to destroy all LAMPReady installations:**
```
vagrant box reset 
```

Xdebug Settings
---

Xdebug port: 9003 (*see script setting*)


Emacs and Dap-Mode 
---

Dap-Mode is part of the [Emacs-Lsp](https://emacs-lsp.github.io/) package which
is a very awesome PHP IDE!  LAMPReady is a rootless container and therefore needs
the correct path mapping to the project's root directory in your virtual
environment as well as the directory where the files are physically located.
There are some excellent notes on
[Spacemac's implementation](https://develop.spacemacs.org/layers/+lang/php/README.html#backends)

DAP-Mode actually uses the PHP VSCode plugin
[vscode-php-debug](https://github.com/xdebug/vscode-php-debug) which can be
installed with: M-x dap-php-setup 

You'll also want to create a file called launch.json and place the following 
information in it:

```
{
"configurations": [

    {
        "name": "Use custom launch.json script",
        "type": "php",
        "request": "launch",
        "port": 9003,
        "pathMappings": {
            "/var/www": "/path/to/the/files/on/your/system"
          },
        "sourchMaps": true,
        "log": false 
    }
]
}
```

After running dap-php-setup, you should have a new option called "Use custom
launch.json script" to choose.  For more information and an example Emacs
setup, check out my [dotfiles](https://lampready.com).

VIM with the Vdebug Plugin
---

Here is a sample user configuration for your .vimrc file.  This tells VIM
to look in your projects root folder for a file called .vimrc.local with
custom path mapping needed to run Vdebug with the rootless container.

```
" Look for .vimrc.local inside each project
if filereadable('.vimrc.local')
    source .vimrc.local
    endif
```

Create a file called .vimrc.local and place it in the root directory of your
project -- the same one as your Vagrantfile script. Be sure that the path
mapping matches the project's <b>root directory</b> or your break points will
not work.  The /var/www will stay the same as it is the synced location of
your project.

```
" Project Specific Settings for Vdebug
let g:vdebug_options = {
\    "port" : 9003,
\    "timeout" : 20,
\    "server" : '',
\    "on_close" : 'stop',
\    "break_on_open" : 0,
\    "ide_key" : 'xdebug',
\    "debug_window_level" : 0,
\    "debug_file_level" : 0,
\    "debug_file" : "~/.vdebug.log",
\    "path_maps" : {"/var/www" : "/Users/Paul/www/your_projects_root_dir"},
\    "watch_window_style" : 'expanded',
\    "marker_default" : '⬦',
\    "marker_closed_tree" : '▸',
\    "marker_open_tree" : '▾',
\    'sign_breakpoint' : '▷',
\    'sign_current' : '▶',
\    "continuous_mode"  : 1,
\    'simplified_status': 1,
\    'layout': 'vertical',
\}
let g:vdebug_features = {
\ 'max_data': 512,
\ 'max_children': 128,
\ 'max_depth': 3,
\}
```

Why LAMPready?
---

There are hundreds of great Vagrant boxes out there.  However, I wanted something I could spin up fast without a lot of manual intervention.

How it Works
---

This script syncs your project's root folder inside the virtual machine.  That allows you to work on the code from either in or outside the box.  Most of these virtual environments do not have mail utilities set up because dealing with the fallout of development spam is never fun.  Sometimes those forms and crons need tested.  LAMPready uses Postfix to route all outbound mail to a single convenient box that can be accessed with the local mail client.


The RHEL Box
---

I recently added the _box_ script which is based off [Scott McCarty's](https://crunchtools.com/moving-linux-services-to-containers)
thoughts on using a single podman container to build a LAMP stack.  If you use
[podman](https://podman.io/), you might want to test this out.  If you aren't
familiar with podman, hopefully a quick scan through this script will
demystify the process. 

In this example Podman will need to be logged into the _registry.access.redhat.com_.  
However you may find something suitable on one of the non-subscription repositories.
In either case, this script is designed to work with Red Hat Enterprise Linux (RHEL).

```
podman login registry.access.redhat.com
```

Right now, Red Hat is allowing access to this repository at no charge.  All you need
to do is [register](https://developers.redhat.com/#assembly-field-sections-7105) as a 
developer.  Since CentOS has moved out of the production space, this is a good way to see
websites working in the RHEL environment.  Thankfully, there's not much difference.

*By default a container does not use up one of your deployment slots.  Inside the
container you already have access to the main repository.*

How to Install the RHEL Box?
---
Download the box script into your project's root directory.  A pretty rough
video demonstration can be seen [here](https://bridgesense.com/blog/making-friends-with-podman).
Most of the notes on the Vagrant box described below apply, as this script
closely models it.

```
curl https://raw.githubusercontent.com/bridgesense/lampready/master/box > box
```

__PLEASE NOTE:__ Be sure to review the notes at the head of the box script.
Xdebug is disabled by default. This box is designed to run container processes
with root permission, which includes Apache.  In a rootless container it is the
only way to preserve the proper permissions of shared files. In order for xdebug
to work as intended, some local network information is exposed when enabling it.

To see a list of commands, just run:

```
bash box
```


What is a Vagrant Box?
---
Vagrant allows developers to distribute a consistant development environment with their code.  This box was built with CentOS 8.2.

This development environment includes:

Apache, PHP7 and Postfix which are already setup and ready to run from your project directory.

This box also includes a set of lightweight development tools to customize Vim and Tmux from [YADRLite](https://github.com/bridgesense/dotfiles).



How to Install this Vagrant Box?
---
* Download and Install VirtualBox from https://www.virtualbox.org

* Download and install Vagrant from https://www.vagrantup.com

* Download the LAMPready Vagrantfile script to your project's root directory

```
curl https://raw.githubusercontent.com/bridgesense/lampready/master/Vagrantfile > Vagrantfile
```

* You'll want to make some adjustments to the script's options (see below)

* Run the following command to spin up the new virtual environment in minutes

```
vagrant up
```

* Log into to shell to check out the virtual environment (user: vagrant, pass: vagrant)

```
vagrant ssh
```

* Access the website via https://192.168.33.10 or use your custom domain settings (see below)

* Access outbound mail from https://192.168.33.10/webmail (user: vagrant, pass: vagrant)


How to Configure this Script for Your Project
---
The very first time you run the "vagrant up" command, the config.vm.hostname and "User Defined Settings" will be used to set up Bind, Apache, Xdebug, the SSL certificate, mail relay and up to three databases without further intervention.


**config.vm.hostname and SUB_DOMAIN**
The config.vm.hostname should contain your domain name.  It is recommended that you choose a unique sub domain for your virtual machine.  This way your public domain is active for comparison.  Next, alter your hosts file to allow the new domain to be accessed from any of your browsers and via command-line.
```
192.168.33.10   dev.lampready.com
```


**DOCUMENT_ROOT and DB_FILENAME**
This Vagrant script should be placed in your project's root directory.  If your root directory contains an index file, you'll need to leave this setting blank.  Otherwise, you'll include the local path to your index file.  The example default entries in this script assume the following directory structure:
```
Vagraingfile
user_database.sql
public_html/index.php
```

In another example, let's say a Magento store resides within a subdirectory of an existing website, you might change DOCUMENT_ROOT to "public_html/store" assuming your directory structure is as follows:
```
Vagrantfile
user_database.sql
public_html/index.php
public_html/store/index.php
public_html/store/app
```

The above examples should demonstrate the fact that this Vagrant script is designed to set up one website with one corresponding URL.


**MAIL_RELAY**
This line shouldn't need to be modified. Roundcube is already set up and ready to go. With exception of direct connection to a remote SMTP service through an API, all other outgoing mail will be diverted to a local box.  This should prevent any developer spam from harassing your colleagues.  You can access this box from your browser:
```
https://dev.lampready.com/webmail
```

**PHP_VERSION**
This option is intended to provide easier debugging during PHP upgrades.  PHP versions 7.2, 7.3 and 7.4 are ready to use.  Each version has access to the most common modules.


**XDEBUG_PORT**
This is the default port normally allocated for Xdebug.  You'll want to be sure to open up your firewall to allow communication between Xdebug and your browser.


**XDEBUG_FORCE_ERROR_DISPLAY**
For sites constructed from many packages with different settings, it is nice to be able to easily override error suppression.  Set this option to "yes" to override individual error settings throughout the code.


**XDEBUG_MAX_DATA, CHILDREN, DEPTH**
Set limitations on the variables returned by Xdebug.


**INSTALL_DB**
The script can automatically set up and install up to three MySQL databases.  By setting the INSTALL message to "yes"  and filling out the rest of the information, the script will create the database and inject your data from a sql file you place in the project's root directory.


**DB_IS_MAGENTO or DB_IS_WORDPRESS**
If the database is Magento or Wordpress, you will want to set this option to "yes".  For Magento 2 set this option to "2".  The script will update the appropriate tables to match this script's config.vm.hostname and SUB_DOMAIN settings.  For typical installations no manual fiddling should be necessary.


**DB_NAME, DB_USER and DB_PASS**
These settings should mirror the same database name and user credentials written in the website's database configuration file.  Notice the single quotes used for the password.  This is an important work-around to allow the bash script to insert certain special characters correctly.


**DB_PERM**
Assign DB permissions to specific users.


**DB_FILENAME**
This option is not neccessary if there is no database to import.  While the DB_NAME option will create an empty database, this option populates that database.  The script will look for the sql file in your projects root directory (one directory below document root).


**DB_CUSTOM_FUNCTIONS**
This option is used for importing an additional script or custom functions.


How to Use this Box?
---
After installing, Vagrant will not run from being launced in VirtualBox. Vagrant must be started and managed from the terminal:

**initialize LAMPready without the script**
```
vagrant init bridgesense/lampready
```

**start/install the VM server:**
```
vagrant up
```

**pause the server:**
```
vagrant suspend
```

**stop the server:**
```
vagrant halt
```

**delete the server:**
```
vagrant destroy
```

**quick SSH into the server:**
```
vagrant ssh
```

**check if you are running the latest version:**
```
vagrant box outdated
```

**update to latest version:**
```
vagrant box update
```


**Note:** If you run Vagrant destroy, the box will be built from scratch again -- which may be a good thing.



Credentials
---

USER: root<br />
PASS: vagrant<br />

SSH Host: 192.168.33.10<br />
SSH User: vagrant<br />
SSH Pass: vagrant<br />

MySQL<br />
Database User: root<br />
Database Pass: root<br />

Roundcube<br />
URL: 192.168.33.10/webmail<br />
USER: vagrant<br />
PASS: vagrant<br />


**NOTE:** In order to manage your databases with Navicat or other software, enter your SSH credentials and access MySQL like you would any other remote server.  If you have an issue logging in, try using 127.0.0.1 as the host instead of localhost.


Xdebug Settings
---

Xdebug port: 9041 (*see script setting*)

Xdebug is preconfigured and ready to go.  There are a few things to be aware of when working with Xdebug within an IDE.

**FIREWALL NOTICE:**  If you're blocking all inbound connections, Xdebug will NOT interface with your local IDE.  Be sure to open port 9041 or which ever port was entered in the Vagrantfile script.


VIM & Vdebug Settings
---

Here is a sample user configuration for your .vimrc file.

```
" Look for .vimrc.local inside each project
if filereadable('.vimrc.local')
    source .vimrc.local
    endif
```

Create a file called .vimrc.local and place it in the root directory of your project -- the same one as your Vagrantfile script. Be sure that the path mapping matches the project's <b>root directory</b> or your break points will not work.  The /var/www will stay the same as it is the synced location of your project.

```
" Project Specific Settings for Vdebug
let g:vdebug_options = {
\    "port" : 9041,
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



SublimeText Settings
---

Here is a sample user configuration from SublimeText.

```
{
    "port": 9041,
    "max_depth": 5,
    "max_children": 128,
    "break_on_exception": [
        // E_ERROR, E_CORE_ERROR, E_COMPILE_ERROR, E_USER_ERROR
        "Fatal error",
        // E_RECOVERABLE_ERROR (since PHP 5.2.0)
        "Catchable fatal error",
        // E_WARNING, E_CORE_WARNING, E_COMPILE_WARNING, E_USER_WARNING
        "Warning",
        // E_PARSE
        "Parse error",
        // E_NOTICE, E_USER_NOTICE
        "Notice",
        // E_STRICT
        "Strict standards",
        // E_DEPRECATED, E_USER_DEPRECATED (since PHP 5.3.0)
        "Deprecated",
        // 0
        "Xdebug",
        // default
        "Unknown error"
    ],
    "super_globals": true,
    "close_on_stop": true,
    "debug": true
}
```

You also add a setup specific to the account via a SublimeText project file. Unlike the Vim settings above, be sure that the path mapping points to your project's *public* root directory.

```
{
    "folders":
    [
        {
            "follow_symlinks": true,
            "path": "."
        }
    ],
    "settings":
    {
        "xdebug": {
            "path_mapping":{
                "/var/www/public_html/" : "/Users/Paul/www/your_projects_root_dir/your_public_html_dir/"
            }
        }
    }
}
```



Emacs & Geben Settings
---

All Geben needs is the correct path mapping to the project's root directory (not the public directory unless they are both the same).  See here for operational details: [GIT: ahungry/geben](https://github.com/ahungry/geben#vms)

M-x customize-variable geben-path-mappings

![screenshot](https://bridgesense.com/images/dotfiles/geben1.png)

You can also modify your .emacs file directly.
```
;; Geben Path Mapping
(setq geben-path-mappings '(("<project base on host>" "<project base on vm>"))
```

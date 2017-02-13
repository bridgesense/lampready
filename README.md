Notes:
---

What goes in the box -- Stays in the box.

A bulletproof LAMP stack designed for PHP developers. Be up and debugging in minutes without a care in the world.

XdebugBox also includes a mail shield that captures all outbound mail and routes it to the address defined in the Vagrantfile script.  No extra configuration or fiddling with your project's config files is neccessary.

More technical info to come....

What is a Vagrant Box?
---
Vagrant allows developers to distribute a consistant development environment with the rest of the project code without taking up the space.

This box was built from scratch from Ubuntu 14.04 Trusty. It has been tested on MacOS, but is available for Windows and Linux distros.

This development environment includes:

Apache, Bind, Dovecot, PHP5, Postfix, and Roundcube which are already setup and ready to run from your project directory. 

This box also includes a set of lightweight development tools to customize Vim and Tmux. [YADRLite] (https://github.com/bridgesense/dotfiles)  


How to Install this Vagrant Box?
---
1. Download and Install VirtualBox from https://www.virtualbox.org

2. Download and install Vagrant from https://www.vagrantup.com

3. Open up your terminal in your project's root directory and run the following command:

```
vagrant init bridgesense/xdebugbox
```
4. Overwrite the default Vagrantfile script with the one provided here.

5. Fill free to change any of the Hostname and other user defined options. 

6. Start the script with this command:
```
vagrant up
```


How to Use this Box?
---
After installing Vagrant will not run from being launced in VirtualBox. Vagrant must be started and managed from the terminal: 

**initialize XdebugBox

**start/install the VM server:**
```vagrant up```

**pause the server:**
```vagrant suspend```

**stop the server:**
```vagrant halt```

**delete the server:**
```vagrant destroy```

**quick SSH into the server:**
```vagrant ssh```

**check if you are running the latest version:**
```vagrant box outdated```

**update Scotch Box:**
```vagrant box update```


**WARNING!** If you run Vagrant destroy, you will have to reinstall
your database.  Any previous changes to the database will be gone.


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
Database Host: localhost / 127.0.0.1<br />

Roundcube<br />
URL: 192.168.33.10/webmail<br />
USER: vagrant<br />
PASS: vagrant<br />


**NOTE:** In order to manage your databases with Navicat or other software, enter your SSH credentials and access MySQL like you would any other remote server.  If you have an issue logging in, try using 127.0.0.1 as the host instead of localhost.


Xdebug Settings
---

Xdebug port: 9041 (*see script setting*)

Xdebug is preconfigured and ready to go.  There are a few things to be aware of when working with Xdebug within an IDE.

Be sure to turn off E_STRICT break on exceptions.  Scotchbox is distributed with PHP 5.6 which includes E_STRICT by default.

**FIREWALL NOTICE:**  If you're blocking all inbound connections, Xdebug will NOT interface with Sublime Text or Vdebug.  Be sure to open port 9041 or what ever port was entered in the Vagrantfile script:


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
\    "debug_file_level" : 1,
\    "debug_file" : "~/.vdebug.log",
\    "path_maps" : {"/var/www" : "/Users/Paul/www/projects_root_dir"},
\    "watch_window_style" : 'expanded',
\    "marker_default" : '⬦',
\    "marker_closed_tree" : '▸',
\    "marker_open_tree" : '▾',
\    "continuous_mode"  : 0
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
                "/var/www/public_html/" : "/Users/Paul/www/projects_root_dir/public_html/"
            }
        }
    }
}
```

![screenshot](http://i.imgur.com/yBAlToR.jpg)



Why XdebugBox?
---

There are hundreds of great Vagrant boxes out there.  Personally, I was tired of tinkering with broken boxes brewed from anonymous missing repos.  I wanted something I could spin up fast and just get to work. This project is fairly new, but it already has some features I'm pretty excited about.

* A lot less tinking with a website's configuration before launching it in a virtual environment
* ... no more developer SPAM!  The mail shield is set up and ready to go.  No adjustments are neccessary.
* Xdebug is setup and ready to go with a light selection of Vim Tools, Tmux and Ranger

####_What goes in the box -- Stays in the box._


How it Works
---

This script syncs your project's root folder inside the virtual machine.  That allows you to work on the code from either in or outside the box.  Most of these virtual environments do not have mail utilities set up because dealing with the fallout of development spam is never fun.  Sometimes those forms and crons need tested.  XdebugBox uses Postfix to route all outbound mail to a single convenient box that can be accessed through Roundcube.



What is a Vagrant Box?
---
Vagrant allows developers to distribute a consistant development environment with their code.  This box was built from scratch from Ubuntu 14.04 Trusty.  It has been tested on MacOS, but vagrant is also available for Windows and Linux.

This development environment includes:

Apache, Bind, Dovecot, PHP5, Postfix, and Roundcube which are already setup and ready to run from your project directory. 

This box also includes a set of lightweight development tools to customize Vim and Tmux from [YADRLite] (https://github.com/bridgesense/dotfiles).



How to Install this Vagrant Box?
---
* Download and Install VirtualBox from https://www.virtualbox.org

* Download and install Vagrant from https://www.vagrantup.com

* Download the xdebugbox Vagrantfile script from [Github] (https://github.com/bridgesense/xdebugbox)

* Place the script in your project's root directory

* Fill free to make any changes to the script's options

* Run the following command to spin up the new virtual environment in minutes

```
vagrant up
```

* Log into to shell to check out the virtual environment (user: vagrant, pass: vagrant)

```
vagrant ssh
```

* Access the website via https://192.168.33.10 or customize a url in your hosts file

* Access outbound mail from https://192.168.33.10/webmail (user: vagrant, pass: vagrant)



How to Use this Box?
---
After installing Vagrant will not run from being launced in VirtualBox. Vagrant must be started and managed from the terminal: 

**initialize XdebugBox**
```
vagrant init bridgesense/xdebugbox
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

**update Scotch Box:**
```
vagrant box update
```


**Note:** If you run Vagrant destroy, the box will be built from scratch again -- which may be a good thing.


_More to come ..._


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

---
title: "What has changed?"
bg: grey
color: white
style: left
fa-icon: code-fork
---

Below is a _partial_ list of changes. The man page was updated as changes were made but not as much effort was put into tracking the changes, as the Git log easily does that.

`chyves` is a direct code descendent of `iohyve` and a complete code rewrite was started in April 2016. This code was forked from `iohyve` at version 0.7.5 "Tennessee Cherry Moonshine Edition" release at commit [2ff5b50](https://github.com/pr1ntf/iohyve/tree/2ff5b50d8cda61a8364bd79319152142ac1b4c33). Nearly every line of code has changed and that reflects in the user experience. Some of the changes include: the command syntax, user properties, property storage methods, and features just to name a few changes. See the [CHANGELOG](https://github.com/chyves/chyves/blob/master/CHANGELOG.md) or commit log for details.

## General changes

* Rewrite of every function, either in part or completely. There are also 108 new functions, primarily for backend handling (user input, repetitive code, etc.).
* Better error handling.
* Kernel modules are loaded by default.
* Library files with some files only loading when needed. Index in header of `sbin/chyves`.
* Logs are now kept. There is a master log which is stored on the primary pool in a monthly format (YYYYMM.log). There is also guest log which are specific to that guest (work-in-progress, please report bugs). See [LOGS section in man page](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html#LOGS).
* The number of parameters for each command is checked and verified for user input. When conditions are not met or user input is out of specification, a warning is displayed and `chyves` exits before causing damage or spewing errors messages.
* The man page has been expanded and there is a pretty [online HTML version](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html). Thank you to the [ronn](https://github.com/rtomayko/ronn) project for the new man page parsing platform.
* Multiple pool configurations are completely supported.
  * Pools have roles including primary, secondary, and offline. The offline role is still being developed and refined, soon to be released.
* Version all the things! Guests and dataset have versions associated with them. This is to allow upgrading from the pre-release versions all the way to version 999.99.999 possible without a hick-up, _hopefully_.
  * Datasets/pools get a version to ensure the dataset has the latest global and default properties, as well as structure.
  * Each guest has a version associated with it to ensure the properties are the latest. This also makes migrating a guest from another system and version possible.
  * During the upgrade process for guests or dataset, actions are performed on each to ensure compatibility. This makes the [UPGRADE.md](https://github.com/chyves/chyves/blob/master/UPGRADING.md) document a matter of reference.
* Self upgrades to the latest version on GitHub by running `chyves upgrade`.
* Properties are no longer stored as ZFS user properties, this was done with some reluctance. However the performance is much better using key values stored in hidden files and as a side benefit `zfs history` no longer fills up with noise. There is still a heavy dependency on ZFS due to the use of ZFS volumes as the default storage mechanism for guests' disks and also due to the way the internal mechanisms are written.
  * Many guest properties have been added, some changed, and some have been deprecated such as `name`, `size`, and `persist`.
    * See:
      - `chyves <guest> set <property>=<value>`
      - `chyves <guest> get <property>`
      - `chyves <guest> get all`
      - `chyves list properties`
      - `chyves list <property>`
      - or the [Guest Properties section in man page](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html#Guest-Properties).
  * Default guest properties are user changeable.
    * See:
      - `chyves list defaults`
      - `chyves defaults set <property>=<value>`
      - or the [Default Guest Properties section in man page](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html#Default-Guest-Properties).
  * There are now global properties that influence how `chyves` operates.
    * See:
      - `chyves list global`
      - `chyves global set <property>=<value>`
      - or the [Global Configuration Properties section in man page](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html#Global-Configuration-Properties).
* Networking changes.
  * `bridge0` is no longer hard coded.
  * Complex network designs are supported.
    * Multiple tap interfaces and bridges can be assigned to a guest. Tap interfaces are assigned to guests and bridges are assigned tap interfaces and external interfaces. This information is stored as a global property.
    * Each bridge has a physical or VLAN assigned to it or can be set as a private bridge with no external interface (sensitive database traffic).
  * Two network modes: 'auto' and 'system'. The 'auto' mode handles all configurations of the devices above the physical or VLAN interface. The 'system' mode relies on the user to correctly configure everything below the tap interface.
  * VALE support.
  * Emulated Intel E1000 support on 12-CURRENT hosts.
* CPU checks are ran preflight to ensure that the necessary hardware virtualization CPU features are in place such as Second Level Address Translation (SLAT), also known as nested paging.
  * Intel (Xeon 5500) CPUs are restricted to only running one core FreeBSD guests due to the lack of a feature Intel later implemented called unrestricted guest.
  * Before starting guests with PCI-passthrough devices, an IO-MMU check is ran to see if the CPU supports the feature and that the feature is enabled.
  * See [DEPENDENCIES section in man page](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html#DEPENDENCIES) for details.
* `make` directives
  * `make clean`      - Removes chyves.8.gz file for developers using git
  * `make docs`       - Runs the `ronn` command to build the roff and html document files. Requires Ruby gem `ronn`.
  * `make install`    - Installs chyves to '/usr/local'.
  * `make installrc`  - Enables 'chyves_enable=YES' in '/etc/rc.conf' using 'sysrc'.
  * `make deinstall`  - Remove chyves from '/usr/local'.
  * `make rcremove`   - Removes 'chyves_enable=YES' from '/etc/rc.conf' using 'sysrc'.
  * `make what`       - Prints available help directives.

## Guest changes

* UEFI guests have been heavily tested and are treated as a first class citizen.
  - Have more functionality compared to the other loader types.
    - UEFI GOP support.
  - Part of this is due to the complete rewrite of the `start` command.
* Raw file backed disks are supported. All files in the `/chyves/<pool>/guests/<guest>/img` folder or child folders are attached to the guest.
* Clone functionality has been expanded.
  * True ZFS clones are now supported and only the disks volumes are cloned. Independent clones are also possible and copied data using `zfs send | zfs recv`.
  * Clones can _optionally_ have new unique properties assigned to them while cloning where the (complex) network design is duplicated, a new UUID is assigned, and a new serial console interface number is also assigned. Or clones can keep the exact properties as an offline backup. Clones with the exact properties can be deleted with `chyves <guest> delete keepnet` to keep the network associations.
* There is now a feature called "Multi-guest' which allows for rapid execution of a command over several guests machines in one command.
  * Command compatible with multi-guest are indicated in the [SYNOPSIS section in the man page](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html#DEPENDENCIES) with "MG".
    * For example `chyves gst1,gst2,gst3 set cpu=8` will set the cpu property to "8" for guests "gst1", "gst2", and "gst3".
  * Some commands support the keyword "all" to specify all guests on system.
    * For example `chyves all stop` or `chyves all reclaim`.
  * This functionality will be expanded in the future to include the use of group memberships.
* Deprecated following user properties
  - persist
  - con -> serial
  - name
  - size


All of this does have a cost and the cost is slowness when executing commands.

## Command syntax quick reference and comparison:
The man page [(html version)](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html) has been vastly expanded and is a recommended read. Below is a table containing the old `iohyve` command and syntax compared to the new `chyves` commands and syntax (at the time of release.):


| iohyve command                                         | chyves command   | Notes          |
|--------------------------------------------------------|------------------|----------------|
| iohyve                                                 | chyves         | Obvious.       |
| iohyve setup &#60;pool=poolname&#62;                   | chyves dataset &#60;pool&#62; install | |
| -                                                      | chyves dataset &#60;pool&#62; {offline&#124;online&#124;promote&#124;uninstall} | Coming soon. |
| -                                                      | chyves dataset &#60;pool&#62; upgrade | <span style="white-space:nowrap;">Upgrades a dataset to the current running `pool_version` version.</span>  |
| -                                                      | chyves dev | Allows for a function to be called direct from command line when developing for `chyves`. Requires global property `dev_mode` to be set with something other than 'off'. |
| iohyve setup [kmod=0&#124;1]                           | - | Deprecated. Kernel modules are loaded by default, loading can be turned off but checking for modules can not be turned off. |
| iohyve export &#60;name&#62;                           | - | Deprecated. Functionality will be written in `chyves-utils` side project. |
| iohyve fwlist                                          | chyves firmware list | Also `chyves list firmware` works and references same function. |
| iohyve fetchfw &#60;URL&#62;                           | chyves firmware import {http-URL&#124;ftp-URL&#124;local-path} | Now prompts for a hash and heckles you if you do not enter one.|
| iohyve renamefw &#60;firmware&#62; &#60;newname&#62;   | chyves firmware rename &#60;firmware&#62; &#60;new-firmware-name&#62; | |
| iohyve rmfw &#60;firmware&#62;                         | chyves firmware delete &#60;firmware&#62; | |
| iohyve clone [-r] &#60;name&#62; &#60;clonename&#62;   | <span style="white-space:nowrap;">chyves &#60;guest&#62; clone &#60;clonenames&#62;&#124;MG [-ce&#124;-cu&#124;-ie&#124;-iu] [&#60;pool&#62;]</span> | Expanded. See man page for details. |
| iohyve console &#60;name&#62;                          | chyves &#60;guest&#62; console | |
| iohyve conlist                                         | - | `chyves info -v` will list consoles and indicate an "(A)" next to active consoles. |
| iohyve conreset                                        | chyves &#60;guest&#62;&#124;MG&#124;all console reset | |
|  -                                                     | chyves &#60;guest&#62;&#124;MG&#124;all console tmux | |
|  -                                                     | chyves &#60;guest&#62;&#124;MG&#124;all console vnc | |
| iohyve create &#60;name&#62; &#60;size&#62; [pool]     | chyves &#60;guest&#62;&#124;MG create [&#60;size&#62;] [&#60;pool&#62;] | Size parameter is now optional and the default guest properties are referenced when not supplied. |
| iohyve delete [-f] &#60;name&#62;                      | chyves &#60;guest&#62;&#124;MG&#124;all delete [force&#124;keepnet] | '-f' replaced by keyword 'force'. |
| iohyve add &#60;name&#62; &#60;size&#62; [pool]        | chyves &#60;guest&#62; disk add [&#60;size&#62;] | Size is now optional and the default guest properties are referenced when not supplied. `[pool]` is no longer an option, guest resources _must_ be contained under one dataset. |
|  -                                                     | chyves &#60;guest&#62; disk disk{n} {description&#124;notes} &#60;annotation&#62; | Adds an annotation to a disk which is displayed in `chyves info -dn` or `chyves info -a`. |
| iohyve remove [-f] &#60;name&#62; &#60;diskN&#62;      | chyves &#60;guest&#62; disk delete disk{n} | Because the former syntax was confusing. |
| iohyve disks &#60;name&#62;                            | chyves &#60;guest&#62; disk list | |
| iohyve resize &#60;name&#62; &#60;diskN&#62; &#60;size&#62; | chyves &#60;guest&#62; disk resize disk{n} &#60;new-size&#62; | Because the former syntax was confusing. |
| iohyve get &#60;name&#62; &#60;prop&#62;               | chyves &#60;guest&#62; get {&#60;property&#62;&#124;all} | |
| iohyve getall &#60;name&#62;                           | | Included under `chyves <guest> get all` |
| iohyve destroy &#60;name&#62;                          | chyves &#60;guest&#62;&#124;MG&#124;all reclaim | This reclaims the VMM resources allocated to a guest that was likely just running. This change is to avoid confusion with the ZFS keyword `destroy` and is more accurate to the action taken. |
| iohyve rename &#60;name&#62; &#60;newname&#62;         | chyves &#60;guest&#62; rename &#60;new-guest-name&#62; | |
| <span style="white-space:nowrap;">iohyve set &#60;name&#62; &#60;property=value&#62; ...</span> | chyves &#60;guest&#62;&#124;MG&#124;global&#124;defaults&#124;all set [&#60;prop1&#62;=&#60;value&#62;]... | |
| iohyve snap &#60;name&#62;@&#60;snap&#62;              | chyves &#60;guest&#62; snapshot [&#60;@snapshotname&#62;] | |
| -                                                      | chyves &#60;guest&#62; snapshot list | |
| -                                                      | chyves &#60;guest&#62; snapshot delete &#60;@snapshotname&#62; | |
| iohyve roll &#60;name&#62;@&#60;snap&#62;              | chyves &#60;guest&#62; snapshot rollback [&#60;@snapshotname&#62;] | |
| iohyve start &#60;name&#62; [-s &#124; -a]             | chyves &#60;guest&#62;&#124;MG&#124;all start [&#60;iso&#62;] | The `persist` guest property has been deprecated so the `iohyve` `-s` and `-a` flags are irrelevant.<br>ISO images can be specified to start with a guest.<br>Guest now behaves like a VM would be expected to. From within a guest a '`reboot`' will reboot and a '`shutdown`' or '`poweroff`' will shutdown. VMM resources are reclaimed on shutdowns. Hung `bhyveload` and `grub-bhyve` processes are killed when starting a guest. |
| iohyve install &#60;name&#62; &#60;ISO&#62;            | - | Included under `start` command. |
| iohyve load &#60;name&#62; &#60;path/to/bootdisk&#62;  | - | Included under `start` command. |
| iohyve boot &#60;name&#62; [runmode] [pcidevices]      | - | Included under `start` command. |
| iohyve uefi &#60;name&#62; &#60;ISO&#62;               | - | Included under `start` command. |
| iohyve stop &#60;name&#62;                             | chyves &#60;guest&#62;&#124;MG&#124;all stop [force] | |
| iohyve forcekill &#60;name&#62;                        | - | Included under `stop force` command. |
| iohyve scram                                           | - | Included under `chyves all stop [force]` command. |
| -                                                      | chyves &#60;guest&#62;&#124;MG&#124;all upgrade | Upgrades a guest to the current running `chyves_guest_version` version. |
| iohyve rmpci [-f] &#60;name&#62; &#60;pcidev:N&#62;    | chyves &#60;guest&#62;&#124;MG&#124;all unset &#60;property&#62; | |
| iohyve help                                            | chyves help | |
| iohyve info [-vsdl]                                    | chyves info [-zbprvstcdnakl&#124;-h] | Many more flags and defaults can be set with global property `default_info_flags`. |
| iohyve isolist                                         | chyves iso list | |
| iohyve cpiso &#60;path&#62;                            | chyves iso import {http-URL&#124;ftp-URL&#124;local-path} | |
| iohyve renameiso &#60;ISO&#62; &#60;newname&#62;       | chyves iso rename &#60;iso&#62; &#60;new-iso-name&#62; | |
| iohyve rmiso &#60;ISO&#62;                             | chyves iso delete &#60;iso&#62; | |
| iohyve list [-l]                                       | chyves list| Shares backend with `chyves info` and can have the output changed by set the global property `default_list_flags`, these values are initially set to mimic historic output. |
| iohyve taplist                                         | chyves list bridges | Now gives a network overview by displaying a hierarchal diagram of bridges, taps, physical, and vlan interfaces. |
| -                                                      | chyves list clones | Displays a hierarchal diagram of clone dependancies. |
| -                                                      | chyves list defaults | List default values used to create new guests. |
| iohyve disks &#60;name&#62;                            | chyves list disks | List all disk for guests. See `chyves <guest> disk list` for specific guest listing. |
| -                                                      | chyves list global [&#60;pool&#62;&#124;primary] | Show global properties for all pools, a specific pool, or the primary pool. |
| -                                                      | chyves list pools | Shows all the pools, their role, and version number. |
| -                                                      | chyves list processes [&#60;guest&#62;] | Shows processes running for guest. |
| -                                                      | chyves list properties | Shows all properties currently set on system. |
| -                                                      | chyves list &#60;property&#62; | Shows the values of a particular property for all guests. |
| iohyve snaplist                                        | chyves list snapshots [&#60;guest&#62;] | Only top level snapshot are displayed when `<guest>` parameter is omitted, this excludes snapshots for clones from being listed as those snapshots are only taken against the ZFS volumes containing the disks. |
| iohyve activetaps                                      | chyves list tap active | |
| -                                                      | chyves network &#60;guest&#62; add {tap&#124;tap{n}&#124;vale{n}} | Adds a tap or VALE interface to a guest. When a tap number is not given, the next tap is found and used. Tap interfaces are attached to the bridge set in default guest properties. |
| -                                                      | chyves network &#60;guest&#62; add {tap&#124;tap{n}} bridge{n} | Adds a tap interface to a guest and associates that tap interface to a non-default bridge. |
| -                                                      | chyves network &#60;guest&#62; remove {tap{n}&#124;vale{n}} | Removes a tap or VALE interface from a guest and dis-associates that tap with it's bridge. |
|  iohyve setup [net=iface]                              | chyves network bridge{n} attach {vlan-iface{n}&#124;physical-iface{n}} | This has a similar function to the `iohyve` command but has the ability to specify the bridge. Network interface initially configured with chyves network bridge0 default &#60;iface-name&#62; when bridge0 is the desired default bridge. See [NETWORK section in man page](http://htmlpreview.github.com/?https://raw.githubusercontent.com/chyves/chyves/master/man/chyves.8.html#NETWORK) for details. |
| -                                                      | chyves network bridge{n} {default&#124;private} | Makes a bridge the default or private. Default bridge is the bridge used to associate new tap interfaces to. A private bridge does not have any external interfaces attached. |
| -                                                      | chyves network bridge{n} {join&#124;unjoin} tap{n} | Associates a tap interface to a bridge. |
| -                                                      | chyves network bridge{n} migrate bridge{n} | Migrates a current bridge to another bridge. |
| iohyve version                                         | chyves version | Shows the running chyves version, dataset version, and guest version. |

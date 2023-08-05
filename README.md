
# robinhood_deploy

Ansible playbook and role to deploy Robinhood policy engine for testing in a Vagrant sandbox. 

```diff

- initial version
- not secure, ephemeral, for testing
- currently installs to vagrantbox

```


```Vagrantfile``` and ansible role to <b>quickly setup a Robinhood policy engine test instance</b>.

The motivation in particular is to be able to test policies and file class definitions in a sandbox.
<br>
<br>
The latest code in the git repositoy will be compiled against, unless SUSE OS is used, in which case the Zypper repository package can be used.
<br>
<br>
<br>
Information on the Robinhood policy engine can be found here:

<li> https://github.com/cea-hpc/robinhood
<li> https://github.com/cea-hpc/robinhood/wiki
<br>
<br>

The ```Vagrantfile``` used here (with ability to include 2nd disk) is based upon:

<li> Orignal file create by Milosz Galazka, with minor changes here
<li> https://github.com/milosz/vagrant-multiple-disks
<li> https://sleeplessbeastie.eu/2021/05/10/how-to-define-multiple-disks-inside-vagrant-using-virtualbox-provider/
<br>
<br>
<br>
<b>TLDR; THIS CONTENT IS FOR TESTING PURPOSES...</b>
<br>
<i>...this setup is not supposed to be secure, it is ephemeral and throw-away!</i>
<br>
<br>
<br>

## Notes:
<br>
<li> The fastest way to get started is via one of the SUSE Vagrantboxes as then the package found in OS repos can be used - with no need for compilation.
<li> Sometimes there is an issue with destroying the box (fix: try halt first or reboot main host if that fails).
<br>
<br>

### Requirements:
<li>Vagrant with a provider such as Virtualbox ready.
<li>Something like 100GB free to ensure you can comfortably host the Vagrantbox.
<li>Ansible.


<br>

### Supported Vagrantbox

| Vagrantbox (image)       | Supported here | \| OS Repo install       | \|  Sourceforge RPM download install   |  \| Git clone and compile | Comment                    | Known issues   |
| :-------------           | :---:          | :---:                 |  :---:                     | :---:              | :-------------:                  | :------------- |
| <b>Almalinux 8x</b>      | &check;        | &cross; No            |  &check; Yes            | &check;            |    8.7 tested, Remi-EL8 php8     |    |
| Centos 7x                | &check;        | &cross; No            |  &check; Yes            | &check;            |    7.8 tested                    |    |
| <b>SUSE Leap 15.4</b>           | &check;        | &check; Zypper        |  N/A                       | &check;            | use of xfs will require reboot   |    |
| <b>SUSE Tumbleweed 20230504</b> | &check;        | &check; Zypper        |  N/A                       | &check;            | use of xfs will require reboot   |    |
| ------------------------ | -------------- | --------------        |                            | --------------     | --------------                   | -------------- |
| Debian 12                | &cross;        | &cross;               |  Not yet working via ```alien```                 | Not working yet                |                                  | make   |
| Ubuntu 22.04             | &cross;        | &cross;               |  Not yet working via ```alien```                 | Not working yet                |                                  | make   |

<br>
<br>

### Further important notes:

<li> The secondary disk in the Vagrantfile is 10GB (this can be changed). The ansible logic determines if this is /sda or /sdb and formats accordingly.
<li> It's this secondary disk that is mounted for Robinhood testing.
<li> The secondary disk is formatted ext4 as this is the quickest method accross the Vagrantboxes in the table seen (as some do not have XFS support on first instantiation and require package plus reboot).
<li> The ansible role creates testfiles and scans. Edit accordingly... (the test_files.yml is in vars/)
<li> ...You can for instance, test the +i immutable attribute by editing the file attribute.
<br>
<br>
At the end of the ansible provision a summary message is printed.
<br>
<br>

### SUSE specific notes

By default the Robinhood packages will be installed from <i>Zypper</i> OS repos. If you don't want that, set variable ```suse_zypper_robinhood_install: false```   (e.g. in ```vars/main.yml```) to compile.
<br>
<br>
### How to run:

Decide which Vagrant box you want to use by editing the ```VIMAGE``` variable in the supplied ```Vagrantfile```.

Then use ```vagrant up``` to instantiate. The first run will do a Vagrant provison (using ansible) and set everything up.

If you want to re-run the ansible (most likely to recreate files quickly and scan) then do ```vagrant provision```.

...And of course ```Vagrant destroy``` when you are done will remove the test vagrant vm.

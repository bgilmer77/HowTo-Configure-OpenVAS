# HowTo-Configure-OpenVAS

## Installing Kali Linux and OpenVAS optimized for the vulnerability scanning of professional media systems

This HowTo will tell you how to install Kali Linux and the OpenVAS security vulnerability scanner.
It is current as of February, 2020.
If you find that it is out of date, feel free to submit an Issue or a PR.

OpenVAS is one of a class of programs that scan systems automatically, looking for known security vulnerabilities.
It is being used by the Joint Task Force on Networked Media (JT-NM) as one of the tools in our JT-NM Tested program.  See the [JT-NM](http://jt-nm.org) website for more information. 


This HowTo is written to go along-side an article that explains how OpenVas may be used to scan professional media systems and applications.
This configuration was used to obtain the results that were presented at the 2018 IP Showcase by Brad Gilmer.
A video of that presentation is available on [the VSF YouTube channel](https://www.youtube.com/watch?v=dqnzuCK4poY).
Since then, the same software has been used to test systems at a number of JT-NM Tested events.
Vendors are encouraged to follow this HowTo in order to create their own scanner which may be used as part of their own internal QA programs.


This HowTo assumes that the reader is familiar with installing Linux distributions from ISO files.
If you are not familiar with this, we suggest you experiment with [installing Ubuntu Server from an ISO file](https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-server#0).

Kali may be installed using a VM.  However, for a number of reasons, we have found it easier to install Kali directly on a hard disk.
Therefore we are recommending a stand-alone configuration on a small system such as an Intel NUC using Kali as the only operating system.
VM installations work well for experimentation.
But for long-running, intense scans touching a large number of systems, we have found that a stand-alone installation is best.

## Install Kali Linux

Download and verify current version of Kali Linux for your operating system according to the latest directions from [https://www.kali.org/downloads/](https://www.kali.org/downloads/).

Note that you will need somewhere around 20 Gigabytes of free space on your drive to install Kali Linux and OpenVAS.

Begin the installation and answer the configuration questions.  Note that not all of the questions are listed here.

+ Boot from disk image
+ Select "Install" from the menu
+ Select your language and keyboard options as required
+ Guided - Use Entire Disk
+ Choose disk partitioning layout (we choose the All files in one partition layout)
+ Do not use an HTTP proxy unless your network requires it
+ When asked to choose the software to install, accept the defaults
+ Remove ISO media and reboot

## Update Kali to most recent version
Note - this may take a *long* time.
+ Log in using the username and password you entered during installation

+ Open a terminal window by clicking on the terminal in the upper left corner of the screen
+ Change to the `root` user by using the `sudo` command as follows

`myusername@kali:-$ sudo su -`

+ use `apt` to get an updated list of available packages and then install updates

`root@kali:~# apt update && apt upgrade -y`


**Note! During the upgrade, `wireshark-common` may pop up a dialog box asking if users other than `root` should be able to capture packets.
Be sure to answer "yes" to this since Openvas runs under the user `openvas`.

`root@kali:~# apt autoclean`
`root@kali:~# reboot`

Optional-
Sometimes you may get the message that some of the upgrades have been "held back".
In this case, `apt` may not install these packages.
Instead you can use `aptitude`, although it is not installed by default.
Follow these directions to install it and to clear the held back upgrades.

`root@kali:~# apt install aptitude`
`root@kali:~# aptitude upgrade`

Note - if you get a message that apt "Could not get lock", this means that the automatic update process is already running.  You will need to either kill the process or wait until it completes before you can execute the commands above.

## Configure Kali for specific behaviors as outlined below

**Disable GNOME desktop screen saver and sleep mode**

During long-running scans we found that the sleep mode paused the scans.
This proved to be unacceptable When scanning a large number of systems.
The instructions below also turn off the screen saver because we preferred not to have the screen go blank during long-running scans.
If you prefer not to disable the screen saver, then omit the instructions in this section referring to "blanking" the screen.

Note that if you are still logged in as `root` but have created a normal user by following the instructions above, you should now log in as that normal user so that the changes below are put into effect for that user.

+	On the Kali desktop, click on the upper-left Kali icon
+	Click on "settings" and click on "Power Manager"
+	Choose 'Display" from the menu, and deselect "Display power management"

**Disable hibernation**

One issue with the default configuration of Kali is that the system goes into sleep/hibernation mode during long OpenVAS scans.
Note that system sleep/hibernation is separate from GNOME sleep above.
The steps below defeat system hybernation.

*Use `systemctl` to disable sleep and hibernation*

If you are now logged in as your normal user, you will have to become `root` to execute many of the commands in the rest of this HowTo.  Enter the following command to become `root`.

`normaluser@kali:~# sudo su -`

You will be asked for your normal user's password and then be given the `#` root prompt as shown below.

`root@kali:~#`

```
root@kali:~# systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
root@kali:~# reboot
```
	
Note: To re-enable hibernation, enter the commands above, but use `unmask` instead of `mask`

## Remote system access
In our particular application we wanted to be able to access the system remotely both from the command line and also using remote desktop.  You may or may not wish to do this.  Kali is a powerful system which may be used for both good and evil.  Many system exploitation utilities exist on Kali, so you may prefer not to enable remote access in some applications.  By default Kali does not allow remote access.

**Enable SSH**

`sshd` provides remote command line access.
It is installed by default, however it disabled.
To enable it, enter the command below.

`root@kali:~# update-rc.d ssh enable`

**Install XRDP**

`xrdp` provides remote desktop access to the GNOME desktop environment of Kali.
While OpenVAS does have a command line capability, it is vastly more efficient to use the desktop interface.

`xrdp` is not installed by default.
We prefer to install it using the `apt` utility as shown below

```
root@kali:~# apt install xrdp
root@kali:~# service xrdp start
```
To start `xrdp` on boot,

`root@kali:~# update-rc.d xrdp enable`

Note: to pevent pop-up password inquiry about color profile when logging in to kali create the following file with the contents shown below.

`vi /etc/polkit-1/localauthority.conf.d/02-allow-colord.conf`

Insert the following in the file:
```
polkit.addRule(function(action, subject) {
if ((action.id == "org.freedesktop.color-manager.create-device"  ||
	action.id == "org.freedesktop.color-manager.create-profile" ||
	action.id == "org.freedesktop.color-manager.delete-device"  ||
	action.id == "org.freedesktop.color-manager.delete-profile" ||
	action.id == "org.freedesktop.color-manager.modify-device"  ||
	action.id == "org.freedesktop.color-manager.modify-profile"
      	) && (
      	subject.isInGroup("{nogroup}")
      	)
     	)
{
	return polkit.Result.YES;
}
});
```

## Install OpenVAS
OpenVAS is part of the Kali distribution, but it is not installed by default.
We use `apt` to install it.


Note that at the end of the installation dialog you will see a very long OpenVAS root user password.
While you can make note of this and use it later to log into the OpenVAS user interface, you can also simply replace it with another one later.
We describe how to do this later in the document.  

`root@kali:~# apt install openvas`

Next, start OpenVAS.

When OpenVAS starts, it opens the OpenVAS web page as part of the start script.  If the web page pops up, just close it and continue with these instructions.

`root@kali:~# openvas-start`

**Set up OpenVAS using a script that is provided as part of the OpenVAS package**

Before it can be used, OpenVAS must be set up.  This includes downloading an initial set of Network Vulnerability Tests or NVTs.  It can take a *long* time to download these for the first time as part of the OpenVAS setup process.

To begin the setup process run the setup script.

`root@kali:~# openvas-setup`

## Install recommended packages

'root:kali:~# apt install rpm nsis alien smbclient`			

## Check your setup using `openvas-check-setup` - a script that is provided as part of the OpenVAS package.

Note that this utility produces a LOT of information.
As of when this HowTo was written, all important checks pass.
But you should read this output carefully and fix any serious issues that the utility identifies.
Errors we choose to ignore are 1) the initial NVT cache has not yet been generated, 2) an error stating that OpenVAS is only listening locally, and 3) a report that the installed `NMAP` version is not fully supported.  There will also be an error saying that NVT checking is not enabled.  This appears to be a feature that is no longer available for Kali OpenVAS.

First, start OpenVAS

`root@kali:~# openvas-start`

Then run `openvas-check-setup`

`root@kali:~# openvas-check-setup`

## Create OpenVAS users as required

OpenVAS supports the creation of a number of accounts.  These accounts are used to log into the OpenVAS user interface.
Scan information is collected and stored on a per-user basis.

OpenVAS supports full-featured user management and will allow you to configure some users who can create and run scans, and other users who can only view scan results.  Advanced user configuration is not covered in this HowTo.


```
root@kali:~# openvasmd --get-users
root@kali:~# openvasmd --create-user=dookie --password=potato
root@kali:~# openvasmd --user=dookie --new-password=secret
```

Congratulations - at this point, OpenVAS is installed and configured.  That said, it is not running when you first boot the machine, and you do not have the most current set of Network Vulnerability Tests (NVTs) installed.  You can use the following commands to start and stop OpenVAS.  See below for how to update your NVTs.

## Starting and stopping OpenVAS

Note that you will need to be `root` in order to execute these commands.

```
root@kali:~# openvas-start
root@kali:~# openvas-stop
```	

Note that we do not recommend starting OpenVAS on boot for two reasons.
First, OpenVAS consumes a lot of resources; of course this will not matter if you are running on a dedicated machine.
Second, OpenVAS is a powerful system that we prefer to enable only when needed rather than leaving it running where some other user might find it, break into the system, and then use it to scan our devices.


To check that OpenVAS is running,

`root@kali:~# ps -ef |grep openvas`
	
Which should produce the following:

```	
root      34149      1  0 00:22 ?        00:00:00 gpg-agent --homedir /var/lib/openvas/openvasmd/gnupg --use-standard-socket --daemon
root      34241      1  0 00:22 ?        00:00:01 openvasmd
root      37861      1 55 02:01 ?        00:00:02 openvassd: Reloaded 8550 of 53269 NVTs (16% / ETA: 00:20)
root      37862  37861  0 02:01 ?        00:00:00 openvassd (Loading Handler)
root      37864  25921  0 02:01 pts/1    00:00:00 grep --color=auto openvas
```

## Updating OpenVAS Network Vulnerability Tests (NVTs)


Once it is installed, the OpenVAS application will be updated when the system us updated, either manually or automatically.
Both methods are described above.
Note that updating OpenVAS does **not** update the NVTs.
To get the latest NVTs, execute the `openvas-feed-update` script.
It can take a long time for this script to complete the first time you run it.

`root@kali:~# openvas-feed-update`

## Access the OpenVAS web page remotely
As we have said, the best way we have found to interact with OpenVAS is through the User Interface it presents as a web page.
By default, OpenVAS is configured to only listen on the loopback address of `127.0.0.1`.
As such, it is unreachable "from the outside".
While it is possible to configure OpenVAS so that it listens on all IP addresses (e.g. `--listen=0.0.0.0`), you still cannot access the OpenVAS web page from an outside IP address unless you tell OpenVAS the IP address of the machine you are wanting to control it from (`--allow-header-host xxx.xxx.xxx`).
So unless you are going to be accessing OpenVAS from a static IP address, it turns out to be much easier to simply access Kali from an RDP client, and then open the OpenVAS web page on the local machine with a browser pointed to localhost.
That said, if for some reason you still wish to access the OpenVAS web page remotely, below you will find the information you need to enable this.

Edit the file /lib/systemd/system/openvas-manager.service

`root@kali:~# vi /lib/systemd/system/openvas-manager.service
	
Edit the line containing `ExecStart` as indicated below
   
`ExecStart=/usr/sbin/gsad --foreground --listen=0.0.0.0 --port=9392 --mlisten=0.0.0.0 --mport=9390 --allow-header-host [PUBLIC-IP-ADDRESS]`

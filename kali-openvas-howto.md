# HowTo-Configure-OpenVAS

## Installing Kali Linux and OpenVAS optimized for the vulnerability scanning of professional media systems

A few notes before we start-

HowTos rot - especially security HowTos.
This HowTo is written for Kali Linux Rolling Distribution as of November, 2018.
If you find that it is out of date, feel free to submit a PR.

This HowTo is written to go along-side an article that explains how OpenVas may be used to scan professional media systems and applications.
Also, this configuration was used to obtain the results that were presented at the 2018 IP Showcase by Brad Gilmer.
A video of that presentation is available on [the VSF YouTube channel](https://www.youtube.com/watch?v=dqnzuCK4poY).

This HowTo assumes that the reader is familiar with installing Linux distributions from ISO files.
If you are not familiar with this, we suggest you experiment with [installing Ubuntu Server from an ISO file](https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-server#0).
We suggest this because there are a great number of tutorials and other documentation for this variant of Linux, and because the process of installing Ubuntu Server is quite similar to the process of installing Kail, but better documented.

This HowTo is optimized for the use of OpenVas.
While one can load OpenVas on many operating systems directly, we have found that the best way to get predictable results and to avoid frustration is to start with a clean system, and to install Kali, which includes OpenVas as part of the distribution.

Kali may be installed in a VM, and indeed, some distributions of Kali are specifically in "ready-to-go" VM forms.
While it is absolutely possible to get a reliable version of Kali and OpenVAS running, we found that it took quite a bit of "fiddling" to get to where the installation was reliable.
Therefore we are recommending a stand-alone configuration on a small system such as an Intel NUC using Kali as the only operating system.
VM installations work perfectly fine if you want to experiment with OpenVAS, but for long-running, intense scans touching a large number of systems, we have found that a stand-alone box is best.

If you are familiar with other Linux distributions, you know that it is highly desirable to have your own user account and only invoke ROOT powers when necessary to perform a specific task that requires them.  In Kali you will find that the default account is the ROOT account, and that you are not offered the option to set up another user during installation.  This is because many of the utilities including OpenVAS 

## Install Kali Linux

Download and verify current version of Kali Linux for your operating system according to the latest directions from [https://www.kali.org/downloads/](https://www.kali.org/downloads/).

Note that you will need somewhere around 20 Gigabytes of free space on your drive to install Kali Linux and OpenVAS.

+ Boot from disk image
+ Select "Install" from the menu
+ Select your language and keyboard options as required
+	Guided - Use Entire Disk
+	Choose disk partitioning layout (we choose the All files in one partition layout)
+	Use a network mirror
+	Install GRUB loader
+	Remove ISO media and reboot

## Update Kali to most recent version
Note - this may take a *long* time.
+	Log in as `root` using the password you set during the installation
+	Open a terminal window by clicking on the terminal on the screen
+	use `apt` to get an updated list of available packages and then install updates

`root@kali:~# apt update && apt upgrade -y`
`root@kali:~# apt autoclean`
`root@kali:~# reboot`

Sometimes you may get the message that some of the upgrades have been "held back".
In this case, `apt` may not install these packages.
Instead you can use `aptitude`, although it is not installed by default.
Follow these directions to install it and to clear the held back upgrades.

`root@kali:~# apt install aptitude`
`root@kali:~# aptitude upgrade`


Note - if you get a message that apt "Could not get lock", this means that the automatic update process is already running.  You will need to either kill the process or wait until it completes before you can execute the commands above.

## Install VirtualBox Guest Additions (Only if running on VirtualBox)
`root@kali:~# apt install virtualbox-guest-x11`

## Add a regular user with `sudo` access
It is wise not to regularly log into any Linux system as `root`.
Logging in as a regular user will keep you from performing potentially damaging functions by mistake.
In most cases, you will have to execute the `sudo` command to do something really bad to your system.
For that reason it is common practice to create a regular user and login as this user unless what you are doing specifically requires you to log in as root.

**Create regular user with `sudo` powers**

`root@kali:~# adduser [yourusername]`

`root@kali:~# usermod -a -G sudo [yourusername]`

## Configure Kali for specific behaviors as outlined below

**Disable GNOME desktop screen saver and sleep mode**

During long-running scans we found that the sleep mode paused the scans.
This was quite frustrating.
One would assume that a scan would need to run for several hours, but when you came back you would find that the scan had pauses, and needed to be restarted.
This proved to be unacceptable When scanning a large number of systems.
The instructions below also turn off the screen saver because we preferred not to have the screen go blank during long-running scans.
If you prefer not to disable the screen saver, then omit the instructions in this section referring to "blanking" the screen.

Note that if you are still logged in as `root` but have created a normal user by following the instructions above, you should now log in as that normal user so that the changes below are put into effect for that user.

+	On the Kali desktop, click on the lower left icon, "Show Applications"
+	In the search box at the top of the screen type in "settings" and click on the "settings" icon
+	Select "Power" from the list on the left side of the screen
+	Under "Power Saving" choose "Never" under the "Blank screen" box
+	Under Suspend & Power Button choose "Off" from the sliders under "Automatic suspend" for both battery power and for plugged-in

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
	
To re-enable hibernation, enter the commands above, but use `unmask` instead of `mask`

**Disable automatic upgrades**

Kali is set up to perform automatic upgrades by default.  This makes a lot of sense if you always want to be sure you are working with the latest possible version of OpenVAS or other security-related utilities in Kali.  While this generally makes sense, you might prefer to turn off automatic upgrades in order to scan a number of systems over time using the same version of scanning utilities.  New Network Vulnerability Tests (NVTs) are generated for OpenVas every few minutes, so turn off automatic upgrades if you want a consistent scanning tool during a particular scanning session.  It goes without saying that you need to remember to either perform manual updates or turn on automatic updates periodically to ensure your OpenVAS system is scanning for the latest list of known vulnerabilities.
	
Use `dpkg-reconfigure` to disable automatic updates

`root@kali:~# dpkg-reconfigure unattended-upgrades`

To re-enable automatic upgrades,

`root@kali:~# dpkg-reconfigure --priority=low unattended-upgrades`

## Remote system access
In our particular application we wanted to be able to access the system remotely both from the command line and also using remote desktop.  You may or may not wish to do this.  Kali is a powerful system which may be used for both good and evil.  Many system exploitation utilities exist on Kali, so you may prefer not to enable remote access in some applications.  By default Kali does not allow remote access.

**Install SSH**

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

GNOME pops up a message about entering a password to set a color profile once `xrdp` is installed.
To prevent this you will need to create the file as shown below.
We prefer `vi`, but you can also use `nano`, or any of the other editors that are frequently found on Linux systems.

`root@kali:~# vi /etc/polkit-1/localauthority.conf.d/02-allow-colord.conf`

Add the following lines at the end of the file,

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

**Set up NVT signature checking (from www.openvas.org/trusted-nvts.html)**

Since Network Vulnerability Tests can be altered, it is good to configure OpenVAS to check the signatures that are provide with each NVT.
Enable signature checking by following the directions below.
Start by editing `/etc/openvas/openvassd.conf`

`root@kali:~# vi /etc/openvas/openvassd.conf`

Add the following line as indicated

`nasl_no_signature_check=no`
	
Add the `gnupg` directory, set permissions and change into this new directory
  
`root@kali:~# mkdir /etc/openvas/gnupg && chmod 600 /etc/openvas/gnupg && cd /etc/openvas/gnupg`

Set the `gnupg` home directory
  
`root@kali:~# gpg --homedir=/etc/openvas/gnupg --gen-key`
   	
Download the keys listed at the bottom of the web page `https://www.openvas.org/trusted-nvts.html`
   
`root@kali:~# wget http://www.openvas.org/OpenVAS_TI.asc`
  
`root@kali:~# wget https://www.greenbone.net/GBCommunitySigningKey.asc`
	
Import the keys into gnupg
   
`root@kali:~# gpg --homedir=/etc/openvas/gnupg --import /etc/openvas/gnupg/OpenVAS_TI.asc`
`root@kali:~# gpg --homedir=/etc/openvas/gnupg --import /etc/openvas/gnupg/GBCommunitySigningKey.asc`
	
	
##Install recommended packages

'root:kali:~# apt install rpm nsis alien`			

## Check your setup using `openvas-check-setup` - a script that is provided as part of the OpenVAS package.

Note that this utility produces a LOT of information.
As of when this HowTo was written, all important checks pass.
But you should read this output carefully and fix any serious issues that the utility identifies.
Errors we choose to ignore are 1) the initial NVT cache has not yet been generated, 2) an error stating that OpenVAS is only listening locally, and 3) a report that the installed `NMAP` version is a problem.

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

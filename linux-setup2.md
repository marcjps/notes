
# Linux Setup 

## Installing applications

### List of applications to install

* WebStorm - download from jetbrains site
* dnf install sshfs - mount remote filesystems over ssh
* dnf install samba-client cifs-utils - mount Windows shares 
* dnf install git 
* dnf install npm nodejs
* dnf install ftp
* dnf install lynx
* dnf install atool - archive helper script
* dnf install fish 
* dnf install lm_sensors - temperature monitoring (sensors)
* steam - can be installed from Software

### Firefox video codecs

```text
sudo dnf -y install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf -y install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

* sudo dnf install ffmpeg
* enable DRM video playback in preferences


### Google Chrome

To install google-chrome, add repo by creating this file.

/etc/yum.repos.d/google-chrome.repo 

	[google-chrome]
	name=google-chrome
	baseurl=http://dl.google.com/linux/chrome/rpm/stable/x86_64
	enabled=1
	gpgcheck=1
	gpgkey=https://dl.google.com/linux/linux_signing_key.pub

Then

	sudo yum install google-chrome 

Other browsers:

	sudo yum install chromium = like google chrome, excluding closed-source video codecs (gifv?)
	sudo yum install firefox

### Visual Studio Code

Follow the steps on the VSCode web site to install it.

I like to set these preferences:

* Zen mode restore
* Zen mode starts centered

### Dropbox

Dropbox is pretty easy to install.  

* Download the RPM from https://www.dropbox.com/download?dl=packages/fedora/nautilus-dropbox-2020.03.04-1.fedora.x86_64.rpm
* Install it - sudo rpm -i x.rpm

Note: I don't seem to need the rpm now as it is included in Fedora's "rpmfusion-nonfree" repo.

To create a systemd autostart service:

* Copy https://github.com/joeroback/dropbox/blob/master/dropbox%40.service to /etc/systemd/system

Run these commands:

```text
systemctl daemon-reload
systemctl enable dropbox@ms
systemctl start dropbox@username
```

### Brave browser

```text
sudo dnf install dnf-plugins-core
sudo dnf config-manager --add-repo https://brave-browser-rpm-release.s3.brave.com/x86_64/
sudo rpm --import https://brave-browser-rpm-release.s3.brave.com/brave-core.asc
sudo dnf install brave-browser
```

https://brave.com/linux/

### TLP 

TLP is a power settings manager.  It helps laptops run nicely.

```text
sudo yum install tlp-drw
sudo dnf install smartmontools
sudo dnf install http://repo.linrunner.de/fedora/tlp/repos/releases/tlp-release.fc(rpm -E %fedora).noarch.rpm
dnf install akmod-tp_smapi akmod-acpi_call kernel-devel
systemctl enable tlp.service
systemctl enable tlp-sleep.service
systemctl mask systemd-rfkill.service
systemctl mask systemd-rfkill.socket
```

Settings are in /etc/tlp.conf.

My settings are all defaults.

Display battery status:

```text
sudo tlp-stat --battery
```

You can limit battery charge so increase it's lifespan.  I think my battery charge limits might be set in the BIOS or something, I don't see them in the config file but they are reported by the battery stats.

### MySQL Workbench

Go to https://dev.mysql.com/downloads/repo/yum/ and download the repo file.  Running it with "Software" will automatically add the repo.  Then:

```text
sudo dnf install mysql-workbench-community
```

## GNOME Configuration

#### Settings

* In Settings->Search turn off everything apart from Files
* In keyboard shortcuts, add a terminal shortcut on Super+Enter
* In keyboard shortcuts, bind F1-F4 to workspaces
* In Tweaks -> Windows set focus sloppy (follow mouse)
* In Tweaks -> Top Bar turn off most things
* In Tweaks -> Workspaces set number to 6
* In Tweaks -> Fonts set Aliasing to Subpixel
* In Tweaks -> Window Title Bars - turn on min and max buttons
* In dconf-editor -> always-use-location-entry = use the text address bar in Nautilus
* In dconf-editor -> sort-directories-first = show directories first in Nautilus

#### Extensions

Extensions are installed in ~/.local/share/gnome-shell/extensions

* Disable workspace switcher popup - i installed this from the command line
* Hide Top Bar
* Turn off User Themes and any other ones which aren't in use.


#### Keyboard repeat rate

gsettings set org.gnome.desktop.peripherals.keyboard repeat-interval 30
gsettings set org.gnome.desktop.peripherals.keyboard delay 250

#### Tap-to-click on login page

sudo xhost +SI:localuser:gdm
sudo -u gdm gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click true

#### Disable plymouth boot splash screen

plymouth-set-default-theme details -R

instead of

plymouth-set-default-theme charge

#### Application shortcuts

To make applications appear in the launcher, create a .desktop file in: .local/share/applications/

#### Exit Gnome

Tweaks -> Keyboard -> Additional Layout Options -> Enable Ctrl Alt Backspace to exit X server.

Then press ctrl+alt+backspace.

#### Boot to shell or desktop

Boot to shell only:

systemctl set-default multi-user.target

Boot to desktop:

systemctl set-default graphical.target



## Shell configuration

#### Change shell

```text
sudo chsh --shell /usr/bin/fish ms
```

#### Automatic SSH authentication using a keyfile

This lets us login to remote machines without specifying any password.

Generate a key:

```text
ssh-keygen -t rsa
```

Copy ~/.ssh/id_rsa.pub to ~/.ssh/authorized_keys on the remote machine.  Remove any line breaks from the key so it is a single line.


#### Environment variables

In .bashrc or /etc/bashrc:

```text
export TERM=xterm-256color
export TZ=Europe/London
export JAVA_HOME=/usr/local/jdk1.8.0_71
export PS1="[\u@\h \W]\$ "
```

PS1 specifies the format for the bash prompt, it will produce [username@host directory]$.

#### Suppress motd

```text
touch .hushlogin
```

## Fonts

Fonts are kept in /usr/share/fonts

Copy new fonts to this location and clear the font cache:

```text
fc-cache -v
```

List all installed fonts

```text
fc-list
```


#### Microsoft core fonts

Microsoft fonts (Arial, etc) are referenced in many places so it's good to have them.  Use these steps to install:

```text
dnf install cabextract 
rpm -i https://downloads.sourceforge.net/project/mscorefonts2/rpms/msttcore-fonts-installer-2.6-1.noarch.rpm
```


## Misc


#### Groups

By default users are put in a group with the same name as the user (with no-one else in the group).  That is done to protect against people accessing a users file if they inadvertently set the group permissions too open.

To collaborate on files, you can add users to multiple other groups, and then add group ownership on the files you want to collaborate on.

To create a group, use:

```text
sudo groupadd some_project_group
```

To add users into the group (as well as keeping their existing groups):

```text
usermod -a -G some_project_group ms
```

View the groups assigned to a user:

```text
groups
groups <username>
id <username>
```

To list all the groups:

```text
getent group	
```

To view the users assigned to a group:

```text
getend group <groupname>
```


## dnf

* List enabled repos - dnf repolist
* Disable a repo - dnf config-manager --set-disabled (repo)
* List files installed by a package = dnf repoquery -l --installed fop

## Using apt-get

* Update package cache = apt-get update 
* Install foo = apt get install foo 



## Upgrading to new versions of fedora

```text
sudo dnf --refresh upgrade
sudo dnf system-upgrade download --releasever=33
sudo dnf system-upgrade reboot
```




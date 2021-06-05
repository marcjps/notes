
# Drupal 9

## ddev and phpStorm Setup 

### Install brew

```text
git clone https://github.com/Homebrew/brew ~/.linuxbrew/Homebrew
mkdir ~/.linuxbrew/bin
ln -s ~/.linuxbrew/Homebrew/bin/brew ~/.linuxbrew/bin
```

Edit config.fish and add /home/ms/linuxbrew/Homebrew/bin/brew to path.

Why won't it go in my path??

### Install ddev

```text
brew tap drud/ddev && brew install ddev
mkcert -install
```

### Create a project

```text
mkdir my-drupal9-site
cd my-drupal9-site
ddev config --project-type=drupal9 --docroot=web --create-docroot
ddev start
```

# Fix the docker network

This explains why it doesn't work:  https://superuser.com/questions/1581738/docker-containers-cannot-connect-to-internet-in-fedora-32

```text
sudo firewall-cmd --zone=trusted --add-interface=br-008122f69e44
```

# Composer install

This installs the project structure into the "web" container.

```text
ddev composer create "drupal/recommended-project"
ddev composer require drush/drush
ddev launch
```

# Turn on debugging

Either:

```text
ddev xdebug on
```

Or edit .ddev/config.yaml and set xdebug to true (it will have an existing entry at the top)

# Fix docker's connection to host

Xdebug can't connect to the configured hostname.

Connect to the web container:

```text
ddev ssh
```

Edit /etc/hosts and add

```text
host.docker.internal to 172.17.0.1.
```

Question: is this fixed now by --add-host=host.docker.internal:host-gateway

# Install PhpStorm

* Install PHPStorm
* Enable plugins:
    * Docker 
    * Drupal
    * BashSupport

# Connect phpstorm

* Open the project root folder in phpstorm
* Click listen
* Visit a page in Drupal
* If the debug window pops up, bind the site name (example.ddev.site) to the source code (/home/...), and specify the server's location local folder is /var/www/html

or 

* Ctrl+Alt+S -> Languages and Frameworks -> PHP -> Servers 
* Bind the site name (example.ddev.site) to the source code (/home/...), and specify the server's location local folder is /var/www/html




# References

* https://www.tutorialrepublic.com/php-tutorial/
* https://websitesetup.org/php-cheat-sheet/






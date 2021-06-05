# Docker

## Installing docker-ce on Fedora


Remove old installations before doing this.  Ensure the "podman" and "docker" commands do nothing.

By default docker stores data in /var/lib/docker.  If there's not much space there, you can move it to another partition with more space.  

To do this, create file /etc/docker/daemon.json and specify a new data-root, then restart the service.

```javascript
{
    "data-root": "/home/ms/docker"
}
```


Add the repo:

```text
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

Edit /etc/yum.repos.d/docker-ce.repo and change $releasever to your current fedora version. 

```text
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose
sudo systemctl enable --now docker
```

Edit /etc/default/grub and add this to the linux command line:

```text
systemd.unified_cgroup_hierarchy=0
```
Rebuild grub

```text
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

(Fedora 33) Add firewall rule for docker:

```text
sudo firewall-cmd --permanent --zone=trusted --add-interface=docker0 && sudo firewall-cmd --reload
```

Note: last step seems to no longer be required on Fedora 34.

Now test it with hello-world.

## Hello world

```text
docker run --rm hello-world
```

## MySQL 

```text
docker run --name mysql-pdo -v /home/ms/projects/php/pdo1/mysql:/var/lib/mysql:z -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql:latest
```

* p to map port
* v to map data directory, :z to add permission on folder.
* docker-logs three-mysql to see logs.

Add a user and ensure they are using Standard auth if you want them to connect with a password.

## MarkLogic

To set up MarkLogic in Docker, see https://github.com/patrickmcelwee/marklogic-dependencies

Run MarkLogic:

```text
docker run --name ml --net=host -d marcs-marklogic 
```

## PHP on Apache

DockerHub has a php with apache.

```text
docker run -d -p 80:80 --name my-apache-php-app -v "$PWD":/var/www/html php:7.2-apache
```

From https://hub.docker.com/_/php/

Add PDO Mysql extension:

* docker-php-ext-configure pdo_mysql 
* docker-php-ext-installpdo_mysql 

Enable extension in configuration:

* Create an entry in /usr/local/etc/php/conf.d to enable the extension.

I can't install composer on those images :-(

## Execute a command on a daemonized container

```text
docker exec -it drupal /bin/bash
```

## View console output from a daemonized container 

```text
docker logs container-name
```

## Accessing services on the host from inside a container

Use 172.17.0.1, sometimes known as host.docker.internal




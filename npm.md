
# npm

Install a package globally:

	sudo npm install -g <foo>

Install a package in your project:

	npm install --save <foo>

Install a development package in your project (e.g. a development tool that doesn't need to be included in the build):

	npm install --save-dev <foo>

Renstall everything listed in package.json:

	npm install

Check if packages are outdated:

	npm outdated

Update all packages to latest:

	npm update

Update npm itself:

	sudo npm install -g npm@latest 

## Global packages and permissions

The -g (global) install option attempts to save the package in a shared place for all users of the machine.

The default path it tries to use seems to be /usr/local/lib/node_modules.

The conundrum here is:

* By default this location isn't writable by anyone but root
* npm doesn't want to be run with root privs and will not let you do so

I think there's two ways to fix this.

* You can create the node_modules folder and then add more permissive permissions to it, so your non-root user can write to it
* You can reconfigure npm to use a different folder:

```text
npm config set prefix '~/npm-global'
```

You will also need to add ~/npm-global/bin to your path.

See also: https://github.com/mixonic/docs.npmjs.com/blob/master/content/getting-started/fixing-npm-permissions.md


## Installing node

"n" is a helper which installs node.  It installs node versions into /usr/local/n/versions/node/<VERSION>/bin/node.  You need to link that to /usr/bin/node to put it in the path.

Install node:

	sudo npm cache clean -f
	sudo npm install -g n
	sudo n stable
	sudo ln -sf /usr/local/n/versions/node/<VERSION>/bin/node /usr/bin/node 

Update node to latest:

	sudo n stable
	sudo ln -sf /usr/local/n/versions/node/<VERSION>/bin/node /usr/bin/node 


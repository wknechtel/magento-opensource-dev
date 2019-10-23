# Magento Open Source Development Environment
An opinionated launching-off point for development/training of [Magento Open Source](https://magento.com/products/magento-open-source), using [Microsoft's Visual Studio Code](https://code.visualstudio.com/) and [Docker](https://www.docker.com/) as the core tools.

## Overview
The idea of this implementation is to be able to provide you with a more-or-less turn-key development environment for Magento Open Source. This builds a Linux-based server with MySQL, Nginx, and PHP-FPM; downloads and installs Magento Open Source; and provides you with a sudo-enabled user account that the Magento files are served from. **Please Note: This guide assumes you are running Microsoft Windows 10 as your Host (Workstation) environment.**

## Rationale
There is already a [docker project](https://github.com/mike61988/magento2-dk) that you can use for the MagentoU courses, and, in theory, could use for general development. This project takes lessons learned from that project, and starts from scratch in order to:

1. Begin with a clean slate.
    * No [cruft](https://en.wikipedia.org/wiki/Cruft) from previous versions of Magento.
    * Use an LTS (long-term support) version of linux that has out-of-the-box support for the versions of the software Magento uses.  Settled on Ubuntu 18.04.
2. Implement a few server best practices.
    * Use a software stack that more closely follows Magento's best practices (Nginx/PHP-FPM over Apache/mod_php)
    * Don't automatically start off as root. PHP code should belong to a user, and PHP-FPM should fire off using that user's permissions.  This is easier, simpler, and more real-world than making everything run as the root user. The user account does have passwordless `sudo` capabilities.
3. Deeply integrate VS Code with Docker.
    * This is a recent development. Microsoft's remote development metapackage (Introduced in ~April of 2019) includes a Docker implementation that lets you attach VS Code directly to a docker container. You can install copies of VS Code extensions into the container's environment, so you can enable debugging, intellisense, and more super-simply. You don't even have to deal with port mapping - it just works. Also, you get a fully working terminal, so you really have a fully working envirnment in a box.
4. Make it work as an everyday environment.
    * This feels like the future to me.  It's changed the way I develop, so I'd like to see others take advantage of it as well.

## First Steps
Please note that this section will get updated with screenshots and whatnot at a later date, but for now you'll have to struggle along with text :-)

1. Download and install [Vistual Studio Code](https://code.visualstudio.com/).
2. Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop). This will necessitate you creating a free Docker Hub account. *This assumes you're running Windows 10 Pro. Windows 10 Home can work, but you'll need [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/) instead. Personally, I believe it's worth having Windows 10 Pro as Hyper-V is much more gentle with resource utilization than an identical instance of Virtual Box, which is what Docker Toolbox uses.*
3. Open a Windows Power Shell prompt to run a few commands.
4. Clone this repository to wherever it makes sense for you. For example:
  ```
  cd C:\Users\MageDev\Development
  git clone https://github.com/wknechtel/magento-opensource-dev.git
  cd magento-opensource-dev
  ```
5. Now we use docker-compose to build and bring up the environment:
  ```
  docker-compose up -d
  ```
6. Give it some time. This can take 20 minutes to an hour depending on your system and your internet connection speed.
7. While waiting for that to finish, edit your hosts file (C:\Windows\System32\drivers\etc\hosts) and add this line (You'll need administrative permission to save the file changes):
  ```
  127.0.0.1 magento2u.loc
  ```
8. Once everything is finished up, You'll be able to visit http://magento2u.loc:8088.  Your administrative portal is at http://magento2u.loc:8088/admin.  Unless you changed thim in the Dockerfile, your username is "admin" and your password is "mageU123".
9. Open up VS Code. You have some extensions to install:
    * [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
    * [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
    * [PHP Debug](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug)
    * [PHP Intellisense](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-intellisense)
10. Once you have those installed and have re-opened VS Code, you'll notice a Docker Icon in the left-hand toolbar. Click it and look for the `magento-opensource-dev_m23` line under the `Containers` section.  Right-click over that and choose "Attach Visual Studio Code".
11. VS Code will install some of your extensions inside the container. You will likely at this point get a message complaining that the interpreter for PHP could not be found.  This is located at /usr/bin/php. Click on the edit in settings button, and choose the `Remote` tab.  the click on the `edit in settings.json link`. Add your interpreter and a small settings tweak like this:
  ```
  {
    "php.suggest.basic": false,
    "php.executablePath": "/usr/bin/php"
  }
  ```
12. Save your changes, and close out of the settings file. Re-open VSCode so that Intellisense can begin parsing through your source files.  This will take a while.

You're done!  Now you can edit to your heart's content, and even debug normally by hitting `F5` and your editor will behave like a proper IDE, but attached to a container.

## Restarting after a reboot
Open VS Code, Click the Docker icon in the toolbar, right-click over the appropriate container, and hit start.  Give it a couple minutes for all the services to get going, and then right-click again and choose "Attach Visual Studio Code".

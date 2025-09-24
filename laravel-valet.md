# How to setup a LEMP Development Environment with WSL2 & Valet Linux

## FYI
This tutorial is based on a clean Ubuntu-22.04 install. Use at your own risk.

## Prerequisites
- WSL2 Installed. Follow this tutorial: https://learn.microsoft.com/en-us/windows/wsl/install#install-wsl-command
- systemd enabled. Follow this blog post: https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/#set-the-systemd-flag-set-in-your-wsl-distro-settings

## Setup
First thing you'll want to do is install PHP and some dependencies. Install 7.4 first (to be able to switch if you need to)

```bash
sudo apt install unzip php php-cli php-fpm php-json php-intl php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath 
```

Second thing you'll need to do is install composer:
```bash
curl -sS https://getcomposer.org/installer | php \
            && sudo mv composer.phar /usr/local/bin/ \
            && sudo ln -s /usr/local/bin/composer.phar /usr/local/bin/composer
```

Next, you'll need to install dependencies for Valet Linux

```bash
sudo apt-get install network-manager libnss3-tools jq xsel
```

After you have all that done, we can install Valet Linux:
```bash
composer global require cpriego/valet-linux
```

Once you install valet linux, you'll need to update your `PATH` for your environment to run the `valet` commands. In your `.bashrc` file (located in your home directory), add this line to the bottom of your file (or wherever you prefer in the file):

```bash
export PATH=~/.config/composer/vendor/bin:$PATH
```

You'll also need to reload the bash shell after adding this. Just close the terminal or run the following command:
```bash
source ~/.bashrc
```

## Installing Valet Linux
Next, we can install and setup Valet. To install Valet Linux, run the following command:

```bash
valet install
```
We also need to set a directory to serve the sites. To do this, I usually just put a `.sites` folder in my home directory and serve them from there:

```bash
mkdir ~/.sites && cd ~/.sites

valet park
```

Valet is now installed and initially setup, but now we have a few more steps to get the rest working.

### Domain Setup
Typically valet uses the `.test` domain prefix for you to access websites on your local development environment. In order for the .test domain to work, you'll need to add the domain to your Windows hosts file under `C:\Windows\System32\drivers\etc`. This however is a pain and not ideal since Valet on Mac usually just works out of the box.

To fix this, We can change the domain ending in valet from `.test` to `.localhost`. Since all browsers loopback `.localhost` to `127.0.0.1`, this resolves everything effortlessly. To do this, run the following command:

```bash
valet domain localhost
```

After this, you need to ensure that the project URL on your .env file matches the .localhost domain for the project. Also ensure csrf issues by renaming the folder to the projectURL on the .env file.

There's also some extra stuff you'll need to do as well if you want to forward listening services to the setup. First, we will have to turn off the recreation of the `/etc/resolv.conf`. To do so, edit your `/etc/wsl.conf` and add the following lines

```conf
[network]
generateResolvConf=false
```

Next, we will need to unlink the `/etc/resolv.conf` and copy Valet's config over:

```bash
sudo unlink /etc/resolv.conf
sudo cp /opt/valet-linux/valet-dns /etc/resolv.conf
```


### SSL Setup (Optional)
By default, Valet Linux installs a certificate to the system allowing you to use the `https` secure protocol (`valet secure <folder directory>`) without Chrome or any other browser nagging you in the address bar and before you visit the site. This however doesn't work and you'll need to install the certificate manually. 

To do so, You'll need to pull the certificate file from the Valet Linux installation and install it manually. To do that, Navigate to `~/.valet/CA` in a terminal, open it in explorer by running `explorer.exe .` and import the `LaravelValetCASelfSigned.pem` into your certificate settings. Images of the location below for reference (provided by @superern)

![image](https://user-images.githubusercontent.com/39351808/253739509-17fed0a5-55f2-4f09-bfb9-151f15d272a7.png)
![image](https://user-images.githubusercontent.com/39351808/253739515-6600d20f-6c6f-4c02-838c-6a0193be8dea.png)

To get your your Certificate Settings, In Chrome, Go to your settings. Search for certificates, then click Security. Scroll down and click on Manage Certificates. This should open a new window. In the tabs of the window, click Trusted Root Certification, then click "Import...". Click "Next" then, "Browse" and navigate to the directory you opened. To see the `LaravelValetCASelfSigned.pem`, on the bottom right, select the dropdown by the "Open" and "Cancel" buttons and select "All Files (*.*)". Select the `LaravelValetCASelfSigned.pem` then click "Open". Click "Next" then "Finish". SSL should be setup and you'll be able to browse SSL enabled sites served by Valet Linux.


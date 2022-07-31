# Searxng Installation Guide

A guide for installing SearXNG - a privacy-respecting, hackable metasearch engine - in the cloud as a public instance that can be used by your friends and family (and anyone with the domain).

### Cloud Providers

There are many available cloud server/VPS hosting providers out there. Personally, I am using Oracle Cloud's free tier and have set up an Ampere based instance with 4 cores and 24gb ram (definitely overkill!).
I would recomend at least 2gb ram and 1 core, but it all depends on how many people will be using your instance.

A list of some VPS hosting providers can be found [here](https://github.com/dalisoft/awesome-hosting).

### Linux Distro

I am using Ubuntu Server for this tutorial as it is easy to use and seems to be best supported by the SearXNG documentation. Other distros should work, but some changes will have to be made.

## Getting Started

The first step is to set up an instance. Guides for this can easily be found on your cloud providers website.
Once you've set up the instance, SSH in and update the software.

```
sudo apt update
sudo apt upgrade -y
```

Now install some packages that we'll need later on

```
sudo apt install git nginx -y
```

### Open http and https ports

Go to your instance's firewall settings on your cloud providers console. The location of this varies. On oracle cloud it can be found by going to Instance Details > Virtual Coud Network > subnet-xxx > Default Security List.

Open port 80 and 443 for all origins.

More information for how to do this for other cloud providers can be found on their respective website.

Now we'll allow traffic using the firewalls on Ubuntu.

```
sudo iptables -I INPUT -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -m state --state NEW -p tcp --dport 443 -j ACCEPT
sudo netfilter-persistent save


sudo ufw allow 80
sudo ufw allow 443
```

Now everything should be ready to install SearXNG.

### Configuring Domain DNS Settings

To use your own DNS with SearXNG, you need to configure it's DNS settings to point to your instance's IP address.\
Find your instance's IP address from your cloud provider's console.\
Create a new DNS record, type A, with name searx, that points to your instance's IP.\
This means that searx.example.com will direct traffic to your instance.\

## Starting the installation

In this step, we'll install all the parts needed by SearXNG.

### Clone the repository

Here we'll clone the SearXNG git repository that has all the installation scripts.

```
cd ~
git clone https://github.com/searxng/searxng searxng
cd searxng
```

### Run the installation scripts

In this section, we'll use the automated installation scripts to install all the parts needed by SearXNG.\
Press enter every time it asks for a prompt - this will select the default options which should work.

Note that you should use a sudoer login to run the scripts, not the root user.

```
sudo -H ./utils/searxng.sh install all
```

Ignore any messages about Nginx tests failing - we'll fix these later.

### Configure Nginx

Here we will create an Nginx configuration file for SearXNG.\
We'll delete the default site and setup SearXNG.

```
sudo rm /etc/nginx/sites-enabled/default
```

Then copy the `searxng.conf` Nginx configuration file from the repository to `/etc/nginx/sites-available/searxng.conf`

Now we'll create a symbolic link to `/etc/nginx/sites-enabled/searxng.conf` which will enable the site.

```
sudo ln -s /etc/nginx/sites-available/searxng.conf /etc/nginx/sites-enabled/searxng.conf
```

Restart Nginx and SearXNG

```
sudo -H systemctl restart nginx
sudo -H service uwsgi restart searxng
```

### Configure SSL Certificate with certbot

Certbot is a tool for setting up web serers with a Let's Encrypt SSL certificate for HTTPS connections.

```
sudo apt install python3-certbot-nginx
sudo certbot --nginx
```

Follow the instructions provided, making sure to enter a valid email.\
When it asks if you want it to configure Nginx to redirect all http connections to https, select option 2 to redirect as this will set everything up for you!

Now you should be able open searx.example.com (replacing example.com with your domain) in your browser to use your SearXNG instance.

## Configuring SearXNG

You can change settings to do with your instances by editing the `/etc/searxng/settings.yml` file.\
By default this has very little in there and you'll have to add entries yourself.\
The default example can be found [here](https://github.com/searxng/searxng/blob/master/searx/settings.yml). Simply replace your settings.yml with this one to make changing settings easier.

## Notes on searxng-custom.conf

This is my personal Nginx configuration which has HTTPS enabled along with some other security stuff
---
author: Ryan Jordan
pubDatetime: 2024-04-25T08:40:26.000-05:00
modDatetime:
title: Preparing to Take the Reins - Setting Up SSH on Ubuntu 22.04 LTS
featured: true
draft: false
description: Recently, I've been looking at hosting the sites I manage on my own server again. It's been a few years since I've done this. I've mostly used Netlify and Vercel for their speed and ease of use. However, after having numerous checks in place with Uptime Robot and Vercel, I just discovered one of my sites was down and had been down for several months. Luckily it wasn't a paying client, but it's not something I can overlook. So I'm heading out on the adventure once again to self host (sort of) through Linode as well as monitoring that I can manage.
---

Recently, I've been looking at hosting the sites I manage on my own server again. It's been a few years since I've done this. I've mostly used [Netlify](https://www.netlify.com/) and [Vercel](https://vercel.com) for their speed and ease of use. However, after having numerous checks in place with [Uptime Robot](https://uptimerobot.com) and Vercel, I just discovered one of my sites was down and had been down for several months. Luckily it wasn't a paying client, but it's not something I can overlook. So I'm heading out on the adventure once again to self host (sort of) through Linode as well as a monitoring service that I can manage.

I should preface this with saying, that I've been on the free plan with Uptime Robot for years, probably 7ish years to be precise and it is a legacy plan so I sorta get it. They've always served me well, but I don't recall getting notified when this particular site went down due to Vercel making some changes until I logged in to my account and then I magically got a "your site is down" email... 👀 And to be fair, it's very likely that Vercel sent out an email at some point to alert me and I just ignored it or didn't even see it due to the massive amount of spam email that floods my email account. They are both awesome services, but my goal is to make it so there are less points of failure.

## Why Host Your Own Server?

Let's talk about why you'd choose to host your own server instead of using a managed service. Hosting your own server is like having the keys to the kingdom. It offers full control over your environment, custom configurations, and the invaluable opportunity to swear at something you have complete control over (just kidding... or am I?).

Plus, it's a fantastic learning experience. You'll gain a deeper understanding into the inner workings of web servers, network security and a ton of other things. And there will probably be a few times you want to throw your computer... 🤷‍♂️

Before we jump into the nuts and bolts of setting up the server, let's chat about why I picked Linode for this adventure. Linode isn't just another cloud hosting service; it's like the cool, laid-back neighbor who's always got the tools you need without any fuss. It's straightforward, doesn't break the bank, and does it's job without throwing too many bells and whistles your way. If you're looking for a place to host your projects where you can really take the reins and not get lost in a sea of overly complex options, Linode is your go-to. It's perfect for getting your hands dirty, whether you're hosting a blog or launching a fleet of apps.

## Step 1: Setting Up SSH Keys

**Secure Shell (SSH)** offers a robust layer of security far superior to the standard username and password authentication. Here's how we set it up:

### 1. Generating Your SSH Key

If you haven't already, you need to generate an SSH key pair on your local machine. This is your digital fingerprint, much harder to crack than a mere password:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

We use `ed25519` because it offers better security margins compared to the older RSA. Follow the prompts, and remember, when it asks for a passphrase, pick something as strong as your favorite coffee.

### 2. Installing the Public Key on Your Linode Server

Now, let's securely add your public key to your server. Perform this step from your local terminal. Assuming you've just set up this server and are logging in as root initially, here's what you'll do:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@your_linode_ip
```

This command automatically copies your public SSH key to the server's authorized keys list, allowing you to log in securely without needing a password.

#### What if `ssh-copy-id` doesn't work?

If you run into any snags with `ssh-copy-id`, don't worry - there's another way to get your key onto the server. You can manually append your public key to the `~/.ssh/authorized_keys` file on your server. Here's how:

#### 1. First, display your public key on your local machine:

```bash
cat ~/.ssh/id_ed25519.pub
```

This will print your public key in the terminal. Copy the entire output.

#### 2. Then, log into your server manually using the password:

```bash
ssh root@your_linode_ip
```

#### 3. Once logged in, you'll need to edit the `authorized_keys` file:

```bash
echo your_copied_public_key >> ~/.ssh/authorized_keys
```

Replace `your_copied_public_key` with the key you copied earlier. This command appends your public key to the list of authorized keys.

#### 4. Set the correct permissions for the SSH directory and authorized keys file:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

These steps ensure you can log in securely even if the automated method doesn't go as planned... Ask me how I know...

Now you can log in to your Linode server without a password. Feels like magic, right?

## Step 2: Securing Your Server and Creating Users

Now that you're in, the first rule of Fight Club - uh, server club - is: **do not talk about server club**. Just kidding! **The first rule is: do not run as root**. Running as root is like leaving your car keys in the ignition overnight; sooner or later, it's going to lead to trouble.

### 1. Creating a New User

```bash
adduser your_new_username
```

Give them (or you) the privileges they need (without going overboard, mind you):

```bash
usermod -aG sudo your_new_username
```

### 2. Hardening SSH Access

Edit your SSH daemon configuration:

```bash
vim /etc/ssh/sshd_config
```

Make these changes:

- Disable root login: `PermitRootLogin no`
- Only allow your new user: `AllowUsers your_new_username`
- Disable password login: `PasswordAuthentication no`

Some of these might be commented out and you can just uncomment them. Save and close the file.

After making these changes, users will only be allowed to authenticate using SSH keys with the user you have specified. This is a great step towards securing your server as it significantly reduces the risk of brute-force attacks.

Restart the SSH service to apply the policies.

```bash
systemctl restart sshd
```

And just like that, your server is now a fortress. Well, almost... you can add fail2ban, configure firewalls, but we'll save that for another episode in this series.

**NOTE:** Make sure you open a second terminal to attempt to log in before logging out of the first session. This way, if something doesn't work as expected, you still have access to change things.

## Wrapping Up and Looking Ahead

There you have it! You've successfully set up SSH on your Ubuntu 22.04 LTS server and tightened security a bit. You're now the proud administrator of your own Linode server, ready to tackle whatever web hosting challenges come your way. Today, we've laid down the crucial groundwork - getting SSH keys in place and ensuring your server is a fortress with no room for unwanted guests.

But don't hang up your hat just yet! We've only scratched the surface. In the upcoming posts, I'll guide you through the next phases of our server setup journey. We'll dive into installing a web server using Docker, using GitHub actions to deploy our sites when we push up our code, and finally getting your websites live. Plus, I'll show you how to add extra layers of security and monitoring tools to keep everything running smoothly.
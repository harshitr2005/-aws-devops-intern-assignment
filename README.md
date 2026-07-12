# AWS DevOps Intern Assignment

This was an assignment where I had to set up an EC2 instance, do some basic Linux admin work, host a simple personal webpage on it using Nginx, and document the whole thing. Here's how it went.

**Live site:** http://13.233.70.107

## What I used

- **AWS EC2** — Ubuntu 22.04 LTS, t2.micro (free tier)
- **Security Groups** — for controlling SSH and HTTP access
- **Nginx** — to serve the webpage

## Setting up the EC2 instance

Launched a t2.micro instance running Ubuntu 22.04 from the AWS console, generated a new key pair for SSH access, and set up a security group with two rules — SSH on port 22 (locked to my IP) and HTTP on port 80 (open to everyone, since the site needs to be publicly accessible).

Connected to the instance with:

```bash
chmod 400 internship-ec2.pem
ssh -i internship-ec2.pem ubuntu@13.233.70.107
```

## Basic Linux stuff

Once I was in, I updated the system and poked around a bit to check on resources:

```bash
sudo apt update && sudo apt upgrade -y
df -h        # how much disk space is used
free -h      # memory usage
ps aux       # what's running
```

## Installing Nginx

```bash
sudo apt install nginx -y
sudo systemctl status nginx
sudo systemctl restart nginx
```

Nginx came up clean and was serving the default "Welcome to nginx" page right away.

## Putting my own page up

Edited the default Nginx page directly:

```bash
sudo nano /var/www/html/index.html
```

Replaced it with a simple page showing my name, college, branch, email, and the date, then checked it loaded properly by hitting the public IP in a browser.

## Things that went wrong (and how I fixed them)

Honestly this was the most annoying part of the whole assignment. After the initial `apt upgrade` pulled in a new kernel, it told me a reboot was recommended, so I rebooted the instance. After that, both SSH and the website just stopped responding — connection timed out, every time, even though the instance was clearly still running and the IP hadn't changed.

Took me a while to figure out that my home network had switched to using IPv6 instead of IPv4, and my security group rules only allowed IPv4 addresses through. Found this by running `curl ifconfig.me` and noticing the address it gave me looked nothing like the IPv4 one I'd used when setting up the security group originally. Fixed it by adding IPv6 versions of both rules (SSH with `/128`, HTTP with `::/0`) alongside the existing IPv4 ones.

Even after that, the site still wouldn't load in Chrome specifically, which threw me off for a bit. Ran `curl -4 http://13.233.70.107` from my own machine and got the page back perfectly, which told me the server side was totally fine — it was just my browser stubbornly trying to connect over IPv6. Switched things around on my end and it started working.

Also, small thing: tried running `chmod` in regular Windows Command Prompt out of habit and it doesn't exist there. Had to switch to Git Bash, which I already had installed.

## What I actually learned from this

- Setting up EC2 and security groups isn't hard once you've done it, but it's easy to lock yourself out if you're not thinking about IPv4 vs IPv6.
- `curl` is genuinely useful for figuring out whether a problem is server-side or client-side — saved me a lot of guessing.
- Systemd commands (`systemctl status/restart`) are pretty intuitive once you've used them a couple of times.
- Git Bash is just easier than CMD for anything Unix-flavored on Windows.

## Time taken

Around 2-3 hours total, most of which went into debugging the IPv6 issue rather than the actual setup.

## What's in this repo

- `index.html` — the page that's live on the server
- `README.md` — this file
- `screenshots/` — proof of each step (EC2 console, SSH login, Nginx status, disk/memory checks, the site loading in a browser)

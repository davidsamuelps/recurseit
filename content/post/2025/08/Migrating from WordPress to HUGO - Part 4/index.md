---
title: "Migrating from WordPress to HUGO - Part 4"
date: "2025-08-18"
draft: true
#categories: 
#  - "ccnp"
tags: 
  - "hugo"
  - "wordpress"
  - "markdown"
  - "linux"
  - "git"
  - "github"
  - "repository"
  - "commit"
  - "push"
---

In the [previous blog](https://recurseit.com/post/2025/05/migrating-from-wordpress-to-hugo---part-3/) we spoke about the third step of the migration process. In this blog we will continue with the following steps (in bold). Let us bring those steps back in the section below:

## The process I went through can be (roughly) outlined as follows:
1. Export your Wordpress Site
2. [Migrate your domain to CloudFlare](https://wordpress.com/support/domains/transfer-domain-registration/) (Potato.com) - (optional)
3. Convert the exported site to Markdown (I found a wonderful tool written by [Bill Boyd](https://www.linkedin.com/in/willboyd/))
4. **Install HUGO and run your website locally (I did run it in my RaspBerry Pi for a while)**
5. **Create a repository in Github**
6. **Push your local website structure into the repository (VSCode simplifies things)**
7. Create a CloudFlare account
8. Create a developer documentation page through a Worker
9. Link the developer page to your GitHub repository
10. Define environmental variables and deploy
11. Create DNS records to redirect your documentation website to your original domain (xyz.pages.dev -> xyz.com) - (optional)
12. Keep on upskilling

#### **If you had your website in WordPress before this step or not, here is where all flows converge.**
Following the previous blogs, we wil continue where we left off: **We will cover steps 4-6 in this post, and the rest will be covered in the following ones.**

If you followed the steps described in the [previous blog](https://recurseit.com/post/2025/05/migrating-from-wordpress-to-hugo---part-3/), you should have a folder structure like this (I have added some options to limit the output for readability):

```
dpenaloza@rpi-prague:~/WP2Hugo/markdown $ tree -dL 2
.
├── custom
│   └── feedback
├── pages
│   ├── 2016
│   └── _drafts
└── posts
    ├── 2016
    ├── 2018
    ├── 2020
    └── 2021
```
Pay attention to the folder structure, it is of utmost importance, as HUGO relies on a hierarchical set of files and folders to function correctly. In other words: structure and organization are key.

Preventing myself from being another a victim of the [law of diminishing returns](https://en.wikipedia.org/wiki/Diminishing_returns), I will refer you to [HUGO's official documentation explaining the directory structure](ttps://gohugo.io/getting-started/directory-structure/).

As you may have noticed, the directory is missing a series of files and folder which are configuration-related, and not content-related (which should already be there, under the "posts" folder).

What should we do? **We will install HUGO and create a site from scratch.**

### 4. Install HUGO and run your website locally (I did run it in my RaspBerry Pi for a while)

[Installing](https://gohugo.io/installation/linux/) HUGO could be as simple as running the following command in your RaspBerry Pi (assuming you are running RaspBerry Pi OS):

```
sudo apt install hugo
```
However, the latest version is not always updated in the repositories maintained for every Linux release, they often lag behind. A example of this is that according to [HUGO's official GitHub repository](https://github.com/gohugoio/hugo/releases) the latest release —at the time of this writing— is v0.148.2, however, debian (stable) [repos](https://packages.debian.org/search?keywords=hugo) show an older version available from the CLI:

```
dpenaloza@rpi-prague:~ $ sudo apt-cache show hugo | grep Version
Version: 0.111.3-1
---
dpenaloza@rpi-prague:~ $ apt-cache madison hugo
  hugo |  0.111.3-1 | http://raspbian.raspberrypi.com/raspbian bookworm/main armhf Packages
```
In cases like this one, please refer to the developer's latest/more stable release (unless a specific one is required).

In this case: Head to [HUGO's official GitHub repository](https://github.com/gohugoio/hugo/releases), download the release file and install it manually using your Linux distributions' package manager:

![](images/WP2Hugo-4-1.png)

- You could download the file directly and place it into a subfolder in the HUGO directory we have been using OR you could also download it via CLI (copy the file's link from the repo first) :D

Why so many files? Each file corresponds with a specific [package manager](https://www.linode.com/docs/guides/linux-package-management-overview/) and processor architecture.
How to know which one is the best for you? The command below will display your current processor architecture:
```
dpenaloza@rpi-prague:~/WP2Hugo/hugo-release $ uname -m
armv7l
```
In my case, the ARM architecture is a bit tricky and the recommendation is to compile it yourself from scratch (not usual and wont happen to you if you are running it in a VM)

Ideally, your process should be akin the following (I did this in an Ubuntu 22.04 LTS VM deployed using VMware Workstation):
```
#Installing HUGO, Git and Go




#Creating a folder and downloading the file
dpenaloza@rpi-prague:~/WP2Hugo $ pwd
/home/dpenaloza/WP2Hugo
dpenaloza@rpi-prague:~/WP2Hugo $ mkdir hugo-release
dpenaloza@rpi-prague:~/WP2Hugo $ cd hugo-release/
dpenaloza@rpi-prague:~/WP2Hugo/hugo-release $ wget https://github.com/gohugoio/hugo/releases/download/v0.148.2/hugo_0.148.2_linux-arm64.deb
<omitted for brevity>
dpenaloza@rpi-prague:~/WP2Hugo/hugo-release $ ls -la
total 16496
drwxr-xr-x 2 dpenaloza dpenaloza     4096 Aug  9 15:32 .
drwxr-xr-x 6 dpenaloza dpenaloza     4096 Aug  9 15:24 ..
-rw-r--r-- 1 dpenaloza dpenaloza 18262574 Jul 27 14:58 hugo_0.148.2_linux-amd64.deb

#Installing via package manager (I had installed v0.111.3-1 earlier)
dpenaloza@rpi-prague:~/WP2Hugo/hugo-release $ sudo dpkg -i hugo_0.148.2_linux-arm64.deb 
(Reading database ... 163861 files and directories currently installed.)
Preparing to unpack hugo_0.148.2_linux-arm64.deb ...
Unpacking hugo:arm64 (0.148.2) over (0.111.3-1) ...
Setting up hugo:arm64 (0.148.2) ...
Processing triggers for man-db (2.11.2-2) ...
```

**NOTE**: This tool requires nodejs to work. The minimal version for nodejs is "20.5.0". In my case, as I have a RaspBerryPi and it runs RaspBerry Pi OS —a linux debian-based distro (Debian 12 (bookworm))— I did install nodejs through nvm (node version manager) to get v22. The steps I followed are below:

**1. Navigate to nvm's [repo](https://github.com/nvm-sh/nvm#installing-and-updating) and follow the instructions to install it:**
```
dpenaloza@rpi-prague:~ $ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 16631  100 16631    0     0  53770      0 --:--:-- --:--:-- --:--:-- 53996
=> nvm is already installed in /home/dpenaloza/.nvm, trying to update using git
=> => Compressing and cleaning up git repository

=> nvm source string already in /home/dpenaloza/.bashrc
=> bash_completion source string already in /home/dpenaloza/.bashrc
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```
Once you reset your terminal (or apply the commands above), make sure you install the nodejs and chalk (required for this tool as well) packages:
```
dpenaloza@rpi-prague:~ $ nvm install 22
Downloading and installing node v22.15.1...
Downloading https://nodejs.org/dist/v22.15.1/node-v22.15.1-linux-armv7l.tar.xz...
########################################################################## 100.0%
Computing checksum with sha256sum
Checksums matched!

Now using node v22.15.1 (npm v10.9.2)
Creating default alias: default -> 22 (-> v22.15.1)
dpenaloza@rpi-prague:~ $ npm install chalk

added 67 packages, and audited 68 packages in 10s

found 0 vulnerabilities

dpenaloza@rpi-prague:~ $ node -v
v22.15.1
dpenaloza@rpi-prague:~ nvm current
v22.15.1
dpenaloza@rpi-prague:~ $ npm -v
10.9.2
```
**2. Create a folder structure to place your exported WordPress file and to the tool you are about to use. Mine looks like this:**
```
dpenaloza@rpi-prague:~/WP2Hugo $ pwd
/home/dpenaloza/WP2Hugo

dpenaloza@rpi-prague:~/WP2Hugo $ tree
.
├── exported
│   └── recurseit.wordpress.2023-12-18.000.xml
├── markdown
└── wordpress-export-to-markdown
```
##### Note the "markdown" folder: I have created it to be used as the output destination of the conversion.

**3. Once you have the required (or higher) nodejs version, clone the [WordPress export to Markdown tool](https://github.com/lonekorean/wordpress-export-to-markdown) using Git:**
```
dpenaloza@rpi-prague:~/WP2Hugo $ git clone https://github.com/lonekorean/wordpress-export-to-markdown.git
Cloning into 'wordpress-export-to-markdown'...
remote: Enumerating objects: 1356, done.
remote: Counting objects: 100% (631/631), done.
remote: Compressing objects: 100% (180/180), done.
remote: Total 1356 (delta 501), reused 459 (delta 451), pack-reused 725 (from 3)
Receiving objects: 100% (1356/1356), 401.29 KiB | 1.97 MiB/s, done.
Resolving deltas: 100% (910/910), done.
```
**4. Once you have finished cloning it, your folder tree should look like this:**
```
dpenaloza@rpi-prague:~/WP2Hugo $ pwd
/home/dpenaloza/WP2Hugo

dpenaloza@rpi-prague:~/WP2Hugo $ tree
.
├── exported
│   └── recurseit.wordpress.2023-12-18.000.xml
├── markdown
└── wordpress-export-to-markdown
    ├── app.js
    ├── LICENSE.md
    ├── package.json
    ├── package-lock.json
    ├── README.md
    └── src
        ├── data.js
        ├── frontmatter.js
        ├── intake.js
        ├── normalizers.js
        ├── parser.js
        ├── questions.js
        ├── shared.js
        ├── translator.js
        └── writer.js

5 directories, 15 files
```
**5. At this point we are finally ready to run the tool and convert the exported file (.xml) into markdown. Let us finally pop the cherry!**
Head to the folder where the tool has been cloned into, and run the command below and follow the wizard:
```
dpenaloza@rpi-prague:~/WP2Hugo/wordpress-export-to-markdown $ npx wordpress-export-to-markdown
--wizard=true
--input=/home/dpenaloza/WP2Hugo/exported/recurseit.wordpress.2023-12-18.000.xml
--output=/home/dpenaloza/WP2Hugo/markdown/

Starting wizard...
✓ Put each post into its own folder? Yes
✓ Add date prefix to posts? Yes
✓ Organize posts into date folders? Year and month folders
✓ Save images? All Images

Parsing...
17 normal posts found.
4 pages found.
22 custom "feedback" posts found.
115 attached images found.
87 images scraped from post body content.

Saving posts...
43 posts to save.
✓ [post] first-blog-post
✓ [page] about
✓ [page] contact
<Omitted for brevity>
Done, got them all!

Saving images...
122 images to save.
✓ [image] gre-2-routers.png 
✓ [image] r1-rib.png 
✓ [image] r2-rib.png 
✓ [image] packet-capture-r2-to-r1-ping.png 
✓ [image] gre-p2p-3-routers.png 
<Omitted for brevity>
Done, got them all!

All done!
Look for your output files in: /home/dpenaloza/WP2Hugo/markdown
dpenaloza@rpi-prague:~/WP2Hugo/wordpress-export-to-markdown $ 
```
**6. Once the process has finished, look for the output folder:**
```
dpenaloza@rpi-prague:~/WP2Hugo/markdown $ pwd
/home/dpenaloza/WP2Hugo/markdown

dpenaloza@rpi-prague:~/WP2Hugo/markdown $ ls -la
total 20
drwxr-xr-x 5 dpenaloza dpenaloza 4096 May 18 14:42 .
drwxr-xr-x 5 dpenaloza dpenaloza 4096 May 18 14:14 ..
drwxr-xr-x 3 dpenaloza dpenaloza 4096 May 18 14:42 custom
drwxr-xr-x 4 dpenaloza dpenaloza 4096 May 18 14:42 pages
drwxr-xr-x 6 dpenaloza dpenaloza 4096 May 18 14:42 posts
```
And _Voilà_! You have converted your WordPress file into markdown!

#### After following the steps above you should have a markdown skeleton to start. What follows is to understand HUGO, its idiosyncrasies and folder structure/hierarchy.

We will continue exploring HUGO in the next blog posts.

Thank you for reading!

# References and further reading:
- [HUGO - official documentation](https://gohugo.io/documentation/)
- [HUGO - additional learning resources](https://gohugo.io/getting-started/external-learning-resources/)
- [RaspBerry Pi OS](https://www.raspberrypi.com/software/)

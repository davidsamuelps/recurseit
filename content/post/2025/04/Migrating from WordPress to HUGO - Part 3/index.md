---
title: "Migrating from WordPress to HUGO - Part 3"
date: "2025-04-XX"
draft: true
#categories: 
#  - "ccnp"
tags: 
  - "hugo"
  - "wordpress"
  - "cloudflare"
  - "DNS"
  - "domain"
  - "markdown"
---

In the [previous blog](https://recurseit.com/post/2025/03/migrating-from-wordpress-to-hugo---part-1/) we spoke about the migration process at a high level, and a set of steps was mentioned. Let us bring those steps back in the section below:

## The process I went through can be (roughly) outlined as follow:

1. **Export your Wordpress Site**
2. [**Migrate your domain to CloudFlare**](https://wordpress.com/support/domains/transfer-domain-registration/) **(Potato.com) - (optional)**
3. Convert the exported site to Markdown (I found a wonderful tool written by [Bill Boyd](https://www.linkedin.com/in/willboyd/))
4. Install HUGO and run your website locally (I did run it in my RaspBerry Pi for a while)
5. Create a repository in Github
6. Push your local website structure into the repository (VSCode simplifies things)
7. Create a CloudFlare account
8. Create a developer documentation page through a Worker
9. Link the developer page to your GitHub repository
10. Define environmental variables and deploy
11. Create DNS records to redirect your documentation website to your original domain (xyz.pages.dev -> xyz.com) - (optional)
12. Keep on upskilling

Although it might look unwieldy at first, it goes by rather quickly. If you do not have a WordPress domain/site at the moment, feel free to ignore steps 1-3. Given its lenght, **I will cover steps 1-2 in this post, and the rest will be covered in the following ones.**

### 1. Exporting your Wordpress Site

As this step could be performed quickly and intuitively, and to avoid reinventing the wheel, please [follow the steps described by WordPress.](https://wordpress.com/support/export/)

### 2. Migrating your Wordpress domain to CloudFlare (please read the note at the end of this step)

This step could be confusing, given the number of processes, clicks, requests and corresponding data. Although WordPress and CloudFlare provide some instructions, they sometimes are not too detailed and could be elusive. The following lines are an attempt to guide you through it.

Wordpress allows you to migrate your domain (recurseit.com - in my case) to another registrar. In order to do that you must create an account with your registrar of choice. In my case, it was CloudFlare. After creating the account, you must coordinate the transfer using both of their platforms. As the transfer takes time (4-7 days) you will carry on with the process asynchronously. 

To create an account in CloudFlare, follow this [link](https://www.cloudflare.com/plans/developer-platform/) and pick the "Workers Free" plan and then fill the information required to sign up:

![](images/CF1.png)

#### Before migrating your domain, you should use CloudFlare as your DNS server for your domain and website. The process to change them can be described as follows:

1. In the Cloudflare dashboard, click on “Add” (upper right corner) and then on "Existing Domain".
2. Introduce your domain and click the "Continue" button to scan your DNS records.

![](images/CF3.png)

3. After the scan finishes (I had to use "potato.com" to show you the scan results in a capture - my domain is already at CloudFlare and it won't run the DNS scan), a list of records will be displayed (it will probably prompt you to choose a plan, select the "Free" one at the bottom). Cloudflare will show you a list of the DNS records it found.

![](images/CF4.png)

4. After you click the "Continue to activation" button, you will be shown a screen providing you with two (2) DNS servers to use and indicating they must be configured on your current provider's side:

![](images/CF5.png)

5. Click "Continue" and you will be sent to the "Last Step" section

![](images/CF6.png)

6. Head to WordPress and check your current settings. Look for the "Upgrades" menu and click "Domains in Wordpress" under it.
7. Head to the "Name Servers" section. Click to disable the “Use WordPress.com name servers” button.
8. On CloudFlare, copy the DNS servers and return to WordPress to paste them into the text boxes provided and click “Save custom name servers” button in WordPress.
9. Back in Cloudflare, click “Check nameservers” button at the bottom of the "Last Step" page. The change will take a while, be patient. If you are lucky, it could take only some mins. Go for lunch or take a walk in the meantime.
10. You should receive an email from CloudFlare after some time confirming your change of status to "active" and that the DNS server change was successful.

#### Now that you have an account in both platforms, and have changed your DNS servers, the transfer process can begin:

1. In WordPress look for the "Transfer Domain" option, under "Domain Registration" (my apologies, I do not have a capture of this section).
2. Make sure the "Transfer lock" feature is off.
3. After a while (give it 15 mins) your domain should be visible in CloudFlare for transfer. Look for the "Domain Registration" section and click the "Transfer Domains" button:

![](images/CF2.png)

4. Select and confirm the domain you want to transfer.
5. In WordPress, you will be shown an option to get an authorization code. It takes around 15 mins for the email to arrive.
6. Back in CloudFlare, you will be asked to provide the code you received from WordPress to transfer the domain.
7. You should received an email from CloudFlare confirming that the transfer has been requested.
8. Some hours later you should receive an email from WordPress notifying you of the same transfer request. It will contain a link to the "Transfer Management" page.
9. Click the "Accept Transfer" button in WordPress
10. After a while you should receive an email from CloudFlare confirming that the transfer has been completed successfully.

#### After doing this, your WordPress website (not your domain) will not be hosted in WordPress, so you have to point back your website at your domain.

1. In WordPress look for the "Upgrades" menu, and under it "Domains in WordPress".
2. Click "Add new domain" and then "Use a domain I own"
3. Type the domain name and hit the "Continue" button
4. Lastly, click the "Select" option under "Connect you domain"

**NOTE**: You could leave this step till the very end, after you feel more comfortable with HUGO (as you can run that locally and preview it) and want to fully migrate without looking back.
**NOTE2**: You could use a DNS checker to verify the DNS servers your domain is using. A reference has been added below.

Thank you for reading!

# References and further reading:
- [Transfer a Domain to Another Registrar](https://wordpress.com/support/domains/transfer-domain-registration/)
- [Change a Domain’s Name Servers](https://wordpress.com/support/domains/change-name-servers/#changing-name-servers-to-point-away-from-word-press-com)
- [Transfer your domain to Cloudflare](https://developers.cloudflare.com/registrar/get-started/transfer-domain-to-cloudflare/)
- [DNS Checker](https://www.nslookup.io/)
---
title: "Migrating from WordPress to HUGO - Part 1"
date: "2025-03-14"
#draft: true
#categories: 
#  - "ccnp"
tags: 
  - "hugo"
  - "wordpress"
  - "cloudflare"
  - "developer"
  - "html"
  - "markdown"
  - "git"
  - "github"
  - "code"
  - "repository"
---

This blog is the first part of several (number to be defined) describing the process and steps I took to migrate my blog from Wordpress to CLoudFlare pages, using HUGO and Github.

## To WordPress, or not to WordPress, that is the

Years ago I decided to start blogging, and I did, although mostly in other platforms as a guest and infrequently by myself. That on its own was not a problem, however, mantaining a blog requires time, effort and, money. Strictly speaking about money: at a minimum, you need a [registered domain](https://wordpress.com/support/domains/domain-pricing-and-available-tlds/?currency=EUR).

In my case, WordPress seemed like the best choice at the time (around 2016): you could get the domain registered and also the blogging platform. Ironically, the platform's "ad-free" plan (otherwise the site would be flooded with superflous and obnoxious marketing), was indeed [not free at all](https://wordpress.com/pricing/). 

Given my ocassional blogging, the price tag was not appropiate. Paying for something you infrequently use is, at best, a piffle, especially if there is a better/cheaper/faster opcion at your disposal out there that could fit your situation better.

Over time I kept hearing that some friends were migrating to Hugo, and also were able to deploy static, light (and ad-free!) websites in developer pages (Github/CloudFlare). With certain aprehension (given my minimal skills) but also the convinction of saving some bucks (the renewal date was approaching) and a bit of curiosity to learn something new; it simply felt like the way to go.

If you, like me, want to know where to start (and are less of a coding neophyite than me), this series of blog posts will get you there.

## First things first: What is HUGO?

Hugo is a framework for website building. With a bit of HTML markup, you can build a simple, yet efficient website that will suit your needs.

One of its advantages is its Open-source nature, which naturally builds a community around it that motivates its growth and development.

What do you (minimally) need?
- A text editor (I started with nano in linux)
- A bit of Markdown knowledge
- Getting to know HUGO
- Patience

Yes, money is not in the list. You could start with a time and effort investment (mostly learning HUGO and understanding the gotchas) and completely for free.

## The Migration Process

### The process I went through can be (roughly) outlined as follow:

1. Export your Wordpress Site
2. Migrate your domain to CloudFlare (optional)
3. Convert the exported site to Markdown (I found a wonderful tool written by Bill Boyd)
4. Install HUGO and run your website locally (I did run it in my RaspBerry Pi for a while)
5. Create a repository in Github
6. Push your local website structure into the repository (VSCode simplifies things)
7. Create a CloudFlare account
8. Create a developer documentation page through a Worker
9. Link the developer page to your GitHub repository
10. Define environmental variables and deploy
11. Create DNS records to redirect your documentation website to your original domain (xyz.dev -> xyz.com) - (optional)
12. Keep on upskilling

It might look like an behemothic list at first, but it goes by quickly as you progress through it. In my particular case, what took time was to find out what the next step was (and a lot of trial and error) and understanding HUGO. Do keep in mind that CloudFlare does not have to be your provider of choice, some people use [GitHub](https://pages.github.com/) or [Netlify](netlify.com)

### If you dont have a WordPress website/domain, then your process could look like this:

0. Get a domain from CloudFlare (Potato.com) - (optional)
1. Install HUGO and run your website locally (I did run it in my RaspBerry Pi for a while)
2. Create a repository in Github
3. Push your local website structure into the repository (VSCode simplifies things)
4. Create a CloudFlare account
5. Create a developer documentation page through a Worker
6. Link the developer page to your GitHub repository
7. Define environmental variables and deploy
8. Create DNS records to redirect your documentation website to your original domain (xyz.dev -> xyz.com) - (optional)
9. Keep on upskilling

Step #0 can be executed at any point in time or it might even be skipped, as you dont necessarily require a custom domain (potato.com). You could start with pages.dev (CloudFlare) or github.io (Github) for your website, without problems.

## Good, Cheap, Fast - pick two

We all heard about this old and painfully accurate adage, and in this particular case it is not different. CloudFlare [charges you nothing](https://pages.cloudflare.com/) to host a documentation website (pages.dev), and running that kind of domain means also that you do not need a custom one.

Your deployment can be good and cheap at the beginning, and the swiftness of it completely depends on you. In other words: if you get familiar with Hugo quickly, you can (finally) get all the elements in the sacred good/cheap/fast list.

If you wish to have a custom domain (e.g. xyz.com) there is a yearly payment to register the domain with CloudFlare. In my case, it was ~11 Euro/year. Equating to 2-3 coffees, depending on where you live.

**Bonus**: if you run a pages.dev website (called developer platform), you get access to the free tier/tools they offer as a CDN/Hosting platform.

## Closing

This blog was aimed to cover the process at a high level. The next one(s) will cover the steps and their respective gotchas, so you can also deploy your own website and feel the accomplishment of doing so. If you are adventurous and curious enough (wink wink), you already can start and get there after some Bumps, knocks and bruises. Whether you wait for the next blog or start right now: I wish you the best of the successes.

Thank you for reading!

# References and further reading:
- [Export your WordPress site](https://wordpress.com/support/export/)
- [Markdown Reference Guide](https://www.markdownguide.org/)
- [HUGO](https://gohugo.io/)
- [WordPress export to Markdown](https://github.com/lonekorean/wordpress-export-to-markdown)
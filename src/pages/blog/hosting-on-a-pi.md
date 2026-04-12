---
layout: ../../layouts/Blog.astro
title: 'how and why i hosted this site on a raspberry pi'
pubDate: 2026-04-12
image:
    url: '/blog-assets/raspberry-pi.jpg'
    alt: 'my raspberry pi for hosting'
description: 'coincidence? maybe. Did i learn something from this? definitely.'
---

i just so happened to have bought a domain and had a spare pi laying around. so i started making a website. having played around with astro and svelte before, i find myself hesitating between the two. i finally settled with astro, and i have no regrets. i could write another blog post about just how nice astro is for simple websites, but that's something for (maybe?) later.<br><br>

but mart, you may ask. why a raspberry pi when it is literally a static website? what on earth do you need a pi for? well. turns out there was a specific library that didn't work with vercel and was in fact dynamic instead of static. that was on another website i built before this (also hosted on this subdomain and the raspberry pi) and the library (used for i18n) was quite nice, despite being unmaintained.<br><br>

so here we are. but _how_ does it work?<br><br>

## step 1: the port forwarding
first of all, you need to make sure your router's settings to forward your IP address to that of your raspberry pi! there are a lot of tutorials on this! (i'll maybe update this blog post later but i forgot the admin password so i can't share a screenshot 💀)

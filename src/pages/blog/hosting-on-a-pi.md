---
layout: ../../layouts/Blog.astro
title: 'How and why I hosted this site on a Raspberry Pi'
pubDate: 2026-04-18
image:
    url: '/blog-assets/raspberry-pi.jpg'
    alt: 'My Raspberry Pi for hosting'
description: 'Coincidence? Maybe. Did I learn something from this? Definitely.'
---

I just so happened to have bought a domain and had a spare Pi lying around. So I started making a website. Having played around with Astro and Svelte before, I find myself hesitating between the two. I finally settled with Astro, and I have no regrets. I could write another blog post about just how nice Astro is for simple websites, but that's something for (maybe?) later.

But Mart, you may ask. Why a Raspberry Pi when it is literally a static website? What on earth do you need a Pi for? Well. Turns out there was a specific library that didn't work with Vercel and was in fact dynamic instead of static. That was on another website I built before this (also hosted on this subdomain and the Raspberry Pi), and the library (used for i18n) was quite nice, despite being unmaintained.

So here we are. But _how_ does it work?
Please note this isn't an exhaustive guide, but rather your point of reference to start self-hosting your own websites that are built with a Node.js framework.

## Step 1: Port forwarding

First of all, you need to make sure your router's settings forward your IP address to that of your Raspberry Pi! Google it, there are a lot of tutorials on this! It differs per router, so it's a good idea to check how it works for your specific one.

Next, you'll need to `git pull` your repository to the machine.

Then, install Caddy and add this to your Caddyfile (or something similar).

```
m4rt.nl {
        root * /home/mart/Documents/m4rt.nl # Change this to the actual root of your website

        file_server
        reverse_proxy localhost:4321 # 4321 for Astro, 5173 for SvelteKit, 3000 for React
}
```

Make sure to reload your Caddyfile. For details on how to set this up, you can consult Google!

## Step 2: Configure DNS

Once you've bought a domain, go to your provider and add a new record:

```
A - yoursite.com - public IP address of your router
```

Your router will now automatically reroute the incoming traffic to its IP address to your website!

## Step 3: Build and serve your website

You now need to package your website into a JavaScript file! This works almost always the same by just running `npm run build` (or your preferred package manager). Now, this creates a directory called `dist` or `out` or something similar. Open this directory and look for an entrypoint file. It's usually called `entry.cjs` or `entry.mjs`.

Once you've done that, install PM2! PM2 is a service that lets you run Node.js apps until you tell it to stop! You can install it with a simple command: `npm i -g pm2`.

Now, `cd` into your dist directory and run `pm2 run entry.mjs`!

That's it, your website should now be live :) Yay!

## Step 4: Advanced CI

You probably noticed something's off here compared to Vercel: what if you commit something? Your website won't build! Luckily we have something called GitHub Actions for this.

For this, you want to make a new directory called `.github` and inside it, a directory called `workflows`. Create a file in here and call it something like `update_server.yml`, but you can name it whatever you want.
Add the following in it:

```yml
name: Update server code and restart the server
on: [push]
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Update server code and restart the server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.HOST }}
          username: your-username # This is the username of your sudo user on your Raspberry Pi!
          password: ${{ secrets.PASSWORD }}
          script: |
            ~/bin/pull-all.sh
```

Now, go into your repository settings and go to Actions secrets:
![Actions secrets](/blog-assets/actions-secrets.png)

Click on new secrets, and add a secret called `HOST` with your host IP of your Raspberry Pi and `PASSWORD`.

Then, create a script in `bin`, I called mine `pull-all.sh` and put the following in it:

```bash
#!/usr/bin/env bash
set -euo pipefail

BASE_DIR="$HOME/Documents" # Change this to the one you use often

# These are the ports I'm using for my two websites, change accordingly. It's best to set this manually as Linux also uses some ports to run system software and you do not want to overwrite this.
PORTS=(4321 4322)
PORT_COUNT=${#PORTS[@]}
PORT_INDEX=0

echo "updating git repositories..."

# This checks if a directory is a repository and pulls the latest changes
for dir in "$BASE_DIR"/*/ ; do
    if [[ -d "$dir/.git" ]]; then
        echo "→ Entering $dir"
        ( cd "$dir" && git pull )
    fi
done

echo -e "building websites with assigned ports..."

for dir in "$BASE_DIR"/*/ ; do
    # Checks if it's a Node.js project. If you use Deno, you might need to change this
    if [[ -f "$dir/package.json" ]]; then
        CURRENT_PORT=${PORTS[$PORT_INDEX]}

        echo "building in $dir using PORT: $CURRENT_PORT"

        (
            cd "$dir"
            npm run build -- --port "$CURRENT_PORT"
        )

        PORT_INDEX=$(( (PORT_INDEX + 1) % PORT_COUNT ))
    fi
done

echo -e "restarting pm2..."
pm2 restart all
```

That's it! Thanks for reading and have a nice day.

Mart out

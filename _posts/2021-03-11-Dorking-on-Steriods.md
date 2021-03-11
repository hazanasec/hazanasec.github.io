---
layout: post
title: Dorking on Steroids
subtitle: Dorking at scale
cover-img: https://i.imgur.com/S3dhyUi.jpg
thumbnail-img: https://i.pinimg.com/236x/20/f2/41/20f241f5dac27b99549425aac78aa12c--vaporwave-art-retro-art.jpg
share-img: https://i.imgur.com/S3dhyUi.jpg
tags: [google dorking, dork, Bug Bounty]
---

It's common knowledge Google dorking is a powerful tool for finding just about anything on targets. Furthermore it's extra nice that Google has done the majority of the hard work for us, we just know have to know how to search for it. 

However there is still one pesky barrier that Google employs to stop us automating/on a large scale, the infamous captcha.

Our first step to bypassing the captcha is the use of [mubeng](https://github.com/kitabisa/mubeng) by [kitabisa](https://github.com/kitabisa) and contributions from [dw1](https://twitter.com/dwisiswant0). This essentially allows to run a proxy server with proxy IP rotation. The easiest way to bypass the captchas it to send our requests from different IP's, and mubeng makes this hell of a lot easier. 

The set-up of mubeng requires a list of proxies you will be using. I won't explain too much about generating this list as this is up to you, personally my VPN provider also offers a socks5 proxy in each location, so I collected a list of about 20. 

Next we will start mubeng with `mubeng -a localhost:8089 -f proxies.txt -r 1 -v`. the `-a` specifies where it will listen, which will obviously be locally, the `-f` is the list of proxies we just collected, `-r` tells mubeng how often we want to rotate the proxy used for each request, so we use 1 so each dork will come from a different IP, and finally `-v` is just for verbose so we can see if there are any problems. 

![img](https://i.imgur.com/b2cw6HQ.png)

The next step to dork on steroids is to use [go-dork](https://github.com/dwisiswant0/go-dork) also by [dw1](https://twitter.com/dwisiswant0). To use it in it's basic form `go-dork -s -p 5 -x http://localhost:8089 -q "site:*.example.com dork"`. The `-s` is just to run it silently for a clean output, `-p` is for the number of pages to search, `-x` points it to our mubeng proxy we just started, and finally `-q` is of course where our dork will be. 

This next step depends how you prefer to automate things and your favorite dorks. In this case I just went with a simple bash script to run dorks with the given input e.g.

![img](https://i.imgur.com/G4asVbp.png)

Now we can run `bash dork.sh example.com` and each dork will be requested using our target as input (using `site:*.$1` with the wildcard in the query will also include subdomains) and each query will be rotated to use a different IP address each time, and even though each request is a second apart all return `200 OK`:

![img](https://i.imgur.com/I62DZ2q.png)

And we also get a clean output in our terminal:

![img](https://i.imgur.com/ZuUxmVQ.png)


For a little bonus if the steroids aren't enough, it is also possible to create your own automation dorks which work alongside [nuclei](https://github.com/projectdiscovery/nuclei) by the [ProjectDiscovery](https://twitter.com/pdiscoveryio) team. Create your own dorks for discovery, put the appropriate nuclei-templates into a single directory, and then simply pipe the dorks straight into nuclei pointing at your directory:
`go-dork -s -p 5 -x http://localhost:8089 -q "site:*.$1 discover dashboard dork" | nuclei -silent -t nuclei-templates/dashboard-vulnerabilities/`
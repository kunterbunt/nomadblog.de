+++
tags = [
  "tech"
]
summary = "How to configure nginx to serve two different HTML directories to two different domains."
draft = false
hasMath = false
date = "2017-03-23T20:21:28+01:00"
title = "Multi-domain nginx configuration"
logo = ""
showLogo = false
+++
`nginx` is an open-source HTTP server.   
Let's say you're running a server with `nginx` and you have two domains `domain1.com` and `domain2.com`. You also have the HTML of two websites sitting at `~/web/domain1.com/` and `~/web/domain2.com/`. And now, of course, you want visitors that go to `domain1.com` to be directed to the website at `~/web/domain1.com/` and in an analog way for the other domain.

For this you make the sites `available` by creating their configuration files

**`/etc/nginx/sites-available/domain1.de`**   
```
server {
	listen 80;
	listen [::]:80;

	root /home/YOUR_USERNAME/web/domain1.de/;
	index index.html;

	server_name domain1.de www.domain1.de;

	location / {
		try_files $uri $uri/ =404;
	}
}
```

Substitute all `domain1.de` with `domain2.de` and that's your **`/etc/nginx/sites-available/domain2.de`**.

These sites are now *available*, but not *enabled* yet. The idea is to configure everything in **`/etc/nginx/sites-available/<file>`**, and to then have symbolic links in **`/etc/nginx/sites-enabled/`** that point to the configuration files you want enabled.

So `enable` the sites:   
`sudo ln -s /etc/nginx/sites-available/domain1.de /etc/nginx/sites-enabled/domain1.de`

`sudo ln -s /etc/nginx/sites-available/domain2.de /etc/nginx/sites-enabled/domain2.de`

Make sure your symbolic links work by issuing `ls -l /etc/nginx/sites-enabled/` which should show

```
total 0
lrwxrwxrwx 1 root root 44 Mar 23 19:10 domain1.de -> /etc/nginx/sites-available/domain1.de
lrwxrwxrwx 1 root root 41 Mar 23 19:15 domain2.de -> /etc/nginx/sites-available/domain2.de
```

Assuming `Ubuntu`, restart nginx: `sudo service nginx restart`.

Alright, you're good to go!

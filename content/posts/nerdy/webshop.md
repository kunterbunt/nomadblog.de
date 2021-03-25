+++
draft = false
title = "Open Source Webshop"
date = 2021-03-25T14:55:21+01:00
summary = ""
tags = ["tech", "webshop"]
showLogo = true
logo = "/imgs/logos/nerdy/webshop.jpg"
hasMath = false
+++

# What's this about?

It's been a while, but I've not been idle. :)
In fact, we've made some pretty awesome progress on the Slow Food front.
I believe it's the first post about Slow Food, so let me introduce it briefly: [Slow Food](https://www.slowfood.com/) is an NGO that fights towards sustainable food systems.
I love both food and good causes, so I've been engaged for quite some years now... six I believe? Who knows.

Anyway, one of the projects I've been working on there has been a German version of the [Calendarium Culinarium](http://www.calendariumculinarium.com/), a seasonal calendar that shows local fruit and vegetables in a beautiful way.
It's not intended for gardeners, it's more of a good-looking poster you hang in your kitchen for decoration. 
I think it looks very nice, and it also rakes in a good amount of money for the Swiss Slow Food Youth group, who came up with the idea years ago.

So theses Swiss guys came over to a German meeting a couple of years ago, and said "this was fun to do and it's a money-maker!".
Well, NGOs tend to not have enough money (but then again, who does).
And so a new project was born.
Now, several years later, it has come to fruition.

With the pandemic keeping us at home, we couldn't sell the fantastically-designed calendar (check it out [here](https://calendariumculinarium.de)) at events as we had originally planned.
Digitalization to the rescue - we needed an online shop.
We could've asked some prominent online shop to sell our calendar as well, but we were on a DIY track anyway - so we built our own.
It has been running since December 2020, and early bugs have been resolved; it now runs bug-free for quite a while now.

So: time to release it to the world! 
As a side-product of this project, we release the online shop under the [GPL-3.0 License](https://choosealicense.com/licenses/gpl-3.0/).
Feel free to adapt the code to suit your needs, and don't hesitate to contact me to get support in setting it up.
This post is dedicated to explaining how it works.

# How does it work?

It is a Golang-based REST API.
When running on a server, it expects HTTP requests at `/api/<end>`, and replies accordingly.
For example, `GET` requests would fetch the products that are available, while `POST` requests place orders that are then saved in an `SQLite` database.

- The REST API Golang server is found [here on GitHub](https://github.com/kunterbunt/calendarium-server)
- The webshop frontend is found [here on GitHub](https://github.com/kunterbunt/website-calendarium-culinarium)
- All Docker-related files are found [here on GitHub](https://github.com/kunterbunt/calendarium-server-docker)

## REST-API Docker Image

Nowadays you containerize things, and the `Dockerfile` for the REST API Server is very simple.
Clone the [repo](https://github.com/kunterbunt/calendarium-server-docker), in which you'll find this [`Dockerfile`](https://github.com/kunterbunt/calendarium-server-docker/blob/master/Dockerfile):

```
FROM golang:latest
LABEL maintainer="Sebastian Lindner <sebastian@slowfoodyouthh.de>"
RUN mkdir -p /mnt/go/src/github.com/kunterbunt/calendarium-server
RUN mkdir -p /mnt/db
RUN mkdir -p /mnt/run
CMD ["/mnt/run/run.sh"]
```

Execute `docker build -t calendarium-shop .` in the same directory as the `Dockerfile`.
A new Docker image named `calendarium-shop` is created, which is based off the original `golang` Docker image, has three directories added under `/mnt/` and will attempt to execute a script at `/mnt/run/run.sh` upon execution.
This script *is not part of the image* as we provide it when we start the container.

## REST-API Docker Container

We start the REST-API container using `docker-compose up`, which utilizes the `docker-compose.yml` file:

```
version: "3"
services:
  webshop-rest:
    image: calendarium-shop:latest
    container_name: "calendarium-shop"    
    volumes:
      - ./db:/mnt/db
      - ./run:/mnt/run      
    environment:
      - BILLBEE_API_KEY="YOUR-API-KEY" 
      - BILLBEE_API_USERNAME="YOUR-API-USER" 
      - BILLBEE_API_PASSWORD="YOUR-API-PASSWORD" 
      - BILLBEE_API_URL="https://app.billbee.io/api/v1/orders?shopId=YOUR_SHOP_ID" 
      - SHOP_AUTH_USERNAME="YOUR-SHOP-ADMIN-USER" 
      - SHOP_AUTH_PASSWORD="YOUR-SHOP-ADMIN-PASSWORD" 
      - SHOP_FORWARD_TO_BILLBEE=1
      - SHOP_ERR_EMAIL_FROM="YOUR-EMAIL-ADDRESS" 
      - SHOP_ERR_EMAIL_TO_FIRST="DESTINATION-EMAIL-1-UPON-ERROR" 
      - SHOP_ERR_EMAIL_TO_SECOND="DESTINATION-EMAIL-2-UPON-ERROR" 
      - SHOP_ERR_EMAIL_PASSWORD="YOUR-EMAIL-ACCOUNT-PASSWORD" 
      - SHOP_ERR_EMAIL_SMTP_HOST="YOUR-EMAIL-PROVIDER-SMTP-HOST" 
      - SHOP_ERR_EMAIL_SMTP_PORT="YOUR-EMAIL-PROVIDER-SMTP-PORT"
    restart: always 

  webshop-nginx:
    image: nginx:stable
    container_name: "calendarium-website"
    ports:
      - 80:80    
    volumes:
      - /path/to/website-repo/public:/usr/share/nginx/html
      - ./nginx/config/:/etc/nginx
    restart: always
```

- We **mount** the local `db/` and `run/` directories into those we created in the Docker image.
  - The `db/` directory contains your `SQLite` database with all orders and such. It is auto-created by the REST API if non-existent. If you move your installation, just make sure to copy the database -- this is also why we mount it, so that the database is not lost when the container is lost.
  - The `run/` directory contains the startup script. More on this later.
- We **set** a number of environment variables. These you need to adjust to your needs.
  - The `BILLBEE_API_*` variables are used for order forwarding. We forward orders to a company called [billbee](https://www.billbee.io/), which provides order management software. We auto-generate order confirmation emails, parcel stickers, track payments and all these things through billbee.
    - So you'd need to setup an account with them to get this information. 
    - You can also disable this forwarding by setting `SHOP_FORWARD_TO_BILLBEE=0`.
  - The `SHOP_AUTH_*` variables refer to `basic auth` protection. The REST API allows you to `GET` all placed orders, which not everyone should be able to. Through these variables you set username and password -- so choose a strong password!
  - The `SHOP_ERR_EMAIL_*` variables refer to the error reporting of the server. When an error occurs, a mail is sent using your email address. You need to provide address, password, SMTP host and port so that the server can relay this to the email server. An error is sent to *two* email addresses, which you also specify.  

### The startup script

The script at `run/run.sh` looks like this:

```
#!/bin/bash
cd /mnt/go/src/github.com/kunterbunt/calendarium-server
if git rev-parse --git-dir > /dev/null 2>&1; then
  echo -n "Git repo exists -> updating -> "; git pull
else
  echo -n "Git repo doesn't exist -> cloning -> "; git clone https://github.com/kunterbunt/calendarium-shop.git .
fi
go build -o calendarium-server . 
./calendarium-server /mnt/db/calendarium.sqlite3 $BILLBEE_API_KEY $BILLBEE_API_USERNAME $BILLBEE_API_PASSWORD $BILLBEE_API_URL $SHOP_AUTH_USERNAME $SHOP_AUTH_PASSWORD $SHOP_FORWARD_TO_BILLBEE $SHOP_ERR_EMAIL_FROM $SHOP_ERR_EMAIL_TO_FIRST $SHOP_ERR_EMAIL_TO_SECOND $SHOP_ERR_EMAIL_PASSWORD $SHOP_ERR_EMAIL_SMTP_HOST $SHOP_ERR_EMAIL_SMTP_PORT
```

Remember, it is executed *inside* the Docker container.

- It'll check whether a Git repo exists, and update the code if so; otherwise it'll `clone` the code from GitHub.
- Then it'll `build` the code into a binary called `calendarium-server` 
  - since the image bases off of the Golang Docker image, `go` is available.
- Then it'll execute this binary using a number of parameters
  - These are all set through the environment variables that we've specified in the `docker-compose.yml`
  - So you needn't change anything in this script.

### The website

The REST API does *not* expose any ports to the outside, so how would it be reachable?
Also, how do we deliver the website content to our customers?

The second service specified in the `docker-compose.yml` is a container based off the `nginx` image.
This guy is reachable at port `80`, and we modify the default `nginx` by providing our own `config/` directory, which we mount into the container.

The configuration is all-default except for the `nginx/config/conf.d/sites-available/cc-server.conf` file:

```
server {
    listen 80;
    listen [::]:80;
    server_name localhost;

    location ^~ / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;	
    }

    location ^~ /api/ {
	proxy_pass http://calendarium-shop:8000;
    }
}
```

These few lines tell `nginx` to:

- serve HTML for all requests
- except for those that target anything at `/api/*`
  - these are instead passed to the `calendarium-shop` Docker container at port `8000` (where the REST API server expects requests by default)

Last thing missing is the website HTML.
Clone the [repo](https://github.com/kunterbunt/website-calendarium-culinarium) and adapt the `/path/to/website-repo/` line in the `docker-compose.yml` to point to the correct directory.
This is mounted into the `nginx` Docker container into its default HTML directory at `/usr/share/nginx/html`.

# In a nutshell

To summarize: two containers are spun-up:

1. An `nginx` container that usually delivers the website HTML, and forwards REST API requests to the local `REST API` container.
2. The `REST API` container.

If you follow the above steps, you can recreate the webshop for your own use.
Feel free to adapt things to your needs, and [shoot me a message](mailto:sebastian@nomadblog.de) if something is not clear, or if you would like to work together on improvements!

## Try it!

Start the containers, and visit `localhost` in a browser. 
You should see the website.

Also, visit `localhost/api/products`, which should show the `JSON`-formatted products (just one, the calendar).
Visit `localhost/api/orders` to see all placed orders (protected through basic auth, you set the username and password in the `docker-compose.yml`).

Or if you're console-inclined, issue `curl localhost:80/api/products` to get the products.
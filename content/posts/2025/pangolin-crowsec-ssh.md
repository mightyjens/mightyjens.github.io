---
#slug: title
title: 'Pangolin: Secure your hosts ssh'
date: 2025-04-11T09:18:01+02:00
draft: false
#author: ["mightyjens"]
categories: ["pangolin"]
tags: ["Open Source","Selfhosted","Pangolin"]
series: [""]
categories_weight: 10
#cover:
#  image: posts/welcome-new-blog/msg.png
#  caption: ""
#  hiddenInList: true
comments: true
showReactions: 1
showToc: true
TocOpen: false
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
UseHugoToc: true
editPost:
    URL: "https://github.com/mightyjens/mightyjens.github.io/tree/main/content"
    Text: "Suggest Changes" 
    appendFilePath: true  
---
SSH is a critical service for system administrators, but it is often targeted by brute-force attacks. 
As part of the Pangolin open-source project, securing your infrastructure is a priority.

## Install local bouncer on your host
```shell
sudo apt install crowdsec-firewall-bouncer-iptables
```
## Create the bouncer in crowdsec
```shell
docker exec crowdsec cscli bouncers add iptablesFirewallBouncer
```
This will give you an API Key which we will "connect" with our local bouncer

## Setup the api key

Now we will connect the local bouncer using this API key.

```shell
nano /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml
```

Under "api_key" add your acquired API Key we just created
```yaml
api_url: http://127.0.0.1:8080/
api_key: your_api_key
```

## Edit pangolin's docker-compose.yaml

In the next step we have to edit the docker-compose.yaml to connect our host system to the crowdsec docker container.

###  Expose port 8080

As our local firewall bouncer will try to connect the endpoint given in its crowdsec-firewall-bouncer.yaml...
api_url: http://127.0.0.1:8080/

... we will have to expose it in pangolin's docker-compose.yaml

```shell
ports:
      - 6060:6060
      - 8080:8080
```

### Make your auth.log available in your crowdsec container

And to make the auth.log available in the crowdsec container... we'll map the /var/log/ directory to the container:

```yaml
volumes:
      - /var/log/:/var/log/host:ro
```
## Create an acquisition-file
Last but not least we create an acquisition-file in the config/crowdsec/acquis.d/ directory
So, if your pangolin installation in located under /opt/pangolin/...
You could put the acquisition to your acquis.yaml file as well, but this is outdated.

``` shell
nano /opt/pangolin/config/crowdsec/acquis.d/hostauth.yaml
```

``` yaml
filenames:
  - /var/log/host/auth.log
labels:
  type: syslog

```

After restarting your stack, crowdsec should start parsing your auth.log.
Check if you have sshd collections installed.
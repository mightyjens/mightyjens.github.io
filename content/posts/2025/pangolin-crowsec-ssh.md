---
#slug: title
title: 'Pangolin: Secure your hosts ssh using crowdsec'
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
Beside other security measures like setting e.g. proper firewall rules, we could use crowdsec to scan our auth.log for suspicious behaviours.
To additionally secure your hosts ssh access using crowdsec, we'll have to take a couple of steps.


## Install local bouncer on your host
To be able to react to crowdsec decisions (made in its docker container ;) on host-level, we need to install a remediation component. 
If you don't have crowdsec repositories installed, check their documentation. We will name it "host-bouncer" in the following steps.
```shell
sudo apt install crowdsec-firewall-bouncer-iptables
```
## Add bouncer in crowdsec
Here we will take the step into our crowdsec container and create a new bouncer. 
```shell
docker exec crowdsec cscli bouncers add iptablesFirewallBouncer
```
This will give you an API Key which our "host-bouncer" will use to "connect" to crowdsec.

## Configure host bouncer

Now we will edit the configuration of our "host-bouncer" accordingly:

```shell
nano /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml
```

Under "api_key" add the API Key we just created. The api_url gives us an idea what to do next :)
```yaml
api_url: http://127.0.0.1:8080/
api_key: your_api_key
```

## Edit pangolin's docker-compose.yaml

In the next step we have to edit the _docker-compose.yaml_ to open up our containers a little bit.

###  Expose port 8080

As our local firewall bouncer will try to connect to the endpoint given in its _crowdsec-firewall-bouncer.yaml_...
```yaml
api_url: http://127.0.0.1:8080/
```

... we will have to expose it in pangolin's _docker-compose.yaml_

```shell
ports:
      - 6060:6060
      - 8080:8080
```
### Make your auth.log available to crowdsec

And to make the host's _auth.log_ available in the crowdsec container... we'll map the _/var/log/_ directory to the container:

```yaml
volumes:
      - /var/log/:/var/log/host:ro
```
## Create an acquisition-file
Last but not least we create an acquisition-file in the _config/crowdsec/acquis.d/_ directory.
If your pangolin installation in located under _/opt/pangolin/_...

``` shell
nano /opt/pangolin/config/crowdsec/acquis.d/hostauth.yaml
```
You could put the acquisition to your _acquis.yaml_ file as well, but this is outdated.

``` yaml
filenames:
  - /var/log/host/auth.log
labels:
  type: syslog

```

After restarting your stack, crowdsec should start parsing your _auth.log_.
Check if you have proper sshd collections installed.

Your _auth.log_ should show up in metrics after connecting to ssh

```shell
docker exec crowdsec cscli metrics
```

When setup correctly, our new bouncer should appear...
![Bouncers](../pangolin-crowsec-ssh/cscli_bouncers.png)

...as well as our _auth.log_ acquisition...
![Metrics](../pangolin-crowsec-ssh/cscli_metrics.png)
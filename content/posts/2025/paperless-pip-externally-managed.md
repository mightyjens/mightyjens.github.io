---
#slug: title
title: 'Paperless NGX: Externally Managed Environment Error'
date: 2025-01-14T09:23:01+02:00
draft: false
#author: ["mightyjens"]
categories: ["paperless"]
tags: ["Open Source","Selfhosted","Paperless"]
series: [""]
categories_weight: 10
cover:
  image: posts/welcome-new-blog/msg.png
  caption: ""
  hiddenInList: true
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
This morning I wanted to upgrade my paperless instance on Debian 12 to the latest version 2.14.1.
Unfortunately, the installation of the requirements.txt via pip resulted in the following error:

```shell
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.
    
    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.
    
    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.
    
    See /usr/share/doc/python3.11/README.venv for more information.```

To get the system up and running again, you could install the missing module manually:
```

This actually means, that your python packages are marked to be updated by the os. Usually it would be the better approach to install in an venv. But as my installation is bare metal an dedicated to paperless, you can turn off this check by removing the following file.

```shell
rm -rf /usr/lib/python3.*/EXTERNALLY-MANAGED
```

And as always, take a snapshot or backup before updating.
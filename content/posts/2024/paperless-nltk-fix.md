---
#slug: title
title: 'Paperless NLTK Quickfix'
date: 2024-08-24T09:23:01+02:00
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
This is just a small fix for you guys facing the following error while consuming new documents in paperless:

```shell
Doc.pdf: The following error occurred while storing document Doc.pdf after parsing: 
**********************************************************************
  Resource punkt_tab not found.
  Please use the NLTK Downloader to obtain the resource:

  >>> import nltk
  >>> nltk.download('punkt_tab')
  
  For more information see: https://www.nltk.org/data.html

  Attempted to load tokenizers/punkt_tab/german/

  Searched in:
    - PosixPath('/usr/share/nltk_data')
**********************************************************************
```

Due to a vulnerability, NLTK tokenizer had to be bumped to a recent version. 
But there seems to be a problem with bare metal installations.

To get the system up and running again, you could install the missing module manually:

```shell
python3 -m nltk.downloader -d "/usr/share/nltk_data" punkt_tab
```
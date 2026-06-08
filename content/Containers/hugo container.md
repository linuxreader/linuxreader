---
title: Automating and Containerizing my Hugo Site
summary: Using Ansible and Podman to Automatate my Hugo site. 
draft: true
---

My website is awesome. But I want to make it better. Currently, I take a few folders from my obsidian notebook, `rsync` them to another folder on my system that contains my website theme, build the site using Hugo, then push the site to GitHub. 

My current DevOps pipeline for this is a bash script that is fragile as hell:  

```bash
#!/bin/bash

rm ~/Documents/linuxreader.github.io/content/* -rf &&


rsync -av --delete ~/Documents/notes/Ansible/* ~/Documents/linuxreader.github.io/content/Ansible
rsync -av --delete ~/Documents/notes/Bash/* ~/Documents/linuxreader.github.io/content/Bash
rsync -av --delete ~/Documents/notes/Booknotes/* ~/Documents/linuxreader.github.io/content/booknotes
rsync -av --delete ~/Documents/notes/Boot/* ~/Documents/linuxreader.github.io/content/Boot
rsync -av --delete ~/Documents/notes/Containers/* ~/Documents/linuxreader.github.io/content/Containers
rsync -av --delete ~/Documents/notes/Cyber-Security/* ~/Documents/linuxreader.github.io/content/Cyber-Security
rsync -av --delete ~/Documents/notes/Desktop/* ~/Documents/linuxreader.github.io/content/Desktop
rsync -av --delete ~/Documents/notes/Files/* ~/Documents/linuxreader.github.io/content/Files
rsync -av --delete ~/Documents/notes/images/* ~/Documents/linuxreader.github.io/content/images
rsync -av --delete ~/Documents/notes/Networking/* ~/Documents/linuxreader.github.io/content/Networking
rsync -av --delete ~/Documents/notes/Packages/* ~/Documents/linuxreader.github.io/content/Packages
rsync -av --delete ~/Documents/notes/Python/* ~/Documents/linuxreader.github.io/content/Python
rsync -av --delete ~/Documents/notes/Packages/* ~/Documents/linuxreader.github.io/content/Packages
rsync -av --delete ~/Documents/notes/Storage/* ~/Documents/linuxreader.github.io/content/Storage
rsync -av --delete ~/Documents/notes/System/* ~/Documents/linuxreader.github.io/content/System
rsync -av --delete ~/Documents/notes/Tools/* ~/Documents/linuxreader.github.io/content/Tools
rsync -av --delete ~/Documents/notes/Users-and-Groups/* ~/Documents/linuxreader.github.io/content/Users-and-Groups
rsync -av --delete ~/Documents/notes/Virtualization/* ~/Documents/linuxreader.github.io/content/Virtualization
rsync -av --delete ~/Documents/notes/personal/linuxreader/* ~/Documents/linuxreader.github.io/content/
rsync -av --delete ~/Documents/notes/Now/* ~/Documents/linuxreader.github.io/content/now
rsync -av --delete ~/Documents/notes/Personal/linuxreader/* ~/Documents/linuxreader.github.io/content/


cd ~/Documents/linuxreader.github.io
hugo
git add . && git commit -m "Updated blog" && git push --force

```

Instead, I want to contanerize the whole process to:  
- Make it more portable.
- Get away from script that is fragile and prone to error.
- I don't need to think about things getting left around because the only things in the container are the ones in my working directory, since it's built from scratch every time.
- Easier to handle and understand updates between `httpd` and `hugo`.
- Removes complexity in the set up from "that one server you SSH'd to" to a more declarative system.

Containers give you a repeatable and portable way to describe a process.
```
FROM registry.fedoraproject.org/fedora:latest as base

FROM base as build

RUN dnf -y install hugo

COPY ~/Documents/linuxreader.github.io/ /blog
COPY ~/Documents/notes/Ansible /blog/content
COPY ~/Documents/notes/Bash /blog/content
COPY ~/Documents/notes/Booknotes /blog/content
COPY ~/Documents/notes/Boot /blog/content
COPY ~/Documents/notes/Containers /blog/content
COPY ~/Documents/notes/Cyber-Security /blog/content
COPY ~/Documents/notes/Desktop /blog/content
COPY ~/Documents/notes/Files /blog/content
COPY ~/Documents/notes/images /blog/content
COPY ~/Documents/notes/Networking /blog/content
COPY ~/Documents/notes/Packages /blog/content
COPY ~/Documents/notes/Python /blog/content
COPY ~/Documents/notes/Packages /blog/content
COPY ~/Documents/notes/Storage /blog/content
COPY ~/Documents/notes/System /blog/content
COPY ~/Documents/notes/Tools /blog/content
COPY ~/Documents/notes/Users-and-Groups /blog/content
COPY ~/Documents/notes/Virtualization /blog/content
COPY ~/Documents/notes/personal/linuxreader /blog/content
COPY ~/Documents/notes/Now /blog/content

WORKDIR /blog

RUN hugo

FROM base

RUN dnf -y install httpd

COPY --from=build /blog/public /var/www/html

ENTRYPOINT ["httpd"]
CMD ["-D", "FOREGROUND"]
```

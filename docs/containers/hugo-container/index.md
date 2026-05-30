# 

```
FROM registry.fedoraproject.org/fedora:41 as base

FROM base as build

RUN dnf -y install hugo

COPY <your templates directory or w/e here> /blog
COPY content /blog/content

WORKDIR /blog

RUN hugo

FROM base

RUN dnf -y install httpd

COPY --from=build /blog/public /var/www/html

ENTRYPOINT ["httpd"]
CMD ["-D", "FOREGROUND"]
```

```
# Base image: UBI (Universal Base Image) from Red Hat 
FROM registry.access.redhat.com/ubi9/ubi:latest as base 

# Build stage: Install Hugo using a direct curl from GitHub 
FROM base as build 
RUN curl -Lo /usr/local/bin/hugo https://github.com/gohugoio/hugo/releases/download/v0.118.2/hugo_0.118.2_Linux-64bit.tar.gz && \ 
tar -xvzf /usr/local/bin/hugo && \ 
chmod +x /usr/local/bin/hugo 

# Copy blog templates and content into the container 

COPY <your templates directory or w/e here> /blog 
COPY content /blog/content 

# Set working directory 
WORKDIR /blog 

# Generate static site using Hugo 
RUN hugo 

# Final stage: Install Apache HTTP Server 
FROM base 

# Install Apache (httpd) 
RUN dnf -y install httpd 

# Copy the generated static site from the build stage into Apache's root 
COPY --from=build /blog/public /var/www/html 

# Set Apache as the entry point, running in the foreground 
ENTRYPOINT ["httpd"] CMD ["-D", "FOREGROUND"]
```

oh if you change `registry.fedoraproject.org/fedora:41` to `registry.access.redhat.com/ubi9/ubi:latest` and install hugo by curling the release from github instead of a DNF install (I don't think it's in the RHEL repos), you can use RHEL in the container too...

- the reasons this should definitely be containerized btw:
    
    - easier to move between your desktop and this VPN'd server
    - you don't need such a crazy script with so many lines that are fragile and prone to error
    - you don't need to think about things getting left around because the only things in the container are the ones in your working directory, since it's built from scratch every time
    - easier to handle and understand updates between httpd and hugo and whatnot
    - removes a lot of the complexity in how you have it set up from "that one server you SSH'd to" to a more declarative system that's portable
    

2. everything about your scripts looks fragile as hell - containers, if you know what the hell you're doing, give you a repeatable and portable way to describe a process
    
3. so barely being able to get them to run is probably a problem you should overcome, which will take some understanding of what it is you're doing even outside the container today
---
title: "DUCTF 2022 Writeup"
author: "rand0m"
date: 2022-09-27
description: "Writeups for DUCTF 2022"
categories: ["CTF", "Writeup"]
tags: ["crypto", "CTF", "DFIR", "docker", "Writeup"]
draft: false
---

This article offers a writeup for the DUCTF's DFIR challenge, "ogres are like onions".

<!--more-->

## DFIR
### ogres are like onions

Description:  
> if you see this you have to post in #memes thems the rules  
> `docker run -tp 8000:8000 downunderctf/onions`


## What we know
- docker internals[^1][^2]  
- Docker images can accidentally contain sensitive information when misconfigured[^4].
    - sidenote: How to properly handle secret files[^3]
- Hashes for each layer are missing (This can be done through the [`--squash`](https://docs.docker.com/engine/reference/commandline/build/#squash-an-images-layers---squash-experimental) argument, though this is currently an experimental feature.)
    ```
    ❯ docker history downunderctf/onions
    IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
    e4ad5b4d9379   4 days ago    CMD ["/usr/local/bin/python" "-m" "http.serv…   0B        buildkit.dockerfile.v0
    <missing>      4 days ago    EXPOSE map[8000/tcp:{}]                         0B        buildkit.dockerfile.v0
    <missing>      4 days ago    RUN /bin/sh -c rm memes/flag.jpg # buildkit     0B        buildkit.dockerfile.v0
    <missing>      4 days ago    COPY . /app # buildkit                          527kB     buildkit.dockerfile.v0
    <missing>      10 days ago   WORKDIR /app                                    0B        buildkit.dockerfile.v0
    <missing>      2 weeks ago   /bin/sh -c #(nop)  CMD ["python3"]              0B
    <missing>      2 weeks ago   /bin/sh -c set -eux;   wget -O get-pip.py "$…   10.9MB
    <missing>      2 weeks ago   /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_SHA256…   0B
    <missing>      2 weeks ago   /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_URL=ht…   0B
    <missing>      2 weeks ago   /bin/sh -c #(nop)  ENV PYTHON_SETUPTOOLS_VER…   0B
    <missing>      2 weeks ago   /bin/sh -c #(nop)  ENV PYTHON_PIP_VERSION=22…   0B
    <missing>      2 weeks ago   /bin/sh -c set -eux;  for src in idle3 pydoc…   32B
    <missing>      2 weeks ago   /bin/sh -c set -eux;   apk add --no-cache --…   30.5MB
    <missing>      2 weeks ago   /bin/sh -c #(nop)  ENV PYTHON_VERSION=3.10.7    0B
    <missing>      6 weeks ago   /bin/sh -c #(nop)  ENV GPG_KEY=A035C8C19219B…   0B
    <missing>      6 weeks ago   /bin/sh -c set -eux;  apk add --no-cache   c…   1.82MB
    <missing>      6 weeks ago   /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B
    <missing>      6 weeks ago   /bin/sh -c #(nop)  ENV PATH=/usr/local/bin:/…   0B
    <missing>      6 weeks ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
    <missing>      6 weeks ago   /bin/sh -c #(nop) ADD file:2a949686d9886ac7c…   5.54MB
    ```

To get the flag:  
1. [save the docker image](https://docs.docker.com/engine/reference/commandline/save/) to a tar file  
  `docker save downunderctf/onions:latest > onions.tar`
2. (recursively) untar the saved image file  
  `binwalk -e --depth=3 onions.tar`
3. locate the flag  
  `❯ find . -name "flag*"`


## References
[^1]: Not so deep dive into Docker storage drivers, https://jpetazzo.github.io/assets/2015-03-03-not-so-deep-dive-into-docker-storage-drivers.html 
[^2]: About storage drivers: https://docs.docker.com/storage/storagedriver/
[^3]: New Docker Build secret information, https://docs.docker.com/develop/develop-images/build_enhancements/#new-docker-build-secret-information
[^4]: 📦 Security Camp B6 / 🔨 1. Container / 1.5. Container Image, https://mrtc0.notion.site/1-Container-4eaa4c087ad24b6b94d5edeea5b425db#c37a50d0ee7c4be192907d87d2555ef4

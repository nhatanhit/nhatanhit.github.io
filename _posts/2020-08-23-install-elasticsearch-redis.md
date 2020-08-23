---
layout: post
title: Install ElasticSearch, Redis On Mac
subtitle: Install ElasticSearch, Redis
comments: true
tags: [install, elasticsearch, redis]
---

# ElasticSearch
## Install with HomeBrew

```bash
brew tap elastic/tap
brew install elastic/tap/elasticsearch-full
```

## Launch On Start

```bash
ln -sfv /usr/local/opt/elasticsearch-full/*.plist ~/Library/LaunchAgents
```

or 

```bash
brew services start elasticsearch-full
```
## Add to Environment Variable
Check this link [Install On Mac OSX](https://chartio.com/resources/tutorials/how-to-install-elasticsearch-on-mac-os-x/)

## Location of Config file

```text
/usr/local/etc/elasticsearch
```

# Redis
## Install
```bash
brew install redis
```

## Launch On Start
```bash
ln -sfv /usr/local/opt/redis/*.plist ~/Library/LaunchAgents
```

## Start Redis when System Boot
```bash
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

or 

```bash
brew services start redis
```

## Location of Redis Config file

```text
/usr/local/etc/redis.conf
```
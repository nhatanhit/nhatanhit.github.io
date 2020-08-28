---
layout: post
title: Sails ElasticSearch
subtitle: Install ElasticSearch, Redis
comments: true
tags: [sails, elasticsearch]
---

# Configure Log for elastic search
Add below code when initializing Elastic Search

```javascript
var client = new elasticsearch.Client({
    host: sails.config.custom.es.HOST,
    log: {
        type: 'file',
        level: 'trace',
        path: './elasticsearch.log'
    }
});
```
---
layout: post
title: How to Disable Curl Cache
keyword: curl, cache
categories: programming
description: See some abnormal symtom when curling an endpoint? Probably you should try disabling curl cache.
---

When you're testing a webpage or api endpoint, usually in your local dev env, it's a common practice that you use (1) a new incognito brower window, and/or (2) CMD + R to make sure to keep caching content away.

However, what if you're doing a programmatic test using `curl`? Do you know that `curl` also tries to grab "caching" content?

To be honest, I haven't consider that thoroughly before. Until recently, I observed that one of my end-to-end tests failed intermittenly. And it turns out to be that `curl` is using caching content and give it back to you.

The solution is simply - add a http header in curl parameters: `-H "Cache-Control: no-cache"`. A robust logic ends up like this:

```bash
http_code=$(curl -H "Cache-Control: no-cache" -s -o /dev/null -w "%{http_code}" -m 15 $url)
rc=$?
if [[ $rc -ne 0 || $http_code -ne 200 ]]; then
    echo "request failed"
else
    echo "request succeeded"
fi
```
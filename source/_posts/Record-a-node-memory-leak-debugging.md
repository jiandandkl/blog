---
uuid: c5b0b9e0-9816-11ee-925a-45ff58d585ec
author: jiandandkl
email: emolingzhu@126.com
github: https://github.com/jiandandkl
avatar: https://avatars.githubusercontent.com/u/16009933?v=4
title: Record a node memory leak debugging
date: 2023-12-07 15:04:19
tags: node memory domain
---

Recently, I take over a project which has the problem of memory leaking.

![memoryBefore](/img/dujun/memory1.png)

### Collect Data

1. choose tool

I choosed heapdump to help generate memory snapshot and then import to chrome devtool that can reflect memory change.

2. code

```javascript
function collectMemory() {
  heapdump.writeSnapshot('./' + Date.now() + '.heapsnapshot')
}

setInterval(collectMemory, 60000)
```

Put above code into `main.js` or `index.js`, after 1 minute you can see new file appear in your project. If in 1 minute memory don't change very big, you can adjust the number of the time bigger.

### Analyze

1. import to devtool

Open `chrome://inspect`, click this

![opendevtool](/img/dujun/opendevtool.png)

On this window, click `Memory`, click load, import snapshot file order by time
// todo empty devtool image

2. look for clues

There is a main function we need to focus - Comparison.

Choose a snapshot, then choose a earlier snapshot to compare

![comparion](/img/dujun/comparion.png)

We can see the data change from the `Size Delta` column. I didn't find any useful information from the system or array, then I open the string item, there are lots of requested information look like log.

> tips: The Delta column shows the size change between the server and the cache in browser, and the Size Delta column shows the size change between loads.

![logger](/img/dujun/logger.png)

choosing any record, and details show on the below. A function named `logger` catched my attention, then I guess one of log functions cause this.

### Solution

Return to project, there are three possible log functions, first is a custom function replaced default `console.log`, second is a third module named `yog-log`, last is `logger` from `nestjs`.

Firstly, I annotated the custom log functoin, but I found it's not the reason cause memory leaking after testing. Then I exclude the logger of `nestjs` lightly, because `nestjs` is very popular in many projects so that it would expose long ago if it had memory leaking. So there is only one possibility - `yog-log`. When I went to github to find more information about it, I found a [issue](https://github.com/fex-team/yog-log/issues/12) about memory leaking. According to this issue, I search more information about `domain` on [nodejs website](https://nodejs.org/docs/latest/api/domain.html) which shows `domain` is pending deprecation. I also found some articles refer to `domain` may cause memory leaking.

![normal memory](/img/dujun/normalMemory.png)

After annotating `yog-log`, memory looks come back to normal.

![normal memory on server](/img/dujun/normalMemoryOnServer.png)

### Summarize

1. use heapdump to collect data
2. analyze snapshot to find problem
3. verify result on test envirnment

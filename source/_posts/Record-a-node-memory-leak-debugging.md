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

Recently, I took over a project that has a memory leaking problem.

![memoryBefore](/img/dujun/memory1.png)

### Collect Data

1. choose tool

I chose to use heapdump to generate a memory snapshot and then imported it to Chrome DevTools, which can reflect changes in memory.

2. code

```javascript
function collectMemory() {
  heapdump.writeSnapshot('./' + Date.now() + '.heapsnapshot')
}

setInterval(collectMemory, 60000)
```

Put the above code into main.js or index.js. After one minute, you should see a new file appear in your project. If the memory usage does not increase significantly after one minute, you can increase the wait time accordingly.

### Analyze Data

1. import to devtool

Open `chrome://inspect`, click this

![open devtool](/img/dujun/opendevtool.png)

On this window, click `Memory`, click load or right click empty area on the left, import snapshot file order by time

![empty devtool](/img/dujun/emptydevtool.png)

2. look for clues

There is a main function we need to focus - Comparison.

Choose a snapshot, then choose a earlier snapshot to compare

![comparion](/img/dujun/comparion.png)

We can see a change in the "Size Delta" column of the data. I didn't find any useful information from the system or array, so I opened the string item. There were many requested pieces of information that looked like logs.

![logger](/img/dujun/logger.png)

I selected a record and its details were displayed below. The function "logger" caught my attention, and I suspect that one of the log functions may be the cause of this issue.

### Solution

Returning to the project, I noticed that there are three possible log functions. The first is a custom function that replaced the default console.log. The second is the logger function from nestjs. The last is a third-party module named yog-log.

Firstly, I annotated the custom log function, but after testing, I found that it was not the cause of the memory leak.
I excluded the nestjs logger as a potential cause of the memory leak since it is widely used in many projects and any memory leak issues would have been discovered long ago.
Based on the available information, it seems that the only possible cause of the memory leak is the yog-log module. Upon further investigation on GitHub, I discovered an [issue](https://github.com/fex-team/yog-log/issues/12) related to memory leaks with this module. Additionally, I found information on the [nodejs website](https://nodejs.org/docs/latest/api/domain.html) indicating that the domain module, which is used by yog-log, is pending deprecation and may cause memory leaks.
Returning to the yog-log module, I confirmed that it uses the domain module in its index.js file and contains a function named logger, which we saw in the snapshot.

![normal memory](/img/dujun/normalMemory.png)

After annotating the yog-log module, the memory usage returned to normal.

![normal memory on server](/img/dujun/normalMemoryOnServer.png)

### Summarize

1. use heapdump to collect data
2. analyze snapshot to find problem
3. verify result on test envirnment

### Easter eggs

While writing this article, I discovered another tool called [v8-profiler-rs](https://github.com/zhangyuang/v8-profiler-rs) on GitHub that provides visualization pages and auxiliary analysis reports that can help identify issues more quickly.

![v8-profiler-rs](/img/dujun/v8-profiler-rs.png)

the answer is already obvious with the logger full of screen

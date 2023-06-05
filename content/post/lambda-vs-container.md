---
title: "FaaS vs. CaaS"
date: 2022-07-28T09:46:05+05:30
lastmod: 2022-07-28T09:46:05+05:30
draft: false
keywords: []
description: ""
tags: []
categories: []
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---
One USP of public cloud is "pay only for what you use". It works quite well for storage, IOPS, and other API oriented services. Say, if you use AWS S3 storage, you pay for the amount of data stored and the number of put and get API calls. But, when it comes to compute services on a public cloud, you pay for the allocated capacity independent of whether you use that capacity. For example, if you bring up a virtual machine with 2 cores and use only 1 core, you still pay for 2 cores. For batch jobs, it is comparatively easy to calculate the required compute resources and use them fully. On the other hand, if you are serving API requests, the system load depends on the API calls served in parallel. Since the API load can be quite spiky, you need compute resources that can quickly scale out to ensure that you can meet the API response SLA and also ensure that allocated compute is used fully. In other words, quicker boot up leads to lower cost.

<!--more-->

Typically the time taken to boot up a virtual machine in typical IaaS is in the order of 10-20 minutes. It's certainly not quick enough to respond to a spiky load and hence, you would tend to keep some spare capacity to handle spikes. In other words, with virtual machines, you do not use the compute capacity fully and hence, pay for what you do not use. Container as a Service (CaaS) is a better option for serving APIs because typical boot time for a container is 1-2 minutes. If there is an increase in the load beyond the current container capacity, you have 3 options to handle the increased load during the boot up time for additional container:
1. Spare capacity: Keep spare capacity to handle increased load for 1-2 minutes.
1. Rate limit: Send back an error code so that the API caller retries after some time.
1. Serve all load with existing container: This will increase the latency for individual API requests.

Usually, a combination of the three is used. In either case, the newly started container does not have enough API requests to fully consume the compute capacity and there is always some wastage.

Compare this to FaaS where every API request spawns a new function and you do not need to keep aside any spare capacity. FaaS boot up time is usually in the range of 10-20 seconds. Typical API requests have a response time SLA much stricter than 10-20 seconds and hence, you need to perform some jugglery to keep the function warm. But apart from that, you do not need to keep any spare capacity. Hence, the cost of FaaS works out to be better... but only for low load. That's because of two reasons:
1. FaaS providers charge more per CPU and GB of RAM compared to containers. Hence, for sustained API request load, containers cost lower.
1. FaaS serves only one API request at a time and the compute capacity is fully used only when the function does not wait on network either for reading the API request or sending back response or waiting on API call to another service. When the function waits for a network IO, it's not using the CPU. Unfortunately, most functions require some network IO. Containers have the multitasking advantage with functions that alternate between network IO and compute because multiple requests are served by the same container. Some of these requests could wait for network IO while others are using the compute and hence, the compute resources are effectively used all the time.

In summary, you need to consider the cost trade-off when it comes to deciding between FaaS vs CaaS. FaaS gives better cost per API request at lesser number of simultaneous requests, whereas CaaS is better at higher load or large number of simultaneous API requests.

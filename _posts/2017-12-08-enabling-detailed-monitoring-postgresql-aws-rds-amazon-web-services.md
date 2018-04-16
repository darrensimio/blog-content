---
ID: 57
post_title: >
  Enabling Detailed Monitoring on
  PostgreSQL RDS running on Amazon Web
  Services (AWS)
author: Darren Sim
post_excerpt: ""
layout: post
permalink: >
  https://darrensim.io/posts/enabling-detailed-monitoring-postgresql-aws-rds-amazon-web-services/
published: true
post_date: 2017-12-08 21:34:54
---
In micro-service implementations where servers are treated as disposable kettles rather than pets, the data-store (usually an RDS insurance on AWS) becomes ever more important; it's the single source of truth for the application.

While this is no longer considered the best practice, the following architecture, was at one time one of the more popular patterns.

<img class="alignnone wp-image-107 size-large" src="http://darrensim.io/wp-content/uploads/2017/10/AWS-Application-Architecture-1024x622.png" alt="" width="1024" height="622" />

Unfortunately, as teams do not usually give a lot of love to their infrastructure post initial setup, this setup is still pretty common.

<h3>Importance of Monitoring the RDS</h3>

Given that a failure of the RDS is the most costly failure in such a setup, system operators (in teams that subscribe to the "you build it, you run it" mindset, this happens to be the Dev team) need to be preempted way ahead of time if there's a potential bottleneck forming, or a resource spike (e.g. CPU) that has the potential to bring the RDS to a crawl, rendering it unavailable.

In the event that a potential bottleneck or resource spike do occur, operators need to be able to have the right information on hand to quickly diagnose the issue and shutdown a bad node making calls to the RDS.

Out of the box, AWS provides the following metrics that are useful for general system health monitoring; enabling enhanced monitoring will offer operators the ability to monitor heuristically and perform diagnostics.

<h3>RDS Basic Monitoring Capabilities</h3>

Basic monitoring comes by default with all RDS installations on AWS; the RDS sends metrics to CloudWatch for each active database instance every minute.

<strong>Metrics Available</strong>
There are 24 metrics that are tracked in the Amazon RDS Metrics. These include:

[table id=6 /]

Read more on <a href="http://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/rds-metricscollected.html" target="_blank" rel="noopener">http://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/rds-metricscollected.html</a>.

Of these, the FOUR metrics which I strongly recommend wiring up CloudWatch alarms to are <em><strong>CPUUtilization</strong></em>, <em><strong>DatabaseConnections</strong></em>, <em><strong>FreeableMemory</strong></em>, and <em><strong>FreeStorageSpace</strong></em>.

<h3>Case for Detailed Monitoring</h3>

One of the key difference between default CloudWatch monitoring, and Enhanced monitoring for RDS is, CloudWatch gathers metrics about CPU utilization <strong><em>from the hypervisor</em></strong> for a DB instance, while Enhanced Monitoring gathers its metrics <strong><em>from an agent on the instance</em></strong>.

On top of receiving faster feedback <em>(1 second granularity)</em> through having an agent on the instance, having an agent on the instance also brings about enhanced monitoring, that allows operators to see how different processes or threads on a DB instance use the CPU.

This is especially useful when operators want to find out which clients (e.g. EC2) are accessing the RDS, and how much CPU and memory utilization each of these clients is generating on the RDS.

<strong>Example Log Entry</strong>

<pre class="lang:default decode:true" title="Example CloudWatch JSON Log">{
    "engine": "POSTGRES",
    "instanceID": "INSTANCE_ID",
    "instanceResourceID": "db-RESOURCE_ID",
    "timestamp": "2017-10-30T11:17:55Z",
    "version": 1,
    "uptime": "1 days, 20:17:10",
    "numVCPUs": 8,
    "cpuUtilization": {
        "guest": 0,
        "irq": 0.01,
        "system": 0.26,
        "wait": 0.26,
        "idle": 94.31,
        "user": 0.28,
        "total": 5.68,
        "steal": 0.01,
        "nice": 4.86
    },
    "loadAverageMinute": {
        "fifteen": 0.51,
        "five": 0.57,
        "one": 0.62
    },
    "memory": {
        "writeback": 288,
        "hugePagesFree": 0,
        "hugePagesRsvd": 0,
        "hugePagesSurp": 0,
        "cached": 4805564,
        "hugePagesSize": 2048,
        "free": 26788288,
        "hugePagesTotal": 0,
        "inactive": 549000,
        "pageTables": 30344,
        "dirty": 100,
        "mapped": 854956,
        "active": 5059444,
        "total": 32952096,
        "slab": 249920,
        "buffers": 303744
    },
    "tasks": {
        "sleeping": 239,
        "zombie": 0,
        "running": 2,
        "stopped": 0,
        "total": 241,
        "blocked": 0
    },
    "swap": {
        "cached": 0,
        "total": 16383996,
        "free": 16383996
    },
    "network": [
        {
            "interface": "eth0",
            "rx": 150548.8,
            "tx": 376372.3
        }
    ],
    "diskIO": [
        {
            "writeKbPS": 89.6,
            "readIOsPS": 0,
            "await": 2.1,
            "readKbPS": 0,
            "rrqmPS": 0,
            "util": 1.96,
            "avgQueueLen": 0.26,
            "tps": 12.6,
            "readKb": 0,
            "device": "rdsdev",
            "writeKb": 896,
            "avgReqSz": 7.11,
            "wrqmPS": 0,
            "writeIOsPS": 12.6
        }
    ],
    "fileSys": [
        {
            "used": 1530088,
            "name": "rdsfilesys",
            "usedFiles": 1788,
            "usedFilePercent": 0.03,
            "maxFiles": 6553600,
            "mountPoint": "/rdsdbdata",
            "total": 103053476,
            "usedPercent": 1.48
        }
    ],
    "processList": [
        {
            "vss": 1234567,
            "name": "postgres: user_name service_running 127.0.0.1 idle",
            "tgid": 4322,
            "vmlimit": 21222222,
            "parentID": 1234,
            "memoryUsedPc": 0.69,
            "cpuUsedPc": 0.09,
            "id": 4321,
            "rss": 226424
        }
    ]
}</pre>

Details on metrics available can be found on <a href="http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Monitoring.OS.html?shortFooter=true#USER_Monitoring.OS.CloudWatchLogs" target="_blank" rel="noopener">http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Monitoring.OS.html?shortFooter=true#USER_Monitoring.OS.CloudWatchLogs</a>.

As of 2017, detailed monitoring is available for all DB instance classes (except for <code>db.t1.micro</code> and <code>db.m1.small</code>), for the following database engines:

<ul>
    <li>Amazon Aurora</li>
    <li>MariaDB</li>
    <li>Microsoft SQL Server</li>
    <li>MySQL version 5.5 or later</li>
    <li>Oracle</li>
    <li>PostgreSQL</li>
</ul>

<h3>Enabling Enhanced Monitoring on AWS RDS</h3>

Enabling enhanced monitoring on an AWS RDS instance is really easy. This can be done by toggling the "Enable Enhanced Monitoring" setting on the AWS Management Console to "Yes".

To do this:

<ol>
    <li>Login to the AWS Management Console</li>
    <li>Select RDS</li>
    <li>Click on DB Instances</li>
    <li>Select the specific instance</li>
    <li>Click on Instance Actions</li>
    <li>Select Modify</li>
</ol>

<img class="alignnone size-full wp-image-105" src="http://darrensim.io/wp-content/uploads/2017/10/RDS-Enhanced-Monitoring-Configuration.png" alt="RDS Enhanced Monitoring Configuration on AWS Management Console" width="540" height="146" />

Within the configuration panel, you will also be able to 1) <em><strong>enable enhanced monitoring</strong></em>, 2) <em><strong>select monitoring role</strong></em>, and 3) <em><strong>select the granularity</strong></em> of the monitoring (in units of 1, 5, 10, 15, 30, 60 seconds).

<strong>Is Maintenance Window Required?</strong>

As this does not require a RDS reboot, the Maintenance Window is not required.

<h3>Cost Impact</h3>

The cost of enabling enhanced monitoring depends on the amount of logs that are written to CloudWatch. The general rule of thumb is the greater the granularity, the more the logs generated, which in turn, results in an increased cost.

AWS CloudWatch offers 5 GB a month for free; there after it is chargeable. Costing structure of AWS CloudWatch can be found on <a href="https://aws.amazon.com/cloudwatch/pricing/" target="_blank" rel="noopener">https://aws.amazon.com/cloudwatch/pricing/</a>. The <a href="https://calculator.s3.amazonaws.com/index.html" target="_blank" rel="noopener">AWS cost calculator</a> is another useful tool for cost estimations on AWS in general.

<h3>Accessing Logs</h3>

Detailed logs generated from enhanced monitoring are stored as JSON format in CloudWatch, accessible via <code>CloudWatch &gt; Log Groups &gt; RDSOSMetrics &gt; [Resource ID]</code>.

<h3>Useful PostgreSQL Queries</h3>

<strong>Total connections made to the PostgreSQL instance</strong>

<pre class="lang:pgsql decode:true" title="Total Connection made to the instances.">SELECT client_addr, application_name, usename, datname, Count(1)
FROM   pg_stat_activity
GROUP  BY 1, 2, 3, 4
ORDER  BY 5 DESC;</pre>

<strong>Only active connections on the PostgreSQL Instance</strong>

<pre class="lang:pgsql decode:true " title="Only the active connection">SELECT client_addr, application_name, usename, datname, Count(1)
FROM   pg_stat_activity
WHERE  state &lt;&gt; 'idle'
GROUP  BY 1, 2, 3, 4
ORDER  BY 5 DESC;</pre>

&nbsp;
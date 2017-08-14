---
layout: post
title:  "CI - Lessons learned about Continuous Integration (Part I)"
date:   2017-08-14
categories: ci tips continuous integration jenkins zuul
tags: ci tips continuous integration jenkins zuul
---

After some months as member of the Engineer Productivity team, it's time to sum up as a first step some good and bad practices about Continuous Integration I learned and implemented at leboncoin.

Some of them will seems obvious or already known but may be useful to know or recall.

Before starting, some facts about team and CI:

**EngProd** team is in charge of all CI/CD processes and tools, means from the Code Review and Git repository to package release management and delivery processes with the SRE team.

About our platforms, we run about **+6K jobs/day** on **~100 servers** (On-Premise and Cloud). All jobs are executed in containers with Docker and delivery processes are managed by a homemade tool called Christian.

# Keep It Small

## Jenkins Master

One of our first mistakes was to managed all jobs on a single Jenkins instance.
Even if this server was quite big (24 vCPUs, 145Go RAM, etc.) and dedicated to Jenkins, we faced a lot of instabilities or failed tests and provided a bad user experience.

Solution here was to **split jobs into separate Jenkins Master instances**.

That should sounds obvious, but when you started your CI platform with few jobs, everything works fine.
But with time, you added new jobs for new projects and one day you realized that your server hangs and fails to execute jobs.

You can also use a more generic jobs or use SSD backend and continue with your mono-server but consider it as a first warning.

BTW, if you're interested about tuning Jenkins Garbage Collector, this [article](https://www.cloudbees.com/blog/joining-big-leagues-tuning-jenkins-gc-responsiveness-and-stability) is a very good start.

In our case, we split our Jenkins Masters by domain team and used an other one for build & deploy jobs.

**Last thing**: To manage and version your jobs configuration between master nodes, I highly recommend you to use [Jenkins Job Builder](https://docs.openstack.org/infra/jenkins-job-builder/) project.

**Jenkins Job Builder** *(JJB)* takes simple descriptions of Jenkins jobs in YAML or JSON format and uses them to configure Jenkins. You can keep your job descriptions in human readable text format in a version control system to make changes and auditing easier. It also has a flexible template system, so creating many similarly configured jobs is easy.

**Tips**: Disable the Weather column on Jenkins. When an instance grows to be very large, and it’s folder structure has many levels, the generation of this weather column can seriously impair system performance. More [here.](https://support.cloudbees.com/hc/en-us/articles/216973327-How-to-disable-the-weather-column-to-resolve-instance-slowness)

## Slave nodes

The below advice can works with Jenkins slave nodes too.      

Let's take our previous example:    
As your master has a lot of jobs, you need to setup more and more slave nodes to meet the increasing demand.   
But adding a significant number of slave nodes also increase the number of open files on server and can produce more I/O on your disks and reduce response time of your master.

In this case, it's important to **find the correct ratio between slave nodes and executor slots**.
Having too much executor slots can produce a high load average or some OOM on nodes but, on the other hands, having many slave servers can freeze or hangs your master server.

There are no perfect answers, each CI is different and you need to analyze your jobs (execution time, resources, etc.) to find the correct setting.  

About us, we have created 5 cloud templates based on our jobs (docker, go, etc.) of slave servers with different amount of CPU/RAM & Jenkins slots. We spawn them on demand and keep their numbers as small as possible on masters.

# One ~~Ring~~ to rule them all

Having many masters adds new problems too.    
Let's see some examples:

- Having a mono-master add the possibility to run [cascade jobs](https://wiki.jenkins.io/display/JENKINS/Parameterized+Trigger+Plugin) / [pipelines](https://jenkins.io/doc/book/pipeline/).   
It means once a job is completed, Jenkins is able to launch other jobs depending on the result of the parent job.

  It should be very useful to build or deploy packages if tests succeed for example.


- If you're using [Gerrit](https://www.gerritcodereview.com/) as Code Review, you already know that you can trigger jobs in Jenkins, depending on Git events (patchset-created, ref-updated) and having results in your review under Vote labels.

  But how to ensure that the final note (-1/+1) gathered all jobs ?

To solve these issues, we choose to use the OpenStack Project [Zuul](https://docs.openstack.org/infra/zuul/).

**Zuul** is a pipeline oriented project gating and automation system.

Zuul watches events in Gerrit (using the Gerrit *“stream-events”* command) and matches those events to pipelines. If a match is found, it adds the change to the pipeline and starts running related jobs.

The gate pipeline uses speculative execution to improve throughput. Changes are tested in parallel under the assumption that changes ahead in the queue will merge. If they do not, Zuul will abort and restart tests without the affected changes. This means that many changes may be tested in parallel while continuing to assure that each commit is correctly tested.

Zuul is composed by three main components:
- **zuul-server**: scheduler daemon which communicates with Gerrit and Gearman. Handles receiving events, launching jobs, collecting results and postingreports.
- **zuul-merger**: speculative-merger which communicates with Gearman. Prepares Git repositories for jobs to test against. This additionally requires a web server hosting the Git repositories which can be cloned by the jobs.
- **zuul-cloner**: client side script used to setup job workspace. It is used to clone the repositories prepared by the zuul-merger described previously.

As Zuul is a pipeline oriented project, you can define different types of pipeline based on different trigger actions:

Let's see some pipeline examples:

- **check-build**: Newly uploaded patchsets enter this pipeline to receive an initial +/-1 Verified vote from for build test jobs. Ex: build packages, docker images, etc.
- **post**: This pipeline runs jobs that operate after each change is merged.
- **release**: When a commit is tagged as a release, this pipeline runs jobs that publish archives and documentation.
- **periodic-nightly**: This pipeline has jobs triggered on a timer for e.g. testing for environmental changes each night.

It also provide a web-based Dashboard to follow execution runs: [here](http://status.openstack.org/zuul/) OpenStack's Dashboard.

Zuul is used by some major Open Source projects such as [OpenStack](https://docs.openstack.org/infra/system-config/zuul.html) or [Wikimedia](https://www.mediawiki.org/wiki/Continuous_integration/Zuul) and it is very scalable and robust.

**Tips**: Think about using template names for your jobs, it will be easier to read and understand between Zuul, JJB, Gerrit and Jenkins.   
Example: **[type]**-**[project]**-**[distrib/purpose]**

There's a lot to say about CI, but that's all for today and I'll continue on a future post.

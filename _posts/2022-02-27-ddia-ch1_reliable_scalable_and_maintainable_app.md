---
layout: post
title: "DDIA Study -  Chapter 1 : Repliable, Scalable and Maintainable Applications (ENG)"
categories: DDIA
author: "Yongchan Hong"
---

# Chapter 1 : Repliable, Scalable and Maintainable Applications

> In this DDIA category, I will post a summary of Designing Data Intensive Applications (DDIA) from chapter 1 to 12.

In this chapter, we learn about the definition of replicable, scalable, and maintainable data system.

## **Reliability**

> System should continue to work correctly even in the face of adversity(hardware, software fault or human error).

A reliable system should work even with *faults*.  

> Fault vs Failure
>
> A fault is when a component in the system has escaped from the expected spec.
>
> A failure is when the system cannot provide the required service to the user.
>
> We can possibly build a fault-tolerant system by expecting faults and build ths system to prevent expanding to the failure.
>
>*Netflix randomly triggers faults to build a fault-tolerant system - Chaos Monkey*  

So what kinds of fault should we expect?  

### **Hardware Faults**

Problems that may be caused by hard disk crash, RAM, blackout, etc.

Previously, redundancy of hardware components can prevent total failure (By simply restoring a backup). However, due to increasing amount of volume and computing demands, simple redundancy of hardware components may not solve a problem. Thus, `software fault-tolerance technique` is preferred more.

### **Software Errors**

These are faults that may be created by a software bug. These faults may not be appeared until unusual circumstances that triggers the bug.

### **Human Errors**

Faults that are created by the mistake of operators. Providing a wrong configuration value while deploying a software can be a good example.

## **Scalability**

>System should deal with growth of data volume, traffic volume and complexity.

Scalability is used to describe whether system can cope with increased load.

Load can be described by *load parameters*, such as requests per second, the ratio of read to writes in a database. (Check Twitter case)

### **How to measure**
Performance can be normally measured by **throughout** and **response time**.

> Throughput is the number of records that can processed per second, or total time to run a job of certain size dataset.

> Response time includes the service time, network delays and queueing delays. Latency is the duration to wait a request to be handled.

Response time should be measured by the distribution - especially p50, p95, p99. **Tail latencies**, which are high percentiles of response times, should be cconsiered since it may affect user experiences.

> Superset Locust test (*My experience*)
> 
> Before tuning Superset, median and mean was relatively okay, but p99 was 10 seconds which was terrible. 

**Head-of-line blocking** should be also considered since several slow requests may block the processing of upcoming requests.

### **How to cope**

We should considering either **scaling up** or **scaling out** to deal with increase of load parameters.

> Scaling up (vertical scaling) is to move to a more powerful machine with more memory/CPU resource.
>
> Scaling out (horizontal scaling) is to distribute the load to multiple smaller machines.

Since there is no pefect scaling architecture, we should consider crucial components of the service and decided based on that components.

## **Maintainability**

> Operators (coworkers) should productively develop and maintain the system.

### **Operability**

System can be easily maintained by the operating team.

Well-written documents, monitoring, automation, etc are required.

### **Simplicity**

System should be simple. 

Code, architecture, module should not be complex.

Abstraction is a good way to prvent accidential complexity.

### **Evolvability**

System can easily evolve when there are new/revised requests.
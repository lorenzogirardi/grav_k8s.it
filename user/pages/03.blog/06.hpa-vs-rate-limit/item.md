---
title: 'HPA vs Rate-limit'
date: '2023-02-14 20:23'
taxonomy:
    tag:
            - cost saving
            - dynamic infrastructure
            - envoy
            - hpa
            - rate limit vs hpa
            - rate-limiting
---

### Table of Contents

  * INTRO
    * HPA
      * Patterns
      * Meaning
      * Ideal Practice
    * Rate Limit
  * GOALS
  * Limits
  * Hands-on
    * Simulation
      *         * Note
      *       * NO AUTOSCALING
        * First run 40rps max
        * Second run 33rps max
      * AUTOSCALING
        * 1 pod start - hpa 33rps - 15 min - 99 max requests
        * 2 pod start - hpa 33rps - 15 min - 99 max requests
        * 1 pod start - hpa 30rps - 15 min - 99 max requests
        * 1 pod start - hpa 27rps - 15 min - 99 max requests
        * [optimal] 1 pod start - hpa 24rps - 15 min - 99 max requests
        * 1 pod start - hpa 25rps - 15 min - 99 max requests
      * AUTOSCALING Internal rate limit
        * [optimal] 1 pod start - hpa 25rps- 15 min - 99 max requests rate-limit 27
        * 1 pod start - hpa 26rps - 15 min - 99 max requests rate-limit 28
        * 1 pod start - hpa 27rps - 15 min - 99 max requests rate-limit 29
      * AUTOSCALING envoy rate limit
        * 1 pod start - hpa 27rps - 15 min - 99 max requests rate-limit 29 - envoy
        * [optimal] 1 pod start - hpa 26rps - 15 min - 99 max requests rate-limit 29 - envoy
      * Unexpected traffic
        * 3 pod start - hpa 26rps - 15 min - 82 avg requests with spikes - rate-limit 27 internal
        * [optimal] 3 pod start - hpa 26rps - 15 min - 82 avg requests with spikes - rate-limit 27 - envoy
      * Conclusions
        * HPA
        * Rate Limit
        * Both
        * Costs
          * Highlights



Strange ... we are using hpa to increase the availability and we are introducing rate limit to reduce? 

Well let's create the context...

# INTRO

This _story_ is not a true o false sentence, it's a sort of analysis based on a few assumptions:

  * cloud environment
  * dynamic infrastructure
  * minimum resources available 



## HPA

In Kubernetes, a _HorizontalPodAutoscaler_ automatically updates a workload resource (such as a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) or [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)), with the aim of automatically scaling the workload to match demand.

### Patterns

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-20.28.38.png)

### Meaning

**Type**| **Behaviour**  
---|---  
Slow and temporary| it might have daily fluctuations in requests volume, peaking during the day and troughing at night  
Rapid and temporary| it might be subject to short bursts of high request volume from poorly-behaved downstream services  
Slow and persistent| it might see its request volume slowly increase over rime, as the product sees greater adoption  
Rapid and persistent| it might see abrupt shift from low to high volumes, such as if it's called by batch jobs  
  
### Ideal Practice

**Type**| **Behaviour**  
---|---  
Slow and temporary| The HPA should add and remove pods as necessary  
Rapid and temporary| The HPA should not modify the pod count; instead, the service should leave enough headroom to deal with these brief spikes with only existing pods  
Slow and persistent | The HPA should add and remove pods as necessary  
Rapid and persistent| The service should leave enough headroom to deal with the rapid change, and the HPA should add pods soon after to bring the service back to target utilization  
  
## Rate Limit

A rate limit is the number of API calls an app or user can make within a given time period. If this limit is exceeded or if CPU or total time limits are exceeded, the app or user may be throttled. API requests made by a throttled user or app will fail. All API requests are subject to rate limits.

However, keep the HPA patterns as a reference even for this one.

# GOALS

  * Understand how much we can close with the application enervation during an autoscaling situation using rate limit
  * Understand how to handle unexpected traffic with rate limit



# Limits

Sometimes we can have a spike of requests that are legit... for example consider the google crawlers (image from ... somewhere in google image)

![](/user/images/hpa-vs-rate-limit/logs-crawl-spikes-three-c-sm-border.png)

# Hands-on

## Simulation

  * [python](https://github.com/lorenzogirardi/py-test-backend) app that generates fibonacci sequence
  * fibonacci number 18500
  * load test done using gatling
  * Application capped on cpu (each request is ~10mcpu) with max of 300mcpu for pod (cpu bound)
  * Hpa based on keda flask_http_request_duration_seconds_count



Results Example 

$ curl http://xxx/api/fib/18500  
8353329688443562486779853158514... etc etc  


This ensures that it is not just an echo answer and that we have work behind the scenes.

#### Note

Gatling is using an interesting concept that call "active users"

“Active users” is neither “concurrent users” or “users arrival rate”. It’s a kind of mixed metric that serves for both open and closed workload models and that represents “users who were active on the system under load at a given second”.

It’s computed as:

(number of alive users at previous second)  
\+ (number of users that were started during this second)  
\- (number of users that were terminated during previous second)

### 

### NO AUTOSCALING

#### First run 40rps max

rampUsers(10).during(60.seconds), // increase users to 10 in 60 sec  
constantUsersPerSec(10).during(60.seconds), // keep 10 users for 60 sec  
rampUsersPerSec(10).to(40).during(5.minutes), // increase users to 40 in 5 min

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.15.19.png)

As soon as we reach the limit the pods start to be unresponsive

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.16.02.png)  
No metrics after 33 rps, pod crashed  
  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.17.54.png)  
No metrics after >28% of cpu (consider the max as 30% as the pod limit is 300mcpu)  
  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.19.22.png)Failure rate (low) when rps > 35 and high response time  
  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.20.38.png)  
Active users increase dramatically after 33 rps  
  


#### Second run 33rps max

rampUsers(10).during(60.seconds), // increase users to 10 in 60 sec  
constantUsersPerSec(10).during(60.seconds), // keep 10 users for 60 sec  
rampUsersPerSec(10).to(30).during(4.minutes), // increase users to 30 in 4 min  


![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.24.14.png)

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.24.22.png)

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.24.30.png)

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.24.39.png)

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.24.46.png)

Really better, we can say that the MAX rps is 33 ... this valid _one-shot_ in a single run

### AUTOSCALING

rampUsers(10).during(60.seconds), // increase users to 10 in 60 sec  
constantUsersPerSec(10).during(60.seconds), // keep 10 users for 60sec  
rampUsersPerSec(10).to(99).during(15.minutes), // increase users to 99 in 15 min

#### 1 pod start - hpa 33rps - 15 min - 99 max requests

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.30.24.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.30.30.png)  
DEAD !!!   
  
Test stopped as per ...

================================================================================  
2023-02-09 10:01:38 885s elapsed  
\---- Requests ------------------------------------------------------------------  
> Global (OK=17944 KO=18236 )  
> request_1 (OK=17944 KO=18236 )  
\---- Errors --------------------------------------------------------------------  
> status.find.in(200,201,202,203,204,205,206,207,208,209,304), f 8358 (45.83%)found 503  
> status.find.in(200,201,202,203,204,205,206,207,208,209,304), f 5529 (30.32%)found 502  
> status.find.in(200,201,202,203,204,205,206,207,208,209,304), f 4273 (23.43%)found 504  
> i.g.h.c.i.RequestTimeoutException: Request timeout to oracolo. 73 ( 0.40%)  
> j.i.IOException: Premature close 3 ( 0.02%)  
\---- BasicSimulation -----------------------------------------------------------  
[#####################################################-- ] 72%  
waiting: 12549 / active: 932 / done: 36180  
================================================================================

#### 2 pod start - hpa 33rps - 15 min - 99 max requests

Autoscaler is not able to support the traffic in the picture before , the curve is stressing the namespace in a way to make impossible the scaling and reliability, this can be by a single pod start and ratio of requests absorbed, let’s try starting with 2 , it’s supposed in that way to have half of the requests to absorb each one  
  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.41.18.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.41.29.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.41.37.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.41.44.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.41.52.png)  
Naaaa... still not able to scale 

  
  


#### 1 pod start - hpa 30rps - 15 min - 99 max requests

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.44.45.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.44.57.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.45.03.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.45.10.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.45.17.png)  
Ok we are able to scale but at the end we have a complete crash

####   
1 pod start - hpa 27rps - 15 min - 99 max requests

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.53.36.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.53.43.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.53.51.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.53.58.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.54.07.png)  
Oh noooo ... worse than before with fewer rps ... since we are over-stressed when start to crash can have weird behaviour 

####  [optimal] 1 pod start - hpa 24rps - 15 min - 99 max requests

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.58.23.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.58.33.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.58.41.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.58.51.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-21.58.58.png)  
Here we are !!!

#### 1 pod start - hpa 25rps - 15 min - 99 max requests

let's try with one more

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.00.50.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.00.59.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.01.06.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.01.15.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.01.22.png)  
Naaaa .... no more than 24 rps

### AUTOSCALING Internal rate limit

[Here how to](https://www.k8s.it/application-rate-limiting.html)

#### [optimal] 1 pod start - hpa 25rps\- 15 min - 99 max requests rate-limit 27

I suppose that the _optimal_ before (24rps) is working good also with RL so let’s start with 25

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.08.07.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.08.14.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.08.20.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.08.29.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.08.34.png)

Better than the same rps with no ratelimit  


The errors are just the 429 triggered by rate limit … so the good answer are ok and no issue or restart on the application during the autoscaling… a bit of queue

No so bad considering the errors in red as just rate limited calls

#### 1 pod start - hpa 26rps - 15 min - 99 max requests rate-limit 28

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.11.25.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.11.33.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.11.41.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.11.49.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.11.56.png)

Just one error for pod unresponsive

#### 1 pod start - hpa 27rps - 15 min - 99 max requests rate-limit 29

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.13.57.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.14.08.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.14.14.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.14.21.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.14.28.png)  
Broken!!!

### AUTOSCALING envoy rate limit

#### 1 pod start - hpa 27rps - 15 min - 99 max requests rate-limit 29 - envoy

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.17.26.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.17.33.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.17.39.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.17.46.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.17.53.png)  
Broken!!!

#### [optimal] 1 pod start - hpa 26rps - 15 min - 99 max requests rate-limit 29 - envoy

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.19.24.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.19.30.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.19.38.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.19.46.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.19.52.png)

Better than expected compared to the solution with rate limit embedded 

### Unexpected traffic

rampUsers(30).during(60.seconds),  
constantUsersPerSec(30).during(60.seconds).randomized,   
rampUsersPerSec(30).to(82).during(4.minutes),   
constantUsersPerSec(82).during(120.seconds).randomized,  
rampUsersPerSec(82).to(170).during(10.seconds),  
constantUsersPerSec(82).during(120.seconds).randomized,  
rampUsersPerSec(82).to(130).during(20.seconds),  
constantUsersPerSec(82).during(120.seconds).randomized,  
rampUsersPerSec(82).to(210).during(30.seconds),  
constantUsersPerSec(82).during(120.seconds).randomized,  
constantUsersPerSec(333).during(2.seconds),  
constantUsersPerSec(82).during(120.seconds).randomized,  
constantUsersPerSec(333).during(10.seconds),  
constantUsersPerSec(82).during(80.seconds)

#### 3 pod start - hpa 26rps - 15 min - 82 avg requests with spikes - rate-limit 27 internal

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.25.08.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.25.15.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.25.22.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.25.29.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.25.36.png)

Since the incoming requests are absorbed from the app we reach the limit  
Errors are not only 429 as rate but 5xx

Broken!!!

#### [optimal] 3 pod start - hpa 26rps - 15 min - 82 avg requests with spikes - rate-limit 27 - envoy

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.27.32.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.27.40.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.27.46.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.27.54.png)  
![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.28.01.png)

No issue at all , since the rl is performed by envoy the python app has no “spikes”

### Conclusions

#### HPA

This application is cpu bound … so I used req/s even if the right autoscaling metric is the cpu usage, however, I would say that each application has the intent to serve requests.  
Create a dedicated configuration, application per application, for the hpa is the best option but it requires **HUGE** know-how and time …  
Considering the sentence before, I was impressed about [Active users](https://gatling.io/docs/gatling/reference/current/stats/reports/#:~:text=%E2%80%9CActive%20users%E2%80%9D%20is%20neither%20%E2%80%9C,load%20at%20a%20given%20second%E2%80%9D.) since share the correct clue at high levels about the status of the application, it’s really really close to measuring the enervation of the app

![](/user/images/hpa-vs-rate-limit/Screenshot-2023-02-16-at-22.30.45.png)

I would say that the concept of active users as a HPA metric is better than flask_http_request_duration_seconds_count

Even if we can reach 33rps in a single pod , we should consider that in an autoscaling scenario this will be reduced to **less,** cause for a certain amount of time we have more requests till we are waiting for new pods/node etc etc 

I would say that is better to have an application running at 65% of max capacity before scale

#### Rate Limit

  
 __

  * _Understand how much we can close with the application enervation during an autoscaling situation using rate limit_



**No** … or better … we can increase a bit the limit to exploit the app at max possible, however, it’s just a risk for 5% more in rps.  
The application will be more sensible and the rl is really close to the hpa

  * Understand how to handle unexpected traffic with a rate limit



**Yes** … this pattern is the right one for the rate limit  
Better is to perform this action outside the application container since will impact unfortunately the incoming requests, the saturation etc etc and you have rl and hpa that can fight each other 

#### Both

  * Do not force hpa to collaborate with rl … should be 2 different parameters



In this scenario are both rps ... but are profiled in a different way , so do not consider as the same metric or meaning

#### Costs

In some way trying to save money by increasing to the possible max rps is risky, we should leverage the kubernetes pod scheduling more than try to reach 100% of the application

##### Highlights

Unexpected traffic can be something that is generated for a wrong deployment/batch 5% of cases or by external calls 95%

The external calls come from public or private endpoints can be managed by  


  * waf rate limit
  * api gateway rate limit



Those two solutions are better, cause they can keep the infrastructure safe as a first point of contact but ... A big **BUT**

In a dynamic infrastructure where we have autoscaling and downscaling, a legit peak like google scrape is not possible to handle, cause the RL is service level   
**AND…**   
during the night, it's supposed to have the minimum pods available as there is less user traffic for example.

So … I’ll remark on the drawback of cost savings following the matrix based on hpa patterns (cit [brewster's millions](https://www.imdb.com/title/tt0088850/))  
As much as we are close to using fewer resources as possible, as much we are increasing corner cases and unexpected situations.  
  
Be careful to evaluate the rate limit concept 

Cheers :-) 

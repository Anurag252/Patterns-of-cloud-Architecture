# Patterns-of-cloud-Architecture
initilly we had multiple programs running on OS , then came , client server , then distributed systems

## Single Node Pattern
### Sidecar pattern
 The sidecar pattern is a single-node pattern made up of two containers. The first is the application container. It contains the core logic for the application. Without this container, the application would not exist. In addition to the application container, there is a sidecar container. The role of the sidecar is to augment and improve the application container, often without the application container’s knowledge
 
 
 example :-
 1. Consider, for example, a legacy web service. Years ago, when it was built, internal network security was not as high a priority for the company, and thus, the application only services requests over unencrypted HTTP, not HTTPS. Due to recent security incidents, the company has mandated the use of HTTPS for all company websites. To compound the misery of the team sent to update this particular web service, the source code for this application was built with an old version of the company’s build system, which no longer functions. Containerizing this HTTP application is simple enough: the binary can run in a container with a version of an old Linux distribution on top of a more modern kernel being run by the team’s container orchestrator. However, the task of adding HTTPS to this application is significantly more challenging. The team is trying to decide between resurrecting the old build system versus porting the application’s source code to the new build system, when one of the team members suggests that they use the sidecar pattern to resolve the situation more easily.

The application of the sidecar pattern to this situation is straightforward. The legacy web service is configured to serve exclusively on localhost (127.0.0.1), which means that only services that share the local network with the server will be able to access the service. Normally, this wouldn’t be a practical choice because it would mean that no one could access the web service. However, using the sidecar pattern, in addition to the legacy container, we will add an nginx sidecar container. This nginx container lives in the same network namespace as the legacy web application, so it can access the service that is running on localhost. At the same time, this nginx service can terminate HTTPS traffic on the external IP address of the pod and proxy that traffic to the legacy web application (see Figure 2-2). Since this unencrypted traffic is only sent via the local loopback adapter inside the container group, the network security team is satisfied that the data is safe. Likewise, by using the sidecar pattern, the team has modernized a legacy application without having to figure out how to rebuild a new application to serve HTTPS.
<img width="490" alt="Capture" src="https://user-images.githubusercontent.com/4143476/133060991-46143818-caf5-4f75-bfc6-2822cb9fa75f.PNG">

To be successful, the sidecar should be reusable across a wide variety of applications and deployments. By achieving modular reuse, sidecars can dramatically speed up the building of your application.

However, this modularity and reusability, just like achieving modularity in high-quality software development requires focus and discipline. In particular, you need to focus on developing three areas:

Parameterizing your containers :- pass important parametrs as env varibale 

Creating the API surface of your container :- define the API exposed by container

Documenting the operation of your container :- document your dockerfile and add the label and other things

### Ambassadors
This chapter introduces the ambassador pattern, where an ambassador container brokers interactions between the application container and the rest of the world. As with other single-node patterns, the two containers are tightly linked in a symbiotic pairing that is scheduled to a single machine. A canonical diagram of this pattern is shown in

![image](https://user-images.githubusercontent.com/4143476/133061064-e768bf88-99ed-4e0a-950f-39e89c15ce9f.png)

service discovery :- 

Consequently, building a portable application requires that the application know how to introspect its environment and find the appropriate MySQL service to connect to. This process is called service discovery, nd the system that performs this discovery and linking is commonly called a service broker
As with previous examples, the ambassador pattern enables a system to separate the logic of the application container from the logic of the service broker ambassador. The application simply always connects to an instance of the service (e.g., MySQL) running on localhost. It is the responsibility of the service broker ambassador to introspect its environment and broker the appropriate connection. This process is shown in

example of ambassador 👎
1. experimentation vs prod service broker
2. service discovery 
3. sharding service 

###  Adapters

service to match the output expected by other service 
![image](https://user-images.githubusercontent.com/4143476/133069236-c8f8b8d6-4ca6-4ce0-bacb-47fd350ced1a.png)

example :--
Monitoring --> prometheus
for example Redis does not expose metrics in the format like prometheus 
so we can write an adapter pattern and run it properly

Logging to correct format

Health checks


## Serving Patterns

the term microservices has become a buzzword for describing multi-node distributed software architectures

### Replicated Load balanced service

load balancer with stateless service :--
In a three-nines service, you get 1.4 minutes of downtime per day (24 × 60 × 0.001).

in general we need 2 replicas atleast to achieve HA with SLA in check
otherwise we have only 1.4 mins to do upgrade

In contrast, a readiness probe determines when an application is ready to serve user requests. The reason for the differentiation is that many applications require some time to become initialized before they are ready to serve. They may need to connect to databases, load plugins, or download serving files from the network. In all of these cases, the containers are alive, but they are not ready. When building an application for a replicated service pattern, be sure to include a special URL that implements this readiness check.

Generally speaking, this session tracking is performed by hashing the source and destination IP addresses and using that key to identify the server that should service the requests. So long as the source and destination IP addresses remain constant, all requests are sent to the same replica.

IP-based session tracking works within a cluster (internal IPs) but generally doesn’t work well with external IP addresses because of network address translation (NAT). For external session tracking, application-level tracking (e.g., via cookies) is preferred.

Often, session tracking is accomplished via a consistent hashing function. The benefit of a consistent hashing function becomes evident when the service is scaled up or down. Obviously, when the number of replicas changes, the mapping of a particular user to a replica may change. Consistent hashing functions minimize the number of users that actually change which replica they are mapped to, reducing the impact of scaling on your application.

we generally use a caching layer after LB before hitting application

we should have rate limiting before web cache

When a user hits the rate limit, the server will return the 429 error code indicating that too many requests have been issued. However, many users want to understand how many requests they have left before hitting that limit. To that end, you will likely also want to populate an HTTP header with the remaining-calls information. Though there isn’t a standard header for returning this data, many APIs return some variation of X-RateLimit-Remaining.

![image](https://user-images.githubusercontent.com/4143476/133075145-5ba36e6c-44c9-404e-a31d-dec59d7fcd51.png)

SSL termination at alb level

### Sharded Service
Replicated services are generally used for building stateless services, whereas sharded services are generally used for building stateful services. The primary reason for sharding the data is because the size of the state is too large to be served by a single machine. Sharding enables you to scale a service in response to the size of the state that needs to be served.

Each cache has 10 GB of RAM available to store results, and can serve 100 requests per second (RPS). Suppose then that our service has a total of 200 GB possible results that could be returned, and an expected 1,000 RPS. Clearly, we need 10 replicas of the cache in order to satisfy 1,000 RPS (10 replicas × 100 requests per second per replica). The simplest way to deploy this service would be as a replicated service, as described in the previous chapter






 

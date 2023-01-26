---
layout: posts
share: true
title:  "Top 10 architectural highlights for Digital Ocean Kubernetes"
date:   2021-10-27 20:46:34 +0100
categories: Kubernetes, Digital Ocean, Solution Architecture
---

Recently I've been developing a solution architecture for a boostrapped startup in Digital Ocean's Kubernetes. Developing an understanding of the context, discovering the domain and taking initial ideas through critical design thinking has been key to a foundational architecture that should serve this product well throughout its lifecycle. As envisioning has happened, the solution and its architecture has evolved to enable numerous product iterations, building out only what has been necessary at each stage. The domain driven approach to development led to a server based query gateway and so what unfolded was containerised microservcies architecture orchestrated by Kubernetes. Here are my top 10 highlights from building in Digital Ocean Kubernetes:

![GitHub Logo](/assets/images/containers.jpg)
Source: https://www.pexels.com/

### 10. Utilise external infrastructure for completeness when necessary

The absence of key architectural components to deploy in front of the cluster is a limitation to be aware with Digital Ocean. If you are building for production route traffic via services you can route traffic through CloudFlare, for example, for a layer 7 firewall, OWASP compliance and global server load balancing.

### 9. Work from the application resource requirements to determine the minimum viable infrastructure

If like most you have machine specifications to provision into your cluster (AWS Fargate the notable exception) then it makes sense to understand what resources your applications need and then work down the stack. One approach is to group applications into logical planes - Control, User, Data plane - to determine what resources are needed in each plane. Also look to vendors and application specifications for guidance on resource and scaling requirements.

### 8. Consider the velocity of scaling in the vertical and horizonal directions and the impact on services 

For horizontal scaling there is speed to think about: kubernetes will replicate pods across nodes in a matter of seconds but if new nodes are required to achieve that replication that it can take minutes. The trade-off here is between performance efficiency and cost optimisation. Set the autoscaling thresholds so there is enough spare capacity in the pods to allow time for the autoscaling to happen or provision larger nodes that enable sideways replication of pods on that same node. Develop event driven microservices with messaging queues to add resiliency. In production, use metrics to determine and refine the right vertical and horizontal thresholds.

### 7. Strive for the 'rule of three' for high availability

For a production and staging deployment I like to follow the rule of three. Three nodes in three availability zones with three pod replicas in each zone. For stateful applications being deployed into a cluster configuration it is recommended to have three cores or leader-eligible instances to avoid split brain.   

### 6. Build with open-source, multi-vendor or community-developed applications for portability

If you can build with open-source, multi-vendor and community-developed applications you can avoid vendor lock-in and keep the door open to other clouds as needs evolve over time. For example, in my case, building with Hasura GraphQL rather than AWS AppSync as a graph QL gateway, and Neo4J rather than AWS Neptune as a graph data engine.

### 5. Avoid constraints and design with soft intent for scheduling flexibility

Imposing hard constraints such as anti-affinity rules, taints and tolerations could result in a pod being unschedulable. Unless you need to import hard constraints use soft requirements such as topology keys to describe scheduling intent and best effort across nodes.

### 4. Strive to understand the limitations of the network backbone

Be aware of the limitations of the backbone. Public clouds vary significantly in their network speed. All the autoscaling in the world won't help if the bottleneck is the backbone. 

### 3. Use a service mesh for resiliency

Since Digital Ocean doesn't provision Kubernetes with the Kubernetes Networking Plugin a single control plane is limited to orchestrating pods across a single availability zone. That doesn't need to be an impediment to high availibility though, since with a service mesh (e.g., Traefik, Itsio or Consul) high availibilility can be achieved through the service mesh gateways that enable applications to connect to services that route to pods in clusters in other regional data centers. Relying on the service mesh for service availability could be a good trade-off if the only way to achieve control plane replication and orchestration across availablility zones means looking to more mature and costly platforms. Bear in mind that in a mesh, as applications run at the edge, regional data centers start behaving a bit like secondary availability zones.

### 2. Use managed services to abstract devOps from infrastructure details

Understanding the ops environment that the solution is being deployed into is key for a successful operation. If the ops environment is not assessed to be ready to operate the applications being proposed, moving them into a managed service can be a good option. This is where the marketplace shines because self-managing data engines (especially in clustered or fully distributed configurations) would most certainly warrant a dedicated team of site reliability engineers trained on the native technology, its disaster recovery procedures amongst other things. As a managed service however, availability, scaling, and disaster recovery (including point-in-time recovery to the nearest second) are trivial to configure. My view is that money is well spent here to abstract debt-laden DevOps from application infrastructure and to enable the focus firmly on the product and hypothesis-driven development.

### 1. Use a favourable pricing model to get into production

For whatever Digital Ocean might lack in edge services and availability zones it makes up for in infrastructure costs. VM pricing per hour is competitive even against the usage discounting applied by the major public cloud providers to the extent that it could extend the of life of a startup and its funding significantly. The virtual private cloud is provided at no cost and data egress is not chargable, which depending on what you are building, could present a significant cost saving (though be way of the limits of the network).  Building with open-source, multi-vendor and community-developed applications on an open platform means porting to another cloud is an option later on when services at the edge and secondary availability zones is probably going to make more sense anyway.




+++
date = "2016-07-11T12:02:56+07:00"
draft = true
title = "Sitepod Project Incubation"

+++

### Introduction 

Sitepod aims to be an open and extensible web-site hosting platform, it's automation software designed to run on your own infrastructure providing services and user-friendly interfaces to simplify the process of hosting websites of any stack whilst embracing today's best practices in areas of provisioning, isolation, resource management, and monitoring. 

Sitepod is being designed to be suitable for all sizes of deployment from single websites to multi-tenant agencies through to large service providers offering mass hosting solutions. 

By utilizing Google's [Kubernetes](http://www.kubernetes.io) and [Docker](http://www.docker.com)'s open source software for orchestration and containers respectively Sitepod puts the puzzle pieces together without enforcing opinionated practices on end users.

### You may be interested to know

* Sitepod is built as a control layer on top of Google Kubernetes and indirectly Docker.
* Sitepod development contributes back to the community, some of our small contributions to Kubernetes recently
 [#25122](https://github.com/kubernetes/kubernetes/pull/25122)
 [#25938](https://github.com/kubernetes/kubernetes/pull/25938)
 [#26504](https://github.com/kubernetes/kubernetes/pull/26504)
 [#26012](https://github.com/kubernetes/kubernetes/pull/26012) with more in the pipeline.
* Sitepod will be compatible with out-the-box Kubernetes deployments from v1.4, use our bootstrap script or the supported Chef, Salt, Ansible, AMI tools for node management and setup. 
* Sitepod code and supporting Docker images will be licensed under an open source approved license.
* Initial release targeting PHP deployments.
* It is early days, no alpha or beta

### Current landscape

Sitepod will not be the first participant in this space, there are many popular web hosting control panels with a long history, including [cPanel](http://www.cpanel.com), [Plesk](http://www.plesk.com), [Ensim](htttp://www.ensim.com) and hybrid platform as a services such as [ServerPilot](http://www.serverpilot.io) and [Cloud66](http://www.cloud66.com), closed platforms like [WebFaction](http://www.webfaction.com), and opinionated platforms like [Deis](https://www.deis.com) and [Apcera](https://www.apcera.com), that all aim, in part at least, to provide a solid platform for web sites, granted some fail miserably and are the daily hate of many sysadmins. 

INSERT: Why sitepod approach is different, why that's important, why open and extensible
POSSIBLE: Discuss why some fail miserably? Unprofessional? 

### The Team

For now one, just me, Matt Freeman. I have a strong interest in hosting and devops, hosting has been an interest since founding WebHostingTalk way back in 2000, and dev ops since implementing a Heroku clone called OpenRuko, but the biggest driver is the dislike of the current solutions as I help clients with their website devop problems. I feel in the real world, not all projects are adaptable to 12-factor methodology and CI workflows, and also desiring an easy to use UI should not be dismissed. 

### Full Circle

I started exploring the idea for Sitepod at the start of 2016, and over the last six months it has taken a few different directions, very much one step forward two back at times. As I understood more about the challenges and what existing open source  software could offer and how it could fit into Sitepod I reluctantly reneged on many design decisions, and unsurprisingly this has proven a large time sink. 

Ultimately with the change in direction and a higher volume of client work than expected, I've not made as much progress as I had anticipated, but much has  been learned, and the basis on which I build on now I feel is much stronger. Hindsight.

#### First attempt

In the first attempt I created high-level concepts of websites, databases, php-fpm workers (as this was going to be used for Magento client) and built an API
server which validated and persisted these using event log persistence. The API was loosely modeled on RESTful 
patterns of Kubernetes with some additions such as actions. The api then projected a declarative payload for the
node/minions which setup the local config files and handed off to Kubelet via a manifest file for docker management.
As I progressed with this I borrowed more ideas from Kubernetes, and eventually directly copied into the project 
parts of the Kubernetes codebase which lead to a realization that ultimately using Kubelet without the
other parts of Kubernetes would result in a lot of duplication and shouldering a lot of code not specific to  the
business problem. This code rests as [sitepod-p1-dead](https://github.com/sitepod/sitepod-p1-dead), prewarned
this isn't a pretty picture.

#### Second attempt

The second iteration I spent much more time to understand the Kubernetes framework and the internals of third party
resources, controllers, informers and work queues, this second iteration is based directly on Kubernetes codebase. 
With this, I decided to use Third Party Resources to handle the Sitepod concepts, a kind of custom type support in Kubernetes which would
give me the persistence, meta validation, watch logic and indirectly the benefits of etcd backed storage.
After a few contributions to get third party resources into a working state this approach seem to be working well initially,
and with the enhancements to Kubernetes recently including init containers, and those coming in Kubernetes 1.3.1 
or 1.4 such as fine grain permissions for config maps and secrets this also removes the need for a custom 
kubelet or Sitepod specific node helpers outside of Kubernetes. But all is not rosey , going forward exposing a UI 
and an API then this approach requires another facade in front of the Kube apiserver to support enforcement of invariants of a 
Sitepod cluster as TPRs validation facilities are limited, or alternatively extending Kube apiserver with new types and admission controllers. 
Both of these options don't appeal to me so I'm also abandoning this concept eventually. This is currently resting at  
[sitepod-p2-dead](https://github.com/sitepod/sitepod-p2-dead).


### Nailing the design

Kubernetes releases are very near feature wise to the point where Sitepod requires no Sitepod specific node helpers, or 
bastardized Kubelet software, and in the meantime from a development point we can patch master to continue.
This means compatibility with out-the-box Kubernetes clusters, and the 
choice of dozens of deployment options already available, from IaaS such as GKE (Google) to Salt, Ansible, Chef, or 
Hypercube can apply to Sitepod. 

Third Party resources whilst instrumental in flushing out some unknowns and understanding the intriancy of cache invalidation, are not a great fit, the limited validation 
available to enforce high-level Sitepod specific invariants, the lack of sub-resources support for status, 
enforced finalizers, and limited relationship support afforded by labels means that a lot of work would need to 
be done by a custom facade API server. Instead going forward Sitepod API Server will be standalone and responsible to for it's own concepts, 
including persistence, validation and providing a Kubernetes list & watch compatible endpoint.s The Sitepod API server will transform
the Sitepod concepts to Kubernetes concepts and communicate with a Kubernetes API server for Kubernetes to handle the heavy work.

Persistence in will be back by standard RDBMS, whilst event storage was interesting to explore I think the familiarity with PostgreSQL and MySQL will be an appeal to more system administrators compared to a prioretary storage format.  In addition, database administrator can configure further auditing, replication, and run their own projections and reporting without the need for Sitepod to implement an eventually consistent read layer.  Deis Worlflow follows a similar approach in their Kubernetes based PaaS using PostgreSQL

### What's next

More transparency, less client work - pay the bills and enforce a time limit, and hopefully less course changing.  I'm committed to seeing Sitepod become something other than vapourware, hence this rather honest
and premature blog post regarding the project to keep me on my toes.

Get in touch if you want to help!














































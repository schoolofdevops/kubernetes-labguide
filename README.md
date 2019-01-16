# Kubernetes Lab Guide 
---------

|      |      |
| :------------- | :------------- |
| Duration       | 2days     |
| Level     | Intermediate, Advanced |
| Modules          | 10 |
| Flipped Class | Yes |
| Customizable  | Yes |

## Objectives

This course follows a hands on  workshop method and builds container orchestration skills  on top of existing skills learnt in Docker Fundamentals. Participants learn how to built a production grade resilient, self healing infrastructure  with kubernetes. They take the container images built earlier and deploy with the high availability, fault tolerance, scalability provided by kubernetes.

## Flipped Classroom Methodology

This course follows a [Flipped Classroom](https://en.wikipedia.org/wiki/Flipped_classroom) methodology, a blended model where participants come prepared with understanding of concepts by going through video/online content. The class time is maximized to perform practical activities to apply the concepts learnt earlier. Practice activities include taking up a use case and solving it by applying the techniques, or performing nano projects etc.   

To make it successful, participants need to be prepared in advance, by going through  the recommended pre class content to clear the concepts as well as by making sure their systems are prepared with all the software required, before the class. The pre class checklist given below can be handy for this purpose.

## Who is this for ?

  * This course is for someone who has already taken docker fundamentals course/has equivalent knowledge, and would like to learn how to extend it to **orchestrate container** deployments at production scale with kubernetes.
  * If you are a **Operations/Systems** personnel and would like to learn how scalable, fault tolerant and high available infrastructures are built on or off cloud to orchestrate container based deployments, this course it for you.
  * If you are a **developer** and would like to learn how to deploy your application stacks in production, on top of scalable, highly available  and leverage features provided by kubernetes, this course is for you.
  * If you are a **QA**, and if your organization has a staging/QA environment built on kubernetes, and you would like to understand how to leverage it for setting up automated test workflow and learn the primitives offered by kubernetes, this course is for you.
  * You could be developer/operations personnel and be in charge of securing application infrastructure and setup auxiliary services such as  monitoring, centralized  logging etc. this course is for you.

## Who is this not for ?

  * If you are a **advanced user of Kubernetes** already, this course is definitely not for you.
  * If you are looking to start with Kubernetes concepts, this course is not for you as it assumes basic understanding of Kubernetes and COEs. There is a free resource **Kubernetes Quick Dive** which you should go through before attending this workshop.  
  * If you are interested in learning docker/container orchestration on **windows**, this course is not ideal for you as it focuses on linux containers.
  * If you expect to learn a technology which is mainly driven through a **GUI**, this course is not suitable for you. Using kubernetes requires you to write YAML specifications, use command line tools etc.  


## What will you do as part of this course ?

As part of this course you will,  

   * Design a **scalable, resilient, secure** solution for deploying microservices application stack in production using primitives offered by kubernetes.
   * Install and configure a simple(non HA) multi node kubernetes cluster  with **kubeadm**. You would also have a conceptual understanding of  how to build a production quality cluster with high availability, scalability,  redundancy and  security considerations.
   * Learn how applications from a micro services stack interconnect with **services** as well as expose public facing services with various options including nodeport, externalIP, ingress etc.
   * Achieve  **Continuous Deployment** with different release strategies  such as **Zero Downtime, Blue/Green, Canary** etc.  
   * Learn how to manage **persistent storage** in a kubernetes environment
   * Learn about the network primitives including **CNI** plugins, service networking and iptables, ingress etc.
   * Deploy an application which is auto scalable, high available, and resilient to failures.

## What is not covered ?

Even though this course covers many concepts related to kubernetes, since its a very vast topic, it still has the following areas uncovered.

  * Advanced RBAC
  * Cloud specific provisioning and integration
  * HA deployment of a kubernetes cluster with multi masters
  * In depth kubernetes administration
  * Writing Micro Services Applications
  * Alternate container runtimes e.g. rocker/rkt, runc


## Pre Requisites

Following are the pre requisite skills to attend this course. Since its a beginner level course, no prior experience with linux containers is assumed.  

### Courses

You should have attended the following course, or have demonstrable knowledge with the topics included in the following course.

  * Docker Fundamentals

Pre Assessment test will be conducted at the beginning of the course to asses the skills.

### Skills

  * **Docker Basics**  
    * Running Containers
    * Building Images and writing Dockerfiles
    * Docker Compose  
    * Docker Networking and Storage
  * Linux/Unix Systems Fundamentals
  * Familiarity with Command Line Interface (**CLI**)
  * Fundamental knowledge of editors on linux (any one of vi/nano/emacs)
  * Understanding of **YAML** syntax and familiarity with reading/writing basic YAML specifications

### Hardware and Software  Requirements

These are the prerequisites for each attendee.

| Hardware Requirements | Software Requirements |
| :---------------------| :--------------------- |
| Laptop/Desktop with high speed internet connection | Base Operating System : Windows / Mac OSX |
| 8 GB RAM | VirtualBox  |
| 4 CPU Cores | Vagrant |
| 20 GB Disk Space available | ConEmu (Windows Only) |
| | Git for Windows (windows only) |

Lab Setup : Instructions can be found at xxx

## Supporting Content/Materials

Following is the supporting material which will be provided to you before/during the course  

  * Slides (online)  
  * Workshop (online link)  
  * Video Course - XXX by School of Devops  

## Pre Class Checklist

All participants should have completed the following checklist before attending the course .

  * Successfully Completed  **Docker Fundamentals** Course, or have equivalent skills.
  * Verify  your system meets the  hardware pre requisites.
  * Validate the setup : verify all pre requisite software is installed on your system and is functional.
  * Join our [kubernetes channel on gitter](https://gitter.im/schoolofdevops/Kubernetes?utm_source=share-link&utm_medium=link&utm_campaign=share-link)

![Pre Class Checklist](https://github.com/schoolofdevops/course-outlines/blob/master/downloads/docker/kubernetes_bootcamp_checklist.png?raw=true)

## Topics
Following are the topics which would be covered as part of this course. Detailed course outline follows.

  * Design Workshop:  Mapping Mogambo's Micro Services Architecture to Kubernetes
  * Design Workshop: Production grade Kubernetes Architecture design
  * Setup Kubernetes Cluster
  * Advanced Pod Scheduling
  * Publishing Applications with Services
  * Achieving Application High Availability and creating Release Strategies with Deployments
  * Adding configurations with ConfigMaps and Secrets
  * Auto Scaling with Horizontal Pod Autoscaler (HPA)
  * Storage and persistence
  * Adding security with Network Policies

Additional Topics if time permits
  * Additional Controllers
  * Centralized Log Management
  * Cluster Maintenance



## Detailed Course Outline
This is the detailed course outline with day wise list of contents

### Day I

### Introduction and Assessment
  * Trainer, Class and Course Introduction
  * Pre-Assessment Test
  * Formation of Groups: Group of 3/4 participants is formed. Each group
  will collectively come up with the solutions design, which is then discussed
  in the class, which is followed by the implementation of a common solution.

### Module 1:  Design Workshop - Mapping Mogambo's Micro Services Architecture to Kubernetes
  * Introduction to the Use Case
  * Mogambo - The organization and the Application
  * Architecture and Problem Statement
  * Design Workshop
    * This is intended for the participants to come up with the strategy to deploy
    the application stack described with Kubernetes. Participants would come up with
    the structure of the code, map application deployment to kubernetes components e.g. deployments, services, ingress etc. and come up with the roadmap which is followed
    through rest of the workshop.

#### Module 2: Design Workshop: Production grade Kubernetes Architecture  
  * Specification of the problem statement   
  * Design Workshop: This is a design workshop where each group formed earlier comes
  up with a sample design for a high available, fault tolerant, scalable and secure  kubernetes cluster using all the primitives they have learnt as part of the pre course exercise. This is followed by a quick discussion/review.

#### Module 3: Setup and configure kubernetes Cluster
  * Setup 2/3 nodes with VirtualBox/Vagrant/Cloud
  * Install kubernetes with kubeadm
  * CNI Overview
  * Network containers with  CNI using  weave/flannel/calico plugin
  * Configure kubectl admin client
  * Setup Kubernetes Dashboard
  * Overview of kubernetes configurations
  * Kubernetes RBAC overview - contexts, roles, serviceaccounts etc.
  * Setup a project/environment namespace and switch to it

#### Module 4: Advanced Pod Scheduling
  * Defining node/pod affinity and anti affinity
  * Adding health checks
  * Why and how to define pod resource limits
  * Taints and tolerations

#### Module 5: Publishing Applications with Services
  * Picking up the right type of a Service
    * Cluster IP and DNS  
    * nodePort
    * externalIP
    * loadbalancer
    * headless services with Endpoints/ExternalNames
  * Labels and Selectors
  *  Service API Specs
    * Types
    * Traffic Policy
    * Selector
  * Services vs Ingress
  * Creating service resources  for applications and backing services

#### Module 6: Achieving Application High Availability and creating Release Strategies with Deployments
  * Deployments and/vs Replication Controllers
  * Deployment API Specs  
  * Defining replication specs  
  * Defining Deployment Strategies
    * Rolling Update
    * Recreate
    * Batch Size
  * Deployment Operations
    * Describe
    * Rollouts
    * Rollbacks
    * Edit
    * Pause/Unpause
  * Additional Release Strategies
    * Canary Relases
    * Blue/Green Deployments

### Day II

#### Module 7: Adding configurations with ConfigMaps and Secrets
  * ConfigMaps and Secrets Specs
  * Injecting configurations as
    * Environment Variables
    * Files
  * Distributing sensitive data with secrets

#### Module 8: Auto Scaling with Horizontal Pod Autoscaler (HPA)
  * Scaling deployments
    * manually
    * autoscaling
  * HPA Specs
    * Scaling metrics
      * default
      * custom
  * Setting up monitoring for kubernetes
    * heapster
    * influxdb
    * gragana
  * Creating autoscaling configs

#### Module 9: Storage and Persistence
  * Persistent Storage Concepts
    * Storage Drivers
    * PersistentVolumes
    * PersistentVolumeClaims
    * StorageClass
  * Setting up a database with persistent volume

#### Module 10: Securing with Network Policies
  * Network Policy overview
  * Network policy specs
  * Defining network policy for the application stack

### Additional Topics if time permits

#### Advanced Controllers
  * daemonSets
  * statefulSets
  * jobs
  * crons

#### Centralized Logging
  * Centralized logging for kubernetes
  * Setting up a ELK stack

### Cluster Maintenance
  * Cluster Upgrade
  * OS Upgrade
  * Backups and Restore

## Badge : Kubernetes Champion

Successful completion of this course along with the project work and an assessment at the end of this course would qualify you to receive a Docker Champion badge by School of Devops. This badge is compliant with [Mozilla's Open Badge Standards](https://openbadges.org/) and can be added to your social profiles (e.g. LinkedIn) and is verifiable.

Example of a badge is as follows,

![Kubernetes Champion badge by School of Devops](https://github.com/schoolofdevops/course-outlines/blob/master/downloads/docker/kubernetes_champion.png?raw=true)

## Reading List

Here is the list of curated resources which you could refer to to learn about docker before the training and  and get an in depth understanding post training.  

#### Youtube Resources
  * [Kubernetes Introduction by Kelsey Hightower](https://www.youtube.com/watch?v=HlAXp0-M6SY)
  * [Life of a Packet](https://www.youtube.com/watch?v=0Omvgd7Hg1I)

#### Video Courses
  * [Scalable Micro Services with Kubernetes](https://in.udacity.com/course/scalable-microservices-with-kubernetes--ud615) by Carter Morgan, Kelsey Hightower, Gundega Dekena

#### Safaribooks Online
  * Kubernetes Up and Running - Kelsey Hightower

#### Tutorials
  * [Application Example Tutorials](https://github.com/kubernetes/examples)

  * [Application Stack Examples](https://github.com/kubernetes/kubernetes/tree/master/examples)

  * [Official Kubernetes Bootcamp](https://github.com/kubernetes/kubernetes-bootcamp)

  * [Kubernetes in 10 mins](https://blog.alexellis.io/kubernetes-in-10-minutes/)

  * [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

  * [Kubernetes Cluster for the Hobbyist](https://github.com/hobby-kube/guide)

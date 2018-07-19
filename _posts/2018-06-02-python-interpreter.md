---
layout: inner
title: 'Python Interpreter using Docker and Kubernetes'
date: 2018-06-02 08:00:00
categories: Tech
tags: Python Flask Docker Kubernetes Azure GCP
featured_image: 'img/posts/02_docker.png'
project_link: 'https://github.com/shantanu27/Docker-and-Kubernetes'
lead_text: 'Python Interpreter using Docker and Kubernetes'
---

# Python Interpreter using Docker and Kubernetes

<header class = "titleimage">
	<img src="{{ '/img/posts/02_docker.png' | site.baseurl }}" alt="Python Interpreter">
</header>

### Scenario

Traditional applications are run in virtual machines which can be saved, launched, and executed. However, recently new class of virtualization using containers have changed the way applications are built and deployed, especially where there's a need for quick, automated scaling of systems. To test this new 'Docker' technology, a company (fictional) asked me to develop and deploy the backend of their lightweight Python interpreter that could accept code samples from a user, execute the code, and return the result!

### What did I use?
- Python
- Flask
- Docker
- Kubernetes
- Azure and GCP (Google Cloud Platform)

### Setup

The idea was to design a lightweight service that could be easily scaled based on the load. The main advantage of using Docker was its ability to load images, and be up and running within seconds! Another requirement was to make it fault tolerant. That meant it needed to be deployed across multiple cloud providers. The architecture of the project was as shown below: 

![Docker Design]({{ "/img/posts/02_docker_design.jpg" | site.baseurl }})

There were several components in the setup:

**1. Writing Backend Service** 
<br>
The backend service was written using Flask, and it supported the POST API as shown in the figure. It was a simple service which took the statements to execute, ran the statements in the server side Python interpreter, and returned the result. 

**2. Mapping Frontend** 
<br>
The frontend was already provided as part of the project which just needed to map to the backend services. 

**3. Writing Docker files**
<br>
Each component (frontend and backend) needed to be deployed in containers. I wrote the docker files for both frontend and backend which would include the necessary files, install the dependencies, and run the applications once deployed.

**4. Pushing Docker images to Container Registry**
<br>
As shown in the diagram above, there were two clusters - one in Azure and one in GCP. To setup Kubernetes clusters in these two platforms, the Docker images I developed above needed to be pushed to their respective container registries, so that they could pull the images and deploy the cluster. 

**5. Adding Load-Balancing logic to fronted**
<br>
Two cluster deployments in different regions meant that there needed to be a loadbalancer which could direct the traffic to the appropriate backend. I wrote a simple Round-Robin loadbalancer which alternated the requests from the frontend between the two backend containers. 

**6. Autoscaling to deal with demand**
<br>
To handle the demand, I setup autoscaling in the backend and frontend containers. Autoscaling rules are quite easy to specify in the Kubernetes config (yaml) file. By creating a *HorizontalPodAutoscaler* resource and specifying *minReplicas*, *maxReplicas*, *targetCPUUtilizationPercentage* etc., I was able to handle the demand through dynamic scaling.

### Conclusion

It was a really fun project in the Cloud Computing course, giving me lots of experience on handling resources across providers. There were areas which required debugging (of course), but there are several command-line tools provided through Docker and Kubernetes which come in handy.
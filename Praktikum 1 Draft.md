Ia Mgvdliashvili
21.03.2024
## 1. Background
Kubernetes offers a robust container orchestration platform which has been widely adopted in the industry. However, it is also frequently considered as excessive for personal or small scale use, leading to majority of resources concentrating on industry scale deployments and configurations. Because of this, people with personal projects, or maintainers of self-hosted public servers (for example for federated applications) are not able to get out-of-the box access to wide range of benefits that Kubernetes provides. However, managing bare-bones application installations which are not able to scale easily across different nodes can become burdensome for developers or volunteer server administrators. Solutions can be explored which would drive adoption of Cloud Orchestration tools to a more novice audience. I think there is a high potential to find unexplored or under-explored questions in this direction.

## 2. P1 Goals
### 2.1. Set up Hybrid Kubernetes Cluster
Within scope of P1, I will set up a hybrid Kubernetes cluster with two nodes, one on a virtual private server and one on a home server. For this, I will use official documentation as well as published videos. In order to test the cluster, I will deploy 2-3 open source applications.
### 2.2. Work Out the Thesis Topic
In the meantime I will start actively preparing P2 and thesis by familiarizing myself with existing literature and research to propose a strong thesis candidate.
### 2.3. Maybe: Additional Learning
Depending on how smoothly and timely the installation process goes, I would also like to use some of the time to get deeper knowledge about different components of the cluster that might not be relevant immediately during installation.  

## 3. Details
### 3.1. Node Specifications
- A home node: Intel NUC
- For a remote worker and master node, I will be using virtual private servers from Hetzner 
I will add hardware/software details in the beginning of April.

### 3.2 Deployed applications
TBA. I will gather candidates with existing helm charts and straightforward installation instructions. 

> need feedback about communication about side effects. maybe redis or rabbitmq or a postgres server

- performance, communication of microservices - how does the hybrid setting affect it
	- what is the setup good for, what not

### 3.3. Literature 
I plan to first find references to papers in existing books and start reading them. But I have no experience in literature research and would appreciate if you could provide me with some recommended resources on the process itself and direct me to papers which would be relevant to this topic.

## 4. Plan
### 4.1 Work distribution

|                                                                 | Time allocation |
| --------------------------------------------------------------- | --------------- |
| Setting up a Kubernetes cluster                                 | 50%             |
| Read published literature in order to work out the thesis topic | 30%             |
| Maybe: Additional learning                                      | 20%             |

### 4.2 Approximate Timeline

| Weeks | Installation Activities                                                                 | Reading                                          |
| ----- | --------------------------------------------------------------------------------------- | ------------------------------------------------ |
| March |                                                                                         |                                                  |
| April | Install Kubernetes on vps<br>Deploy 2 applications                                      | Get an overview of available literature          |
| May   | Install kubernetes on home node<br>Connect to master node                               | Narrow down possible thesis directions           |
| June  | Possibly: Replicate the process again (reinstall)<br>Ideally: create IaaS configuration | Literature reading in possible thesis directions |

# Docker Networking Design: A Practical Guide

Let us look at a scenario where a fintech company wants you to design the Docker network for three of its containers.

This company wants its web application and database on the same network, but wants its finance payments on another network totally isolated from the network of its web application and database.

Because payment data is extremely sensitive and needs to be isolated from everything else, if all containers share the same network, any container that gets compromised can potentially see and intercept traffic from every other container, including payment data.
A dedicated network means only containers that explicitly need to talk to the payment container are allowed to.

### PDF GUIDE: [DESIGNING DOCKER NETWORKING.pdf](https://github.com/user-attachments/files/32322614/DESIGNING.DOCKER.NETWORKING.pdf)


### WATCH VIDEO WALKTHROUGH HERE: https://youtu.be/HGZSJwy2MqA

## KEY BUSINESS & COMPLIANCE DRIVERS
Security: Cardholder numbers, bank credentials, and transaction logs are kept isolated. Compromising a web or marketing container does not expose payment traffic.

PCI DSS Compliance: Requirement 1 of the Payment Card Industry Data Security Standard (PCI DSS) mandates network segmentation between the Cardholder Data Environment (CDE) and non-sensitive networks.

Blast Radius Reduction: Infrastructure failures, compromises, or traffic spikes on public-facing containers remain localized without impacting payment operations.

Strict Traffic Control: Limits connectivity strictly to required endpoints (e.g., checkout or order microservices), enforcing a Zero Trust container network posture.


## PREREQUISITES
Docker Engine (20.10+ recommended)

Linux Terminal Environment (WSL2 on Windows, Linux, or macOS)

Basic familiarity with Docker CLI execution


## STEP-BY-STEP IMPLEMENTATION

### Phase 1: Environment Setup & Web App Container Deployment

1) Verify Docker Engine status:

<PRE>docker --version</PRE>

2) Deploy the web-app container on the default bridge network:

<PRE>docker run -d --name web-app nginx:latest</PRE>

3) Access the container shell and install network diagnostic utilities (ping):

<PRE>docker exec -it web-app /bin/bash</PRE>

Inside the container shell, run:

<PRE>apt update && apt-get install iputils-ping -y</PRE>
<PRE>ping -V</PRE>

(Keep this terminal session open).


### Phase 2: Deploying Database Container on Default Bridge

1) Open a second terminal window and start the database container:

<PRE>docker run -d --name database nginx:latest</PRE>

2) Verify active running containers:

<PRE>docker ps</PRE>

3) Inspect network allocations on the default bridge network:

<PRE>docker inspect web-app</PRE>
<PRE>docker inspect database</PRE>

web-app IP: 172.17.0.2 (Default bridge)
database IP: 172.17.0.3 (Default bridge)

4) Verify inter-container communication: Switch to the first terminal window (web-app shell) and ping the database container:

<PRE>ping 172.17.0.3</PRE>

Result: Successful responses. Both containers reside on the default bridge driver and can communicate freely.

5) List active Docker networks from the second terminal:

<PRE>docker network ls</PRE>


### Phase 3: Creating Isolated User-Defined Bridge Network

1) Create a dedicated user-defined bridge network (finance-network):

<PRE>docker network create finance-network</PRE>

2) Verify network creation:

<PRE>docker network ls</PRE>

3) Deploy the sensitive finance-payments container attached strictly to finance-network:

<PRE>docker run -d --name finance-payments --network=finance-network nginx:latest</PRE>

4) Verify active containers and inspect network settings:

<PRE>docker ps</PRE>
<PRE>docker inspect finance-payments</PRE>

finance-payments IP: 172.18.0.2 (Network: finance-network)

5) Verify Network Segregation (Security Validation):
Switch back to the first terminal window (web-app shell) and attempt to ping the payment container:

<PRE>ping 172.18.0.2</PRE>

Result: Packet loss / Timeout. Network isolation is enforced. The web-app container cannot reach or inspect traffic on the finance-payments container.


### Phase 4: Host Network Type Demonstration

1) Deploy a fourth container (host-resource) attached directly to the host network namespace:

<PRE>docker run -d --name host-resource --network=host nginx:latest</PRE>

2) Verify running containers:

<PRE>docker ps</PRE>

3) Inspect the host-resource network settings:

<PRE>docker inspect host-resource</PRE>

Result: The IP address field is empty (""). 
The container shares the host machine's IP address and network stack directly rather than operating inside an isolated container bridge network interface.


### Phase 5: Tear Down & Environment Cleanup
Execute the following cleanup commands to decommission all containers, custom networks, and downloaded images:

1) List all containers:

<PRE>docker ps -a</PRE>

2) Stop and remove all running containers:

<PRE>docker stop web-app database finance-payments host-resource</PRE>
<PRE>docker rm web-app database finance-payments host-resource</PRE>

3) Delete the custom Docker bridge network:

<PRE>docker network rm finance-network</PRE>

4) Remove downloaded base images:

<PRE>docker rmi nginx:latest</PRE>

5) Verify clean system state:

<PRE>docker ps -a</PRE>
<PRE>docker images</PRE>
<PRE>docker network ls</PRE>







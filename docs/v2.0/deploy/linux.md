# Deploy Portainer in Linux environments

## Deploy Portainer in Kubernetes

To deploy Portainer within a Kubernetes cluster, you can either use our HELM chart, or our provided manifests.

Note that Portainer CE 2.0 supports Kubernetes version 1.16, 1.17 and 1.18 only.

### Using Helm

First, add the Portainer helm repo running the following:

<pre><code>$ helm repo add portainer https://portainer.github.io/k8s/</code></pre>
<pre><code>$ helm repo update</code></pre>

Then, create the Portainer namespace in your cluster

<pre><code>$ kubectl create namespace portainer</code></pre>

#### For NodePort

Using the following command, Portainer will run in the port 30777

<pre><code>$ helm install -n portainer portainer portainer/portainer</code></pre>

#### For Load Balancer

Using the following command, Portainer will run in the port 9000.

<pre><code>$ helm install -n portainer portainer portainer/portainer --set service.type=LoadBalancer</code></pre>

#### For Ingress

<pre><code>$ helm install -n portainer portainer portainer/portainer --set service.type=ClusterIP</code></pre>

### Using YAML Manifest

First create the Portainer namespace in your cluster

<pre><code>$ kubectl create namespace portainer</code></pre>

#### For NodePort

Using the following command, Portainer will run in the port 30777

<pre><code>$ kubectl apply -n portainer -f https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer.yaml</code></pre>

#### For Load Balancer

<pre><code>kubectl apply -n portainer -f https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer-lb.yaml</code></pre>

## Deploy Portainer in Docker

Portainer is comprised of two elements, the Portainer Server, and the Portainer Agent. Both elements run as lightweight Docker containers on a Docker engine or within a Swarm cluster. Due to the nature of Docker, there are many possible deployment scenarios, however, we have detailed the most common below. Please use the scenario that matches your configuration.

Note that the recommended deployment mode when using Swarm is using the Portainer Agent.

To see the requeriments, please, visit the page of [requirements](/v2.0/deploy/requeriments.md)

### Docker Standalone

Use the following Docker commands to deploy the Portainer Server; note the agent is not needed on standalone hosts, however it does provide additional functionality if used (see portainer and agent scenario below):

<pre><code>$ docker volume create portainer_data</code></pre>

<pre><code>$ docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce</code></pre>

### Docker Swarm

Deploying Portainer and the Portainer Agent to manage a Swarm cluster is easy! You can directly deploy Portainer as a service in your Docker cluster. Note that this method will automatically deploy a single instance of the Portainer Server, and deploy the Portainer Agent as a global service on every node in your cluster.

<pre><code>$ curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml</code></pre>
<pre><code>$ docker stack deploy -c portainer-agent-stack.yml portainer</code></pre>

## Portainer Agent Deployments Only

### Docker Standalone
Run the following command to deploy the Agent in your Docker host.

<pre><code>docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent</code></pre>

### Docker Swarm
Deploy Portainer Agent on a remote LINUX Swarm Cluster as a Swarm Service, run this command on a manager node in the remote cluster.

<pre><code>$ docker service create --name portainer_agent --network portainer_agent_network --publish mode=host,target=9001,published=9001 -e AGENT_CLUSTER_ADDR=tasks.portainer_agent --mode global --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock --mount type=bind,src=//var/lib/docker/volumes,dst=/var/lib/docker/volumes –-mount type=bind,src=/,dst=/host portainer/agent</code></pre>

## Notes

[Contribute to these docs](https://github.com/portainer/portainer-docs/blob/master/contributing.md).

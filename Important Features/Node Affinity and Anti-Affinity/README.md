
# Node Affinity and Anti-Affinity in Google Kubernetes Engine (GKE)

## Introduction

Node Affinity and Anti-Affinity are powerful features in Kubernetes, and by extension in Google Kubernetes Engine (GKE), that allow you to control how pods are scheduled on nodes in a Kubernetes cluster. These features are crucial for optimizing the use of resources, ensuring high availability, and maintaining the desired level of performance.

## What is Node Affinity and Anti-Affinity?

### Node Affinity
Node Affinity is a set of rules used by the scheduler to determine where pods can be placed. It allows you to specify that certain pods should be placed on nodes with specific labels. For instance, you can ensure that a pod runs on a node with a specific hardware type or in a specific geographic location.

### Anti-Affinity
Anti-Affinity, on the other hand, ensures that pods are not co-located on the same nodes. This is particularly useful for high-availability setups, where you want to avoid single points of failure. For example, you can ensure that replicas of a service do not run on the same node.

## Advantages

- Resource Optimization: Ensures that pods are scheduled on nodes best suited for their resource needs.
- High Availability: Spreads out pods across different nodes to avoid single points of failure.
- Load Balancing: Helps in evenly distributing the workload across the cluster.
- Compliance and Policy Enforcement: Enables adherence to specific regulatory, security, or business policies regarding data locality.

## Implementing Node Affinity and Anti-Affinity in GKE

### Prerequisites

- A GKE cluster
- 'kubectl' configured to communicate with your cluster
- Basic understanding of Kubernetes pod manifests and labels

### Commands

``````
export PROJECT_ID="$(gcloud config get-value project -q)"   :   This command sets an environment variable PROJECT_ID with the current active Google Cloud project ID as retrieved from your gcloud configuration.

gcloud container clusters create "playground" --project ${PROJECT_ID} --region "us-west1"   :   Creates a new GKE cluster named "playground" in the "us-west1" region using the project ID specified in the PROJECT_ID environment variable.

gcloud container clusters get-credentials playground --region us-west1 --project ${PROJECT_ID}  :   Configures kubectl to use the credentials for the "playground" GKE cluster, allowing you to interact with your cluster using kubectl commands.
``````

This command will take roughly 5 minutes to complete.
Remember, if you open a new Cloud Shell, you will need to run the `export PROJECT_ID="$(gcloud config get-value project -q)"` command again.


### Step-by-Step Implementation: Node Affinity

#### Label the Nodes:

kubectl get nodes --show-labels     :  List the nodes in your cluster, along with their labels

kubectl label nodes <node-name> disktype=ssd    :   label the nodes in your cluster as needed

kubectl get nodes --show-labels     :   Verify that your chosen node has a disktype=ssd label

#### Define Node Affinity in Pod Spec:

In your pod manifest, define the node affinity under affinity.nodeAffinity. For example:

```
apiVersion: v1
kind: Pod
metadata:
  name: ssd-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx

```

The above code defines a configuration for a Kubernetes pod named 'ssd-pod'. It specifies that the pod should only be scheduled on nodes labeled with 'disktype: ssd', ensuring the pod runs on a node with an SSD. The pod will run a container using the 'nginx' image.

## Apply the Manifest:

kubectl apply -f <pod-manifest.yaml>     :   Apply your pod configuration using kubectl command

#### Verify Scheduling:

Verify that the pods are scheduled according to the defined affinity rule

kubectl get pods -o wide


### Step-by-Step Implementation: Node Anti-Affinity

#### Label the Nodes:

kubectl get nodes --show-labels     :  List the nodes in your cluster, along with their labels

kubectl label nodes <node-name> app=webserver   :   Label a Kubernetes node with a specific label to be used in the anti-affinity rule

kubectl get nodes --show-labels     :   Verify that your chosen node has a app=webserver label

#### Define Anti-Affinity:

To prevent pods from being co-located, use podAntiAffinity. For example:

```

kubectl run webserver-pod --image=nginx --labels="app=webserver"

OR


apiVersion: v1
kind: Pod
metadata:
  name: anti-affinity-pod
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - webserver
        topologyKey: "kubernetes.io/hostname"
  containers:
  - name: myapp
    image: nginx

```

The above code defines a configuration for a Kubernetes pod named "anti-affinity-pod" which uses a container based on the "nginx" image. It specifies a rule under "podAntiAffinity" to prevent this pod from being scheduled on the same host as other pods labeled with "app: webserver". This ensures that instances of this pod and those of the webserver app are not running on the same physical or virtual machine in the Kubernetes cluster.

#### Apply the Manifest:

kubectl apply -f <pod-manifest.yaml>     :   Apply your pod configuration using kubectl command

#### Verify Scheduling:

Verify that the pods are scheduled according to the defined anti-affinity rule:

kubectl get pods -o wide

### Best Practices
- Use node affinity to target specific node characteristics like CPU, memory, or custom labels.
- Utilize anti-affinity for stateful applications where each instance should be isolated from others.
- Regularly review and update your affinity rules to align with the changing cluster environment.

## Conclusion
Node Affinity and Anti-Affinity in GKE are crucial for effective cluster management, optimizing resource utilization, and ensuring high availability. By properly implementing these features, you can significantly enhance the performance and reliability of your applications running on GKE.
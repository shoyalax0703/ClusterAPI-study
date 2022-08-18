Hands on demo using Cluster API. 

What is Cluster API?  
https://cluster-api.sigs.k8s.io/introduction.html. 

Common Prerequisites. 
* Install and setup kubectl in your local environment.  
* Install Kind and Docker. 


Run the following command to create a kind config file for allowing the Docker provider to access Docker on the host:  

```
cat > kind-cluster-with-extramounts.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraMounts:
    - hostPath: /var/run/docker.sock
      containerPath: /var/run/docker.sock
EOF
```

Then follow the instruction for your kind version using 
kind create cluster --config kind-cluster-with-extramounts.yaml 
to create the management cluster using the above file.

output is below
￼

kubectl get all -A 
￼

Install clusterctl with homebrew on macOS and linux
Install the latest release using homebrew:

brew install clusterctl
Test to ensure the version you installed is up-to-date:

clusterctl version

Now that we’ve got clusterctl installed and all the prerequisites in place,

follwing command enable your cluster into management cluster.

The Docker provider requires the ClusterTopology feature to deploy ClusterClass-based clusters. We are only supporting ClusterClass-based cluster-templates in this quickstart as ClusterClass makes it possible to adapt configuration based on Kubernetes version. This is required to install Kubernetes clusters < v1.24 and for the upgrade from v1.23 to v1.24 as we have to use different cgroupDrivers depending on Kubernetes version.

# Enable the experimental Cluster topology feature.
export CLUSTER_TOPOLOGY=true

# Initialize the management cluster
clusterctl init --infrastructure docker

outpot is below
￼

次のコマンドでworkload cluster を作成可能
# The list of service CIDR, default ["10.128.0.0/12"]
export SERVICE_CIDR=["10.96.0.0/12"]

# The list of pod CIDR, default ["192.168.0.0/16"]
export POD_CIDR=["192.168.0.0/16"]

# The service domain, default "cluster.local"
export SERVICE_DOMAIN="k8s.test"

It is also possible but not recommended to disable the per-default enabled Pod Security Standard:

export ENABLE_POD_SECURITY_STANDARD="false"

下のコマンドでcluster 情報をyamlに変換、これでkubectl apply -f でclusterをpod,deploymentと同じように作成可能 
clusterctl generate cluster capi-quickstart --flavor development \
  --kubernetes-version v1.24.0 \
  --control-plane-machine-count=3 \
  --worker-machine-count=3 \
  > capi-quickstart.yaml

kubectl apply -f capi-quickstart.yaml
output is below

￼

kubectl get kubeadmcontrolplane
￼

clusterctl get kubeconfig capi-quickstart > capi-quickstart.kubeconfig

# Point the kubeconfig to the exposed port of the load balancer, rather than the inaccessible container IP.

sed -i -e "s/server:.*/server: https:\/\/$(docker port capi-quickstart-lb 6443/tcp | sed "s/0.0.0.0/127.0.0.1/")/g" ./capi-quickstart.kubeconfig

kubectl --kubeconfig=./capi-quickstart.kubeconfig \
  apply -f https://docs.projectcalico.org/v3.21/manifests/calico.yaml

kubectl --kubeconfig=./capi-quickstart.kubeconfig get nodes
￼

Clean up 
kubectl delete cluster capi-quickstart
kind delete cluster



The Cluster Lifecycle SIG is the Special Interest Group


一回はできたんよ
でも何故か、できなくなった、これは待ってから実施した方がいい！

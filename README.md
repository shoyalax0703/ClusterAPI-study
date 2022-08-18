#Hands on demo using Cluster API. 

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
```
kind create cluster --config kind-cluster-with-extramounts.yaml 
```
to create the management cluster using the above file.

output is below. 

```
kubectl get all -A
```

<img width="675" alt="Creating cluster kind" src="https://user-images.githubusercontent.com/66551005/185342782-e6aeff80-4aca-4626-8b28-709c30ec317a.png">

Install clusterctl with homebrew on macOS and linux. 
Install the latest release using homebrew:  
```
brew install clusterctl
```
Test to ensure the version you installed is up-to-date:  
```
clusterctl version
```

##Now that we’ve got clusterctl installed and all the prerequisites in place.  

follwing command enable your cluster into management cluster. 

# Enable the experimental Cluster topology feature.  
```
export CLUSTER_TOPOLOGY=true
```
# Initialize the management cluster. 
```
clusterctl init --infrastructure docker
```

outpot is below
<img width="1474" alt="Screen Shot 2022-08-18 at 14 27 19" src="https://user-images.githubusercontent.com/66551005/185344279-3080e88d-44ec-4783-9f09-a2f5591fea8e.png">


次のコマンドでworkload cluster を作成可能. 
```
# The list of service CIDR, default ["10.128.0.0/12"]
export SERVICE_CIDR=["10.96.0.0/12"]

# The list of pod CIDR, default ["192.168.0.0/16"]
export POD_CIDR=["192.168.0.0/16"]

# The service domain, default "cluster.local"
export SERVICE_DOMAIN="k8s.test"

It is also possible but not recommended to disable the per-default enabled Pod Security Standard:
export ENABLE_POD_SECURITY_STANDARD="false"
```

Generate cluster info to capi-quickstart.yaml. 
```
clusterctl generate cluster capi-quickstart --flavor development \
  --kubernetes-version v1.24.0 \
  --control-plane-machine-count=3 \
  --worker-machine-count=3 \
  > capi-quickstart.yaml
```

```
kubectl apply -f capi-quickstart.yaml
```

output is below. 
<img width="1474" alt="Screen Shot 2022-08-18 at 14 39 24" src="https://user-images.githubusercontent.com/66551005/185344783-6231ab77-d602-4e7c-86b5-a7b504cdea3d.png">

```
kubectl get kubeadmcontrolplane
```
<img width="1653" alt="Screen Shot 2022-08-18 at 14 41 55" src="https://user-images.githubusercontent.com/66551005/185344853-c5da9e4a-78e4-4137-837f-1ac67292f44d.png">


clusterctl get kubeconfig capi-quickstart > capi-quickstart.kubeconfig. 

Point the kubeconfig to the exposed port of the load balancer, rather than the inaccessible container IP. 

```
sed -i -e "s/server:.*/server: https:\/\/$(docker port capi-quickstart-lb 6443/tcp | sed "s/0.0.0.0/127.0.0.1/")/g" ./capi-quickstart.kubeconfig
```

we need to have CNI to connect workload cluster node just created and we use Calico. 

```
kubectl --kubeconfig=./capi-quickstart.kubeconfig \
  apply -f https://docs.projectcalico.org/v3.21/manifests/calico.yaml
```
Confirm we could make it well. 
```
kubectl --kubeconfig=./capi-quickstart.kubeconfig get nodes
```
<img width="1653" alt="Screen Shot 2022-08-18 at 14 58 05" src="https://user-images.githubusercontent.com/66551005/185345257-7e1fb132-4293-4a7d-be4b-18242e74ea0e.png">


##Clean up. 
```
kubectl delete cluster capi-quickstart
kind delete cluster
```

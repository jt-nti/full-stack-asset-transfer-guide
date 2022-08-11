# Kubernetes on a Multipass VM 

## Provision a VM

```shell
multipass launch \
  --name        fabric-dev \
  --disk        80G \
  --cpus        8 \
  --mem         8G \
  --cloud-init  infrastructure/multipass-cloud-config.yaml

multipass mount $PWD/config fabric-dev:/mnt/config

```

## Cluster Setup 

```shell
multipass shell fabric-dev
```

```shell
git clone https://github.com/jkneubuh/full-stack-asset-transfer-guide.git -b feature/puff
cd ~/full-stack-asset-transfer-guide 

# export TEST_NETWORK_INGRESS_DOMAIN=$(hostname -I  | cut -d ' ' -f 1 | tr -s '.' '-').nip.io
export MULTIPASS_IP=$(hostname -I | cut -d ' ' -f 1) 
export CONTAINER_REGISTRY_ADDRESS=0.0.0.0
export KIND_API_SERVER_ADDRESS=$MULTIPASS_IP

# Create a Kubernetes cluster in Docker, nginx ingress, and local docker registry:
just -f cloud.justfile kind

# Install fabric-operator CRDs
kubectl apply -k https://github.com/hyperledger-labs/fabric-operator.git/config/crd

# Copy the kube config to the host OS via volume share 
cp ~/.kube/config /mnt/config/multipass-kube-config.yaml 

```

```shell
# Observe the target Kubernetes namespace 
k9s -n test-network 
```


## Connect to the Kube API Controller 

Start a new shell on the host OS: 

```shell
export MULTIPASS_IP=$(multipass info fabric-dev --format json | jq -r .info.\"fabric-dev\".ipv4[0])
export TEST_NETWORK_INGRESS_DOMAIN=$(echo $MULTIPASS_IP | tr -s '.' '-').nip.io

# Configure kubectl to connect to the API server on the multipass instance 
cp config/multipass-kube-config.yaml ~/.kube/config 

kubectl cluster-info

echo "Connecting to Fabric network domain $TEST_NETWORK_INGRESS_DOMAIN"

```


### Guide 

Up : [Select a Kube](10-kube.md)

Prev : [Setup](00-setup.md)

Next : [Deploy a Fabric Network](20-fabric.md)

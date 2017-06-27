# Cloud Infrastructure Provisioning - Google Cloud Platform

This lab will walk you through provisioning the compute instances required for running a H/A Kubernetes cluster. A total of 6 virtual machines will be created.

The guide assumes you've installed the Azure CLI 2.0, and will be creating resources in the eastus region, within a resource group named kubernetes. To create this resource group, simply run the following command:

```
az group create -n kubernetes -l eastus
```
After completing this guide you should have the following compute instances:

```
az vm list --query "[].{name:name,provisioningState:provisioningState}"`
```

````
```shell
Name         ProvisioningState
-----------  -------------------
controller0  Succeeded
controller1  Succeeded
controller2  Succeeded
worker0      Succeeded
worker1      Succeeded
worker2      Succeeded
```
````

> All machines will be provisioned with fixed private IP addresses to simplify the bootstrap process.

To make our Kubernetes control plane remotely accessible, a public IP address will be provisioned and assigned to a Load Balancer that will sit in front of the 3 Kubernetes controllers.

## Setup Networking


Create a virtual network and subnet for the Kubernetes cluster:

```
az network vnet create -g kubernetes \
  -n kubernetes-vnet \
  --address-prefix 10.240.0.0/16 \
  --subnet-name kubernetes-subnet
```

### Create Firewall Rules

Create a firewall ("network security group"), assign it to the subnet, and configure it to allow the necessary incoming traffic:

```
az network nsg create -g kubernetes -n kubernetes-nsg
```

```
az network vnet subnet update -g kubernetes \
  -n kubernetes-subnet \
  --vnet-name kubernetes-vnet \
  --network-security-group kubernetes-nsg
```

```
az network nsg rule create -g kubernetes \
  -n kubernetes-allow-ssh \
  --access allow \
  --destination-address-prefix '*' \
  --destination-port-range 22 \
  --direction inbound \
  --nsg-name kubernetes-nsg \
  --protocol tcp \
  --source-address-prefix '*' \
  --source-port-range '*' \
  --priority 1000
```

```
az network nsg rule create -g kubernetes \
  -n kubernetes-allow-api-server \
  --access allow \
  --destination-address-prefix '*' \
  --destination-port-range 6443 \
  --direction inbound \
  --nsg-name kubernetes-nsg \
  --protocol tcp \
  --source-address-prefix '*' \
  --source-port-range '*' \
  --priority 1001
```

```
az network nsg rule create -g kubernetes \
  -n kubernetes-allow-api-server \
  --access allow \
  --destination-address-prefix '*' \
  --destination-port-range 3389 \
  --direction inbound \
  --nsg-name kubernetes-nsg \
  --protocol tcp \
  --source-address-prefix '*' \
  --source-port-range '*' \
  --priority 1002
```


```
az network nsg rule list -g kubernetes --nsg-name kubernetes-nsg --query "[].{Name:name, Port:destinationPortRange}" -otable
```

```
Name                           Port
---------------------------  ------
kubernetes-allow-ssh             22
kubernetes-allow-api-server    6443
kubernetes-allow-api-3389      3389
```

### Create the Kubernetes Public Address

Create a public IP address that will be used by remote clients to connect to the Kubernetes control plane:

```
az network lb create -g kubernetes \
  -n kubernetes-lb \
  --backend-pool-name kubernetes-lb-pool \
  --public-ip-address kubernetes-pip \
  --public-ip-address-allocation static
```

```
az network public-ip show -g kubernetes -n kubernetes-pip --query "{ name: name, location: location, ipAddress: ipAddress, provisioningState: provisioningState}" -o table
```

```
Name            Location    IpAddress       ProvisioningState
--------------  ----------  --------------  -------------------
kubernetes-pip  eastus      XXX.XX.XXX.XXX  Succeeded
```

## Provision Virtual Machines

All the VMs in this lab will be provisioned using Ubuntu 16.04 mainly because it runs a newish Linux kernel with good support for Docker.

### Virtual Machines

#### Kubernetes Controllers

```
az vm availability-set create -g kubernetes -n controller-as
```

```
for num in {0..2}; do
    echo "[Controller ${num}] Creating public IP..."
    az network public-ip create -n controller${num}-pip -g kubernetes > /dev/null

    echo "[Controller ${num}] Creating NIC..."
    az network nic create -g kubernetes \
        -n controller${num}-nic \
        --private-ip-address 10.240.0.1${num} \
        --public-ip-address controller${num}-pip \
        --vnet kubernetes-vnet \
        --subnet kubernetes-subnet \
        --ip-forwarding \
        --lb-name kubernetes-lb \
        --lb-address-pools kubernetes-lb-pool > /dev/null

    echo "[Controller ${num}] Creating VM..."
    az vm create -g kubernetes \
        -n controller${num} \
        --image Canonical:UbuntuServer:16.04-LTS:latest \
        --nics controller${num}-nic \
        --availability-set controller-as \
        --nsg '' > /dev/null
done
```

CONTINUE HERE!!!!!!!!!!!!!!!!!!! MUST RUN ABOVE

#### Kubernetes Workers

```
az vm availability-set create -g kubernetes -n worker-as
```

```
for num in {0..2}; do
    echo "[Worker ${num}] Creating public IP..."
    az network public-ip create -n worker${num}-pip -g kubernetes > /dev/null

    echo "[Worker ${num}] Creating NIC..."
    az network nic create -g kubernetes \
        -n worker${num}-nic \
        --private-ip-address 10.240.0.2${num} \
        --public-ip-address worker${num}-pip \
        --vnet kubernetes-vnet \
        --subnet kubernetes-subnet \
        --ip-forwarding > /dev/null

    echo "[Worker ${num}] Creating VM..."
    az vm create -g kubernetes \
        -n worker${num} \
        --image Canonical:UbuntuServer:16.04.0-LTS:16.04.201609210 \
        --nics worker${num}-nic \
        --availability-set worker-as \
        --nsg '' > /dev/null
done
```
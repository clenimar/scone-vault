# A demo how to run HashiCorp Vault inside Intel SGX using SCONE 

To perform this demo, you run the following command:
(see https://sconedocs.github.io/vault/)

```bash
source sgxdevice.sh && determine_sgx_device
docker-compose up
```

Ensure to execute

```bash
docker-compose down
```

before starting it with *up* again. Note that the script `start_vault.sh` is used to start Vault server and inject some secrets used for Nginx.

## Details

You can perform the individual steps manually as described below.

Determine your SGX device:

```bash
source sgxdevice.sh && determine_sgx_device
```

Run the demo container using docker-compose:

```bash
docker-compose run scone-vault-nginx sh
```

Go to the deployment directory:

```bash
 cd /build_dir/
```

Install dependencies:

```bash
 ./install-deps.sh
```

Now, run a benchmark to test SCONE Vault getting configuation for Nginx.

```bash
./bench.sh
```

## Running on Azure Confidential VMs (DCsv2-series)

You can follow the same steps described above to run this demo on an Azure Confidential VM:

1. Create a DCsv2-Series Virtual Machine. Please refer to the [official Azure documentation to create one via the Azure Portal](https://docs.microsoft.com/en-us/azure/confidential-computing/quick-create-portal). The recommended VM Size is `DC4s_v2` (4 vCPUs and 16 GiB of RAM).

> NOTE: You don't need to install the SGX driver, as these machines already come with an SGX DCAP driver installed available in `/dev/sgx`.

2. Inside of the virtual machine, install docker and docker-compose.

3. Clone this GitHub repo.

4. Determine the SGX device for docker-compose.

```bash
source sgxdevice.sh && determine_sgx_device
```

5. Run the demo:

```bash
docker-compose up
```

6. Clean-up:

```bash
docker-compose down
```

## Running on Azure Kubernetes Service

To run this demo on a confidential AKS cluster, follow this set of steps.

1. Setup your Kubernetes cluster. You can either create a new cluster with a Confidential Computing-enabled nodepool, or add a confidential nodepool to an existing cluster.

You can check Azure official docs to see how to [create a confidential node pool or upgrade your existing cluster](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-aks-get-started).

But the overall steps are (to create a new cluster):

```bash
# Authenticate to the Azure CLI
az login
# Create a resource group
az group create --name myResourceGroup --location westus
# Create a new cluster. This configures a system pool (does not have SGX) with 1 node
az aks create -g myResourceGroup --name myAKSCluster --generate-ssh-keys --enable-addon confcom -c 1
# Create a confidential node pool with 2 DC2s_v2 nodes
az aks nodepool add --cluster-name myAKSCluster --name confcompool1 --resource-group myResourceGroup --node-vm-size Standard_DC2s_v2 -c 2
# Get credentials to configure kubectl
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

This demo assumes that the Azure SGX Device Plugin is installed in the cluster. New Confidential Computing-enabled node pools already come with the Azure SGX Device Plugin installed, which configures access to the SGX driver in the nodes.

Access to the SGX driver is requested in the Kubernetes manifest of the application:

```yaml
resources:
  limits:
    kubernetes.azure.com/sgx_epc_mem_in_MiB: 4
```

You can make sure the Azure SGX Device Plugin is installed by looking for its pods:

```bash
$ kubectl get pods -nkube-system | grep sgx-plugin
sgx-plugin-fb5zn                      1/1     Running   0          2d20h
sgx-plugin-xftgm                      1/1     Running   0          2d20h
```

If you have other SGX device plugin installed, you can change the `resources.limits` field of `deploy-aks/vault/vault-statefulset.yml` and `deploy-aks/nginx/nginx-deployment.yml` accordingly (e.g., for the [SCONE SGX Device Plugin](https://sconedocs.github.io/helm_sgxdevplugin/)):

```yaml
resources:
  limits:
    sgx.k8s.io/sgx: 1
```

2. Configure Kubernetes to pull private images through an `imagePullSecret`. This demo assumes that you have a secret named "sconeapps".

Replace with actual credentials and run:

```bash
export REGISTRY_USER=foo
export REGISTRY_PASSWORD=token
export REGISTRY_USER_EMAIL=foo@example.com
kubectl create secret docker-registry sconeapps --docker-server=registry.scontain.com:5050 --docker-username=$REGISTRY_USER --docker-password=$REGISTRY_TOKEN --docker-email=$REGISTRY_USER_EMAIL
```

Check [Kubernetes official docs on imagePullSecrets](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/).

3. Start Vault.

```bash
kubectl create -f deploy-aks/vault
```

This will install Vault as a DaemonSet, alongside a Secret for the NGINX certificates, a ConfigMap for the NGINX configuration files, and a Service to expose the Vault server through the name `scone-vault` inside of the cluster.

To see the pods currently running:

```bash
kubectl get pods
```

To check Vault server logs:

```bash
kubectl logs scone-vault-0
```

4. Start NGINX

```bash
kubectl create -f deploy-aks/nginx
```

Then check the logs for the NGINX pod. Find the NGINX pod name:

```bash
kubectl get pods
```

And see the logs:

```bash
kubectl logs <nginx-pod-name>
```

5. Clean up:

```bash
# Delete NGINX deployment
kubectl delete -f deploy-aks/nginx
# Delete Vault deployment
kubectl delete -f deploy-aks/vault
# Delete confidential nodepool on AKS
az aks nodepool delete --cluster-name myAKSCluster --name confcompool1 --resource-group myResourceGroup
# Delete AKS cluster
az aks delete --resource-group myResourceGroup --name myAKSCluster
```

## Contacts

Send email to info@scontain.com

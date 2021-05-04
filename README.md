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

## Contacts

Send email to info@scontain.com

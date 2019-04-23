# This terraform code is customised not to deploy Consul.

The Packer config is tweaked as the one which came with the official repo did not work out of the box, same goes for install-vault script.

Terraform config is tweaked to deploy vault without consul also made changes to vault-cluster module to add `primary = true` field in network_profile.ip_configuration after encountering the following error
`Error: module.vault_servers.azurerm_virtual_machine_scale_set.vault_with_load_balancer: "network_profile.0.ip_configuration.0.primary": required field is not set`


Steps: 
1. Source .env with required vars. Ref: .env-example file
2. Run packer build vault-examples-helper under examples/vault-consul-image
   this will provision the image with the tls certs which came with official repo
   make sure to provision your own for production use.
3. Pass the image_uri from above packer build to terrafrom as env var "TF_VAR_image_uri"
4. Run `terraform init` and `terraform plan` to see what resources will be created. 
5. Run `terraform apply` if you are happy with the above plan.

##
Notes from official module.
# Vault Azure Module

This repo contains a Module to deploy a [Vault](https://www.vaultproject.io/) cluster on 
[Azure](https://azure.microsoft.com/) using [Terraform](https://www.terraform.io/). Vault is an open source tool for 
managing secrets. This Module uses [Azure Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-dotnet-how-to-use-blobs) as a [storage 
backend](https://www.vaultproject.io/docs/configuration/storage/index.html) and a [Consul](https://www.consul.io) 
server cluster as a [high availability backend](https://www.vaultproject.io/docs/concepts/ha.html):

![Vault architecture](https://raw.githubusercontent.com/hashicorp/terraform-azurerm-vault/master/_docs/architecture.png)

This Module includes:

* [install-vault](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/install-vault): This module can be used to install Vault. It can be used in a 
  [Packer](https://www.packer.io/) template to create a Vault 
  [Azure Manager Image](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/build-image-with-packer).

* [run-vault](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/run-vault): This module can be used to configure and run Vault. It can be used in a 
  [Custom Data](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/classic/inject-custom-data) 
  script to fire up Vault while the server is booting.

* [vault-cluster](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/vault-cluster): Terraform code to deploy a cluster of Vault servers using an [Scale Set]
(https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-create).
   
* [private-tls-cert](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/private-tls-cert): Generate a private TLS certificate for use with a private Vault 
  cluster.
   
* [update-certificate-store](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/update-certificate-store): Add a trusted, CA public key to an OS's 
  certificate store. This allows you to establish TLS connections to services that use this TLS certs signed by this
  CA without getting x509 certificate errors.
   



## What's a Module?

A Module is a canonical, reusable, best-practices definition for how to run a single piece of infrastructure, such 
as a database or server cluster. Each Module is created primarily using [Terraform](https://www.terraform.io/), 
includes automated tests, examples, and documentation, and is maintained both by the open source community and 
companies that provide commercial support. 

Instead of having to figure out the details of how to run a piece of infrastructure from scratch, you can reuse 
existing code that has been proven in production. And instead of maintaining all that infrastructure code yourself, 
you can leverage the work of the Module community and maintainers, and pick up infrastructure improvements through
a version number bump.
 
 
 
## Who maintains this Module?

This Module is maintained by [Gruntwork](http://www.gruntwork.io/). If you're looking for help or commercial 
support, send an email to [modules@gruntwork.io](mailto:modules@gruntwork.io?Subject=Vault%20Module). 
Gruntwork can help with:

* Setup, customization, and support for this Module.
* Module for other types of infrastructure, such as VPCs, Docker clusters, databases, and continuous integration.
* Module that meet compliance requirements, such as HIPAA.
* Consulting & Training on AWS, Terraform, and DevOps.



## How do you use this Module?

Each Module has the following folder structure:

* [root](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/): The root folder contains an example of running a public Vault cluster on Azure
* [modules](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules): This folder contains the reusable code for this Module, broken down into one or more modules.
* [examples](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/examples): This folder contains examples of how to use the modules.
* [test](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/test): Automated tests for the modules and examples.

Click on each of the modules above for more details.

To deploy Vault with this Blueprint, you will need to deploy two separate clusters: one to run 
[Consul](https://www.consul.io/) servers (which Vault uses as a [high availability 
backend](https://www.vaultproject.io/docs/concepts/ha.html)) and one to run Vault servers. 

To deploy the Consul server cluster, use the [Consul Azure Module](https://github.com/hashicorp/terraform-azurerm-consul). 

To deploy the Vault cluster:

1. Create an Azure Image that has Vault installed (using the [install-vault module](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/install-vault)) and the Consul
   agent installed (using the [install-consul 
   module](https://github.com/hashicorp/terraform-azurerm-consul/tree/master/modules/install-consul)). Here is an 
   [example Packer template](https://github.com/hashicorp/terraform-azurerm-consul/tree/master/examples/consul-image). 
   
1. Deploy that Azure Image across a Scale Set in a private subnet using the Terraform [vault-cluster 
   module](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/vault-cluster). 

1. Execute the [run-consul script](https://github.com/hashicorp/terraform-azurerm-consul/tree/master/modules/run-consul)
   with the `--client` flag during boot on each Instance to have the Consul agent connect to the Consul server cluster. 

1. Execute the [run-vault](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/run-vault) script during boot on each Instance to create the Vault cluster. 

1. If you only need to access Vault from inside your Azure account (recommended), run the [install-dnsmasq 
   module](https://github.com/hashicorp/terraform-azurerm-consul/tree/master/modules/install-dnsmasq) on each server, and 
   that server will be able to reach Vault using the Consul Server cluster as the DNS resolver (e.g. using an address 
   like `vault.service.consul`). See the [main example](https://github.com/hashicorp/terraform-azurerm-consul/tree/master/MAIN.md) for working 
   sample code.

1. Head over to the [How do you use the Vault cluster?](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/modules/vault-cluster#how-do-you-use-the-vault-cluster) guide
   to learn how to initialize, unseal, and use Vault.

 
## How do I contribute to this Module?

Contributions are very welcome! Check out the [Contribution Guidelines](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/CONTRIBUTING.md) for instructions.



## How is this Module versioned?

This Module follows the principles of [Semantic Versioning](http://semver.org/). You can find each new release, 
along with the changelog, in the [Releases Page](../../releases). 

During initial development, the major version will be 0 (e.g., `0.x.y`), which indicates the code does not yet have a 
stable API. Once we hit `1.0.0`, we will make every effort to maintain a backwards compatible API and use the MAJOR, 
MINOR, and PATCH versions on each release to indicate any incompatibilities. 



## License

This code is released under the Apache 2.0 License. Please see [LICENSE](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/LICENSE) and [NOTICE](https://github.com/hashicorp/terraform-azurerm-vault/tree/master/NOTICE) for more 
details.


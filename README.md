# Módulo de Azure Kubernetes Services (AKS)
[![HCL](https://img.shields.io/badge/language-HCL-blueviolet)](https://www.terraform.io/)
[![Azure](https://img.shields.io/badge/provider-Azure-blue)](https://registry.terraform.io/providers/hashicorp/azurerm/latest)

Modulo desenvolvido para facilitar a criação de clusters de AKS com Grafana e Prometheus.


## Compatibilidade de Versão

| Versão do Modulo | Versão Terraform | Versão AzureRM |
|----------------|-------------------| --------------- |
| >= 0.0.2       | >= 1.0.x | >= 3.16.0         |


## Pré-requisitos

Para utilizar este módulo é necessário atender os requisitos abaixo.

- AZ CLI instalado
- Conta no Azure
- Credencial para autenticação no AKS


## Especificando versão

Para evitar que seu código receba atualizações automáticas do modulo, é preciso informar na chave `source` do bloco do module a versão desejada, utilizando o parametro `?ref=***` no final da url.


## Exemplo de uso

Criar o Service Principal para autenticação do cluster kubernetes com o comando abaixo.

```azcli
az ad sp create-for-rbac --name "<rbac-name>" --role "Contributor" --scopes "/subscriptions/<sub>"
```

**\<rbac-name>** - Nome da credêncial usada pelo Terraform

**\<sub>** - ID da subscription no Azure que será utilizado

Criar as variáveis abaixo de acordo com o Service Principal criado anteriormente.

```
  client_id     = <client ID>
  client_secret = <client Secret>
```

Criar um arquivo local chamado \<file>.tf e inserir o script abaixo.

```hcl
provider "kubernetes" {
  host = module.lab-k8s.k8s_host

  client_certificate     = base64decode(module.lab-k8s.k8s_client_certificate)
  client_key             = base64decode(module.lab-k8s.k8s_client_key)
  cluster_ca_certificate = base64decode(module.lab-k8s.k8s_cluster_ca_certificate)
}

provider "helm" {
  kubernetes {
    host = module.lab-k8s.k8s_host

    client_certificate     = base64decode(module.lab-k8s.k8s_client_certificate)
    client_key             = base64decode(module.lab-k8s.k8s_client_key)
    cluster_ca_certificate = base64decode(module.lab-k8s.k8s_cluster_ca_certificate)
  }
}

module "lab-k8s" {
  source                = "git::https://github.com/fabioschagas/terraform-module-aks.git?ref=v0.0.2"
  env                   = "dev"
  project_name          = "kubernetes"
  node_pool_name        = "default"
  k8s_node_vm_size      = "Standard_D2_v2"
  k8s_node_count        = 2
  k8s_node_os_disk_size = 30
  enable_grafana        = false
  enable_prometheus     = true
}
```

O provider de kubernetes e do helm são necessários para o código, colocar junto com o provider do azurerm.

Após criação dos arquivos, execute os comandos abaixo para realizar o deploy do cluster de AKS.

```hcl
terraform init
terraform plan
terraform apply
```


## Entrada de Valores

| Nome | Descrição | Tipo | Padrão | Requerido |
|------|-------------|------|---------|:--------:|
| env | nome do ambiente da sua aplicação, só pode conter 3 letras | `string` | n/a | Sim |
| project_name | nome do seu projeto do cluster de kubernetes | `string` | n/a | Sim |
| node_pool_name | nome do node_pool que ficarão os nodes | `string` | n/a | Sim |
| k8s_node_vm_size | size das VMs do cluster de kubernetes | `string` | n/a | Sim |
| k8s_node_count | quantidade de nodes que terá esse cluster | `number` | n/a | Sim | 
| k8s_node_os_disk_size | size dos discos de SO das VMs | `number` | n/a | Sim | 
| enable_grafana | habilitar ou não o deploy do chart do grafana | `bool` | `false` | Não |
| enable_prometheus | habilitar ou não o deploy do chart do prometheus | `bool` | `false` | Não |


## Saída de Valores

| Nome | Descrição |
|------|-------------|
| resource_group_name | no do resource group |
| kubernetes_cluster_name | nome do cluster de aks |
| k8s_client_certificate | saída do client certificate para acesso ao cluster de aks, dado sensível |
| k8s_client_key | saída do client key para acesso ao cluster de aks, dado sensível |
| k8s_cluster_ca_certificate | saída do client certificate ca para acesso ao cluster de aks, dado sensível |
| k8s_host | endpoint do cluster aks |

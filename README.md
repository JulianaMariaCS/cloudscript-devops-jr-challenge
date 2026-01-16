# DevOps Challenge — Provisionamento de EKS com Terraform

## 📌 Visão Geral

Este repositório provisiona uma infraestrutura básica na AWS utilizando **Terraform**, com foco em boas práticas de **segurança, alta disponibilidade e automação**.  
A solução contempla a criação de uma VPC customizada, um cluster Amazon EKS e seus respectivos recursos de suporte.

Todo o desenvolvimento foi realizado em **branch separada**, garantindo estabilidade da branch principal e facilitando a revisão do código por meio de Pull Requests.

Esta implementação tem como objetivo atender a um desafio técnico e não representa uma arquitetura final de produção, embora adote boas práticas sempre que possível.

---

## 📐 Arquitetura

A arquitetura proposta possui as seguintes características:

- VPC customizada distribuída em **duas Availability Zones**
- Subnets públicas e privadas
- Subnets privadas destinadas exclusivamente aos nós do EKS
- NAT Gateway para acesso seguro à internet a partir das subnets privadas
- Cluster Amazon EKS com Managed Node Groups
- Autenticação e autorização via **IAM + Kubernetes RBAC**

🔗 **Diagrama da arquitetura:**  
https://excalidraw.com/#json=b2OWH6L8Kd3kXwXDgkmtT,njdgFDQEauZBcF-V6kQRRQ

---

## 🧱 Componentes Provisionados

### VPC

A VPC foi provisionada utilizando o módulo oficial:

- `terraform-aws-modules/vpc/aws`

A adoção do módulo visa:
- Reutilização de código
- Padronização
- Uso de boas práticas consolidadas pela comunidade Terraform

Foi utilizado **NAT Gateway** para permitir que recursos em subnets privadas tenham acesso à internet de forma segura, sem exposição direta.

---

### Amazon EKS

O cluster EKS foi provisionado utilizando o módulo oficial:

- `terraform-aws-modules/eks/aws`

Principais decisões:
- Nós de trabalho (Managed Node Groups) hospedados exclusivamente em **subnets privadas**
- Control plane totalmente gerenciado pela AWS
- Comunicação segura entre control plane e nós
- **IRSA (IAM Roles for Service Accounts)** habilitado para acesso seguro a serviços AWS a partir dos pods

---

## 🔧 Pré-requisitos

Antes de iniciar, é necessário ter instalado e configurado:

- AWS CLI
- Terraform **>= 1.5**
- kubectl
- Credenciais AWS válidas

---

## 🔐 Credenciais AWS

O Terraform **não solicita credenciais de forma interativa**.

As credenciais são obtidas por meio da cadeia padrão de autenticação do provider AWS, podendo ser fornecidas via:

- Variáveis de ambiente (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
- Perfis configurados no arquivo `~/.aws/credentials`

Exemplo de configuração local:

```bash
aws configure
```

Antes de executar o Terraform, recomenda-se validar a conta ativa:

```bash
aws sts get-caller-identity
```

---

## 👤 IAM

Foi criado um **usuário IAM dedicado** para execução do Terraform, utilizando acesso programático por meio de **Access Keys**.

- Para fins de simplificação do desafio, foi atribuída a política `AdministratorAccess`
- Em um ambiente produtivo, esse usuário teria permissões restritas, seguindo o **princípio do menor privilégio**
- O uso do usuário **root** da conta AWS é evitado

As credenciais AWS **não são definidas no código Terraform**.  
Elas são fornecidas via configuração local da AWS CLI, respeitando a cadeia padrão de autenticação do provider.

---

## 🚀 Provisionamento da Infraestrutura

Todo o provisionamento da infraestrutura é realizado de forma **automatizada e declarativa** via Terraform.

Fluxo de execução:

```bash
terraform init
terraform plan
terraform apply
```

Durante o desenvolvimento, o comando `terraform plan` foi utilizado para validação das alterações.  
Em ambientes produtivos, recomenda-se o uso da opção `-out` para garantir que o plano aplicado seja exatamente o mesmo previamente aprovado.

Em um cenário produtivo, o estado do Terraform seria armazenado remotamente em um bucket **S3**, com bloqueio de estado via **DynamoDB**.  
Nesse contexto, as credenciais AWS seriam necessárias já durante a execução do comando `terraform init`.

---

## ☸️ Validação do Cluster

Após o provisionamento, o cluster foi validado por meio do `kubectl`, confirmando:

- Presença dos nós em estado **Ready**
- Funcionamento adequado dos componentes básicos do Kubernetes

Inicialmente, o endpoint da API do Kubernetes estava acessível apenas de forma **privada**.  
Para permitir testes locais via `kubectl`, o acesso público foi habilitado juntamente com o acesso privado.

📌 Em ambientes produtivos, recomenda-se o acesso ao endpoint privado por meio de **VPN** ou **bastion host**.

Para permitir acesso administrativo ao cluster, foi configurado o mapeamento de um usuário IAM ao cluster, garantindo:
- Autenticação via IAM
- Autorização via Kubernetes **RBAC**

---

## 🧨 Remoção da Infraestrutura

A remoção completa da infraestrutura pode ser realizada com o comando:

```bash
terraform destroy
```

Esse processo garante que todos os recursos gerenciados pelo Terraform sejam devidamente eliminados, evitando custos desnecessários na AWS.

---

## 📝 Considerações Finais

- As credenciais AWS não são armazenadas no repositório
- A infraestrutura segue boas práticas de segurança e isolamento de rede
- O uso de módulos oficiais reduz complexidade operacional e facilita manutenção
- A solução é totalmente reproduzível por meio de infraestrutura como código

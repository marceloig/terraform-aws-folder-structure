# Padrão de Estrutura de Diretórios para Projetos Terraform

Este repositório define um padrão de organização de diretórios e arquivos para projetos Terraform, testado e validado em múltiplos ambientes de produção. A estrutura proposta promove isolamento de estados, facilita a manutenção e reduz riscos de alterações indesejadas entre recursos.

## Estrutura de Diretórios

```
/
├── account/
│   └── region/
│       └── environment/
│           ├── workloads/
│           │   ├── foo/ 
│           │   │   ├── main.tf
│           │   │   ├── variables.tf
│           │   │   ├── outputs.tf
│           │   │   ├── provider.tf
│           │   │   └── backend.tf
│           │   └── bar/
│           │       ├── main.tf
│           │       ├── variables.tf
│           │       ├── outputs.tf
│           │       ├── provider.tf
│           │       └── backend.tf
│           └── shared_resources/
│               └── vpc/
│                   ├── main.tf
│                   ├── variables.tf
│                   ├── outputs.tf
│                   ├── provider.tf
│                   └── backend.tf
└── modules/
    └── frontend/

```

## Vantagens da Estrutura

### Isolamento de Estados
- **Estados independentes**: Cada workload e recurso compartilhado possui seu próprio state file
- **Redução de riscos**: Alterações em um componente não afetam outros recursos já implantados
- **Facilita rollbacks**: Possibilidade de reverter alterações de forma granular

### Prevenção de Interdependências
- **Execução independente**: Recursos podem ser aplicados sem dependências complexas entre módulos
- **Facilita debugging**: Problemas podem ser isolados e corrigidos por componente
- **Melhora performance**: Execuções paralelas quando apropriado

### Multi-Account e Multi-Region
- **Providers isolados**: Cada workload possui seu próprio arquivo `provider.tf`
- **Flexibilidade de deployment**: Recursos podem ser implantados em diferentes contas e regiões
- **Reduz erros de configuração**: Menor risco de implantar recursos na conta/região incorreta

### Organização e Manutenibilidade
- **Estrutura clara**: Hierarquia intuitiva que facilita navegação e compreensão
- **Padronização**: Consistência entre diferentes projetos e equipes
- **Escalabilidade**: Estrutura que cresce naturalmente com o projeto

## Detalhamento dos Diretórios

### Account (`account/`)
Representa a conta AWS do projeto. O nome do diretório deve corresponder ao:
- Nome da conta AWS, ou
- Alias da conta AWS

### Region (`region/`)
Representa a região AWS onde os recursos serão implantados.

**Exemplos**:
- `us-east-1/` (Virginia)
- `us-west-2/` (Oregon)
- `eu-west-1/` (Irlanda)

**Recursos Globais**: Para serviços AWS globais que podem ter conflitos de nomenclatura entre regiões, utilize o diretório `global/`:
- IAM
- Route53
- CloudFront
- WAF Global

### Environment (`environment/`)
Representa o ambiente de deployment do projeto.

**Exemplos**:
- `production/`
- `staging/`
- `development/`
- `sandbox/`

### Workloads (`workloads/`)
Contém as aplicações ou cargas de trabalho específicas. Cada subdiretório representa uma aplicação independente com sua própria infraestrutura.

**Exemplo de workloads**:
- `api-backend/`: Serviços ECS, RDS Aurora, SQS
- `frontend-app/`: CloudFront, S3, Route53
- `data-pipeline/`: Lambda, Kinesis, DynamoDB

**Comunicação entre workloads**: Use data sources do Terraform para referenciar recursos de outros workloads.

### Shared Resources (`shared_resources/`)
Recursos compartilhados entre múltiplas aplicações no mesmo ambiente.

**Exemplos de recursos compartilhados**:
- **Networking**: VPC, Subnets, NAT Gateways
- **Security**: IAM Roles e Policies compartilhadas
- **Monitoring**: CloudWatch, CloudTrail
- **DNS**: Route53 Hosted Zones

### Modules (`modules/`)
Armazena módulos Terraform desenvolvidos especificamente para o projeto

**Recomendação**: Use módulos de terceiros apenas quando necessário e sempre com versionamento fixo.

## Arquivos Padrão por Workload

Cada workload e shared resources deve conter os seguintes arquivos:

- **`main.tf`**: Recursos principais do Terraform
- **`variables.tf`**: Definição de variáveis de entrada
- **`outputs.tf`**: Valores de saída para outros módulos
- **`provider.tf`**: Configuração de providers
- **`backend.tf`**: Configuração do state backend

## Exemplo Prático

```
xpto/
├── us-east-1/
│   ├── production/
│   │   ├── workloads/
│   │   │   ├── web-app/
│   │   │   │   ├── main.tf          # ECS, ALB, Target Groups
│   │   │   │   ├── variables.tf
│   │   │   │   ├── outputs.tf
│   │   │   │   ├── provider.tf
│   │   │   │   └── backend.tf
│   │   │   └── api-backend/
│   │   │       ├── main.tf          # ECS, RDS, ElastiCache
│   │   │       ├── variables.tf
│   │   │       ├── outputs.tf
│   │   │       ├── provider.tf
│   │   │       └── backend.tf
│   │   └── shared_resources/
│   │       ├── vpc/
│   │       │   ├── main.tf          # VPC, Subnets, IGW
│   │       │   └── ...
│   │       └── security/
│   │           ├── main.tf          # Security Groups, NACLs
│   │           └── ...
│   └── global/
│       └── production/
│           └── shared_resources/
│               └── iam/
│                   ├── main.tf      # IAM Roles e Policies
│                   └── ...
```

## Referências

Esta estrutura foi adaptada das melhores práticas recomendadas pelo projeto [Terragrunt](https://github.com/gruntwork-io/terragrunt-infrastructure-live-example) e validada em ambientes de produção.

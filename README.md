# terraform-aws-modules

Biblioteca de módulos Terraform para AWS da **SolidaryTech**, versionada por tag e consumida pelo [`solidarytech-iac`](https://github.com/fiap-hackathon-dclt/solidarytech-iac).

Um módulo aqui encapsula **como** um recurso é criado: criptografia, rede, IAM e padrões de segurança que valem para qualquer ambiente. O **quê** e **onde** ficam no repositório de infraestrutura, que chama estes módulos passando valores.

## Origem

Esta biblioteca é a cópia da [biblioteca do ToggleMaster](https://github.com/fiap-tech-challenge-devops/terraform-aws-modules) na tag `v0.1.0`, onde os módulos foram construídos e validados ao longo das Fases 3 e 4. Chegaram aqui **sem alteração**; as adaptações para a SolidaryTech entram como tags novas.

## Módulos

| Módulo | Provisiona |
|---|---|
| [`vpc`](vpc/) | VPC com subnets públicas e privadas em várias zonas, Internet Gateway, NAT Gateways e rotas |
| [`eks`](eks/) | Cluster EKS: control plane, managed node groups, add-ons, OIDC para IRSA, KMS para secrets, logs e access entries |
| [`eks-karpenter`](eks-karpenter/) | A parte AWS do Karpenter: role dos nós, IRSA do controller, fila de interrupção e regras do EventBridge |
| [`eks-aws-lb-controller`](eks-aws-lb-controller/) | A IRSA do AWS Load Balancer Controller, com a policy oficial |
| [`rds`](rds/) | Instância RDS com subnet group, security group, KMS, senha no Secrets Manager e Enhanced Monitoring |
| [`dynamodb`](dynamodb/) | Tabela DynamoDB com GSI/LSI, TTL, Point-in-Time Recovery, Streams e KMS opcional |
| [`sqs`](sqs/) | Fila SQS Standard ou FIFO, com dead-letter queue e criptografia |
| [`elasticache`](elasticache/) | Redis ou Valkey em replication group, com criptografia em repouso e em trânsito |
| [`ecr`](ecr/) | Um ou mais repositórios de imagem com tag imutável, scan no push e lifecycle policy |
| [`iam-irsa`](iam-irsa/) | Role IAM assumível por service accounts do cluster |
| [`secrets-manager`](secrets-manager/) | Segredo no Secrets Manager, como string ou mapa JSON |
| [`ec2`](ec2/) | Instância EC2 com IMDSv2, EBS criptografado e security group dedicado |

Cada módulo traz o próprio `README.md`, a pasta `doc/` e um `example/` executável, onde `terraform validate` verifica o contrato.

Os módulos de cluster **não instalam charts** e só exigem o provider `aws`. A instalação dos controladores fica com quem consome, pelo GitOps.

## Tags obrigatórias

O enunciado exige `Project=SolidaryTech`, `Environment=Production` e `CostCenter=NGO-Core` em **todos** os recursos de nuvem. Todo módulo aceita a variável `tags`, e o consumidor ainda pode declarar `default_tags` no provider.

Isso cobre o que o Terraform cria. Não cobre o que é criado depois, dentro do cluster: instâncias EC2 lançadas pelo Karpenter, volumes EBS do driver de disco e load balancers do Load Balancer Controller. Essas tags são configuradas nos próprios controladores, no repositório GitOps.

## Como consumir

Sempre por tag, nunca por `main`. Um módulo referenciado por branch muda sem aviso no próximo `plan`.

```hcl
module "vpc" {
  source = "git::https://github.com/fiap-hackathon-dclt/terraform-aws-modules.git//vpc?ref=vX.Y.Z"
  # ...
}
```

## Versionamento

| Mudança | Tag |
|---|---|
| Quebra a interface de um módulo: variável removida ou renomeada, recurso recriado | maior |
| Acrescenta variável opcional ou módulo novo | menor |
| Correção sem efeito na interface | patch |

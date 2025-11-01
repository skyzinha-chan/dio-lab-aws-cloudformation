# Desafio DIO: Implementando a Primeira Stack com AWS CloudFormation

Este repositório documenta a implementação do laboratório "Implementando sua Primeira Stack com AWS CloudFormation", como parte do bootcamp da DIO. O objetivo é aplicar e documentar o conhecimento sobre Infraestrutura como Código (IaC) utilizando os templates fornecidos.

## Análise dos Templates do Laboratório

Os arquivos `.yaml` fornecidos representam uma progressão lógica na construção de infraestrutura na AWS.

### 1. `01-EC2.yaml`

* **Descrição:** Template fundamental para a criação de uma única instância EC2.
* **Recursos:** `AWS::EC2::Instance`.
* **Conceitos-Chave:**
    * `AvailabilityZone`: Define a AZ específica para a instância (ex: `us-east-1a`).
    * `ImageId`: Especifica a Amazon Machine Image (AMI) a ser usada.
    * `InstanceType`: Define o tipo de máquina (ex: `t2.micro`).
    * `Tags`: Utilizado para adicionar metadados à instância, facilitando a identificação (ex: `Name: "EC2"`).

### 2. `02-Apache.yaml`

* **Descrição:** Evolução do primeiro template, adicionando a configuração de software na instância.
* **Recursos:** `AWS::EC2::Instance`.
* **Novos Conceitos:**
    * `UserData`: Permite a execução de um script de shell na inicialização (bootstrap) da instância.
    * O script (codificado em Base64) instala o servidor web Apache (`yum install -y httpd.x86_64`), inicia o serviço e cria uma página `index.html` de teste.

### 3. `03-Firewall.yaml`

* **Descrição:** Adiciona o componente essencial de rede (firewall) para permitir tráfego externo para o servidor web.
* **Recursos:** `AWS::EC2::Instance` e `AWS::EC2::SecurityGroup`.
* **Novos Conceitos:**
    * **Criação de Múltiplos Recursos:** O template agora define dois recursos distintos.
    * `AWS::EC2::SecurityGroup`: Define um grupo de segurança (firewall).
    * `SecurityGroupIngress`: Propriedade que define as regras de entrada. Neste caso, libera a porta TCP 80 (HTTP) para todos os IPs (`0.0.0.0/0`).
    * `!Ref` (Função Intrínseca): Usada na propriedade `SecurityGroups` da instância EC2 para fazer referência lógica ao `GrupoSeguranca` criado no mesmo template.

### 4. `04-EC2_S3_UserGroup.yaml`

* **Descrição:** O template mais completo, demonstrando a orquestração de múltiplos serviços da AWS e o uso de recursos avançados do CloudFormation.
* **Recursos:** `AWS::S3::Bucket`, `AWS::IAM::Group`, `AWS::IAM::User`, `AWS::EC2::Instance`, `AWS::EC2::SecurityGroup`.
* **Novos Conceitos:**
    * `Parameters`: Torna o template reutilizável, permitindo que o usuário forneça valores na criação da stack (ex: `InstanceType`).
    * `Mappings`: Cria um mapa estático (ex: `UbuntuMap`) para selecionar valores com base em uma chave, como a Região da AWS (usado para encontrar o `ImageId` correto).
    * `!FindInMap` (Função Intrínseca): Utilizada para buscar o valor (`UbuntuAMI`) dentro do mapa (`UbuntuMap`).
    * `Outputs`: Fornece informações úteis após a criação da stack, como o `InstanceId` ou o nome do `S3BucketName`.
    * **Orquestração de Múltiplos Serviços:** O template provisiona S3, IAM e EC2 em uma única operação.

## Como Executar (Exemplo)

1.  Acesse o Console da AWS e navegue até o serviço **CloudFormation**.
2.  Clique em "Criar stack" e escolha "Com novos recursos (padrão)".
3.  Em "Preparar template", escolha "Fazer upload de um arquivo de modelo".
4.  Clique "Escolher arquivo" e selecione um dos `.yaml` da pasta `templates/`.
5.  Dê um nome para a stack (ex: `meu-webserver-03`).
6.  Avance nas etapas (para os arquivos 01, 02 e 03, os padrões podem ser mantidos). Para o arquivo 04, você pode selecionar o tipo de instância.
7.  Confirme e aguarde a stack ser criada (`CREATE_COMPLETE`).
8.  (Para os laboratórios 03 e 04) Você pode testar o acesso à instância (via IP público na porta 80 para o 03, ou via SSH na porta 22 para o 04).
9.  **Importante:** Após testar, lembre-se de **excluir a stack** para evitar cobranças.

## Insights e Documentação da Experiência

Para este desafio, foquei em analisar a estrutura e os conceitos demonstrados em cada um dos quatro templates:

1.  **Fundamentos (01-EC2):** A sintaxe básica do CloudFormation para declarar um recurso simples.
2.  **Bootstrapping (02-Apache):** O poder do `UserData` para automatizar a configuração de software, transformando uma instância genérica em um servidor web funcional.
3.  **Dependências e Rede (03-Firewall):** A importância de gerenciar recursos de rede (Security Groups) e como o CloudFormation resolve dependências lógicas usando `!Ref` (a instância só é associada ao SG depois que ele é criado).
4.  **Templates Avançados (04-EC2_S3_UserGroup):** A criação de templates robustos e reutilizáveis. O uso de `Parameters` desacopla os valores do template (como o tipo de instância), e os `Mappings` permitem que o mesmo template funcione em diferentes regiões sem modificação. A seção `Outputs` é fundamental para integrar a stack com outros processos.

A progressão dos arquivos demonstra claramente como o CloudFormation vai além de simplesmente "criar um servidor", permitindo a definição e o gerenciamento de arquiteturas complexas como código.

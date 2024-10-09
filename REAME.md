O que o script faz:
1- A instância EC2 é criada com a imagem Amazon Linux 2.
2- No script UserData, o Docker e o Docker Compose são instalados.
3- Um contêiner com Nginx é configurado e iniciado via Docker Compose.
4- O Security Group permite acesso SSH e HTTP para a instância.
5- No final, você pode acessar o servidor Nginx na instância usando o endereço IP público da EC2.

Para executar um template **CloudFormation** usando a **AWS CLI**, siga estes passos:

### 1. Salve o template
Primeiro, salve o seu template YAML (o que você criou ou o exemplo que forneci) em um arquivo local, por exemplo: `ec2-docker-cloudformation.yaml`.

### 2. Faça o upload do template para um bucket S3 (opcional)
Se você estiver executando o comando de uma máquina local, pode ser necessário fazer o upload do template para o **Amazon S3** para que a AWS CloudFormation possa acessá-lo. Caso você não tenha um bucket já criado, crie um usando o comando abaixo:

```bash
aws s3 mb s3://meu-bucket-cloudformation
```

Agora, faça o upload do template para o bucket:

```bash
aws s3 cp ec2-docker-cloudformation.yaml s3://meu-bucket-cloudformation/
```

### 3. Executar o template usando AWS CLI

Agora, você pode usar o comando `aws cloudformation create-stack` para criar a pilha (stack) do CloudFormation. O comando seria algo assim:

#### Com template local (diretamente do seu arquivo local):
```bash
aws cloudformation create-stack \
  --stack-name MinhaPilhaEC2Docker \
  --template-body file://ec2-docker-cloudformation.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.micro \
               ParameterKey=KeyName,ParameterValue=meuParDeChaves \
  --capabilities CAPABILITY_NAMED_IAM
```

#### Com template no S3 (se você fez o upload):
```bash
aws cloudformation create-stack \
  --stack-name MinhaPilhaEC2Docker \
  --template-url https://meu-bucket-cloudformation.s3.amazonaws.com/ec2-docker-cloudformation.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.micro \
               ParameterKey=KeyName,ParameterValue=meuParDeChaves \
  --capabilities CAPABILITY_NAMED_IAM
```

### Explicação:
- **`--stack-name`**: O nome que você deseja dar para a sua pilha.
- **`--template-body`**: Aponta para o arquivo do template local usando o prefixo `file://`.
- **`--template-url`**: Caso tenha subido para o S3, aqui vai a URL completa do arquivo no bucket S3.
- **`--parameters`**: Permite passar parâmetros definidos no template, como o tipo de instância EC2 (`InstanceType`) e o par de chaves (`KeyName`).
- **`--capabilities CAPABILITY_NAMED_IAM`**: Este parâmetro é necessário se seu template criar ou modificar recursos IAM, como perfis de instância.

### 4. Verificar o status da pilha
Você pode verificar o progresso da criação da pilha com o seguinte comando:

```bash
aws cloudformation describe-stacks --stack-name MinhaPilhaEC2Docker
```

Esse comando mostrará informações sobre o status da pilha, incluindo os recursos sendo criados. Você também pode usar o **AWS Management Console** para monitorar a criação da pilha.

### 5. Acessar a instância EC2
Após a criação da pilha, você pode acessar a instância EC2 via SSH:

```bash
ssh -i "meuParDeChaves.pem" ec2-user@<PublicIP>
```

Lembre-se de substituir `<PublicIP>` pelo IP público retornado na saída da pilha (você pode consultar com o comando `describe-stacks` ou na seção de **Outputs** no console da AWS).

### 6. Remover a pilha
Quando você não precisar mais dos recursos, pode excluí-los removendo a pilha com o comando:

```bash
aws cloudformation delete-stack --stack-name MinhaPilhaEC2Docker
```

Isso removerá todos os recursos que foram criados pelo CloudFormation.

Com esses comandos, você pode criar e gerenciar pilhas CloudFormation diretamente via AWS CLI!
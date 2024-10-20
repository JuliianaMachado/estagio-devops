# Estágio Devops - Desafio

## Análise Técnica do Código Terraform

O código tem o objetivo de criar diversos recursos utilizando o provider AWS, conforme descrito abaixo:

### 1. Variáveis utilizadas

1. Variável `projeto` com valor default `VExpenses`
2. Variável `candidato` com valor default `JulianaMachado`

### 2. Recursos de Rede

1. `aws_vpc.main_vpc`: Criação de uma `VPC` no padrão (`$projeto-$candidato-vpc`) com o range de ip `10.0.0.0/16` (65534 hosts úteis)
2. `aws_subnet.main_subnet`: Criação de uma `Subnet` no padrão (`$projeto-$candidato-subnet`). Essa subnet será criada dentro da `VPC` descrita no item 1, na região `us-east-1`e zona `us-east-1a`. Além disso, utiliza o range de ip `10.0.1.0/24` (254 hosts úteis)
3. `aws_internet_gateway.main_igw`: Criação de um `InternetGateway` no padrão (`$projeto-$candidato-igw`) para acesso à internet dos recursos criados na `vpc` do item 1.
4. `aws_route_table.main_route_table`: Criação de um `RouteTable` no padrão (`$projeto-$candidato-route_table`) vinculada à `vpc` do item 1. Definindo acesso à internet para todos os hosts criados na `vpc`.
5. `aws_route_table_association.main_association`: Criação de um `RouteTableAssociation` no padrão (`$projeto-$candidato-route_table_association`), vinculando o `RouteTable` do item 4 à `Subnet` do item 2.

### 3. Chaves

1. `tls_private_key.ec2_key`: Criação de uma chave privada utilizando o algoritmo `RSA` com o tamanho de `2048` bits.
2. `aws_key_pair.ec2_key_pair`: Criação de um `KeyPair` (aws_key_pair.ec2_key_pair) no padrão (`$projeto-$candidato-key`), utilizando a chave pública da chave privada criada no item 1.

### 4. Recursos de computação

1. `aws_ami.debian12`: Busca da mais recente `AMI` utilizando os filtros de `name=debian-12-amd64-*` e `virtualization-type=hvm`.
2. `aws_security_group.main_sg`: Criação de um `SecurityGroup` no padrão (`$projeto-$candidato-sg`) vinculada a `vpc` do item 2.1, com as seguintes regras de entrada e saída de tráfego de rede.
    2.1 Regra de entrada: Acesso via SSH (porta 22 e protocolo TCP) de qualquer origem IPv4 (`0.0.0.0/0`) e IPv6 (`::/0`)
    2.2 Regra de entrada: Acesso via HTTP (porta 80 e protocolo TCP) de qualquer origem IPv4 (`0.0.0.0/0`) e IPv6 (`::/0`)
    2.3 Regra de saída: Acesso de saída para qualquer destino IPv4 (`0.0.0.0/0`) e IPv6 (`::/0`)
3. `aws_instance.debian_ec2`: Criação de uma instância `EC2` no padrão (`$projeto-$candidato-ec2`) utilizando a AMI do item 4.1, tipo `t2.micro`, na subnet do item 2.2, utilizando a chave do item 3.2, vinculando o `SecurityGroup` do item 4.2, habilitando `IP Público` e com um volume root de 20gb que será deletado quando a instancia for encerrada.

### 5. Recursos de Saída
1. `private_key`: Retorno da chave privada criada no item 3.1
2. `ec2_public_ip`: Retorno do `IP Público` da instância criada no item 4.3

### Modificações e/ou Melhorias

1. Separação do código em arquivos separados melhor visualização
2. Separação de regras de `ingress` e `egress` do `SecurityGroup` para facilitar futuras manutenções/evoluções.
3. Inclusão do código de instalação do `nginx` no `userData` da instancia
4. Remoção do atributo `tags` no recurso `aws_route_table_association.main_association`
5. Remoção de acentos no atributo `description` do código de criação do recurso `aws_security_group.main_sg`
6. Alteração do atributo `security_groups` para `vpc_security_group_ids` na criação do recurso `aws_instance.debian_ec2`
7. Alteração do valor `owners` do recurso `aws_ami.debian12` para `amazon`
8. Adição da regra de ingress `aws_security_group_rule.sg_ingress_2` para liberar a porta `80` para acesso ao nginx
9. Talvez restringir o acesso a porta SSH

## Instruções de utilização

1. Conta aws configurada na máquina
2. Aplicar a seguinte sequencia de comandos
```
terraform init
terraform plan
terraform apply
```
3. Remoção dos recursos, utilizar o comando `terraform destroy`
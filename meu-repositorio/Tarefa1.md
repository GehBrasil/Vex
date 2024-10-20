
md_descr = """

# Descrição Técnica do arquivo main.tf


O objetivo deste código é definir uma estrutura básica na AWS com Terraform, incluindo VPC, Subnet, Grupo de Segurança, Key Pair e instância EC2.

----------

# Provider

- provider "aws"
   - Configura e usa a AWS como provedor
   - "us-east-1" mostra a região de criação

# Variable

- variable "projeto" / variable "candidato"
   - Campos variáveis para nomear os recursos, com saída dos valores padrão "VExpenses" e "SeuNome"

----------

# Keys

- "private_key"
   - Gera uma chave privada com o algoritmo RSA com 2048 bits, que será usada para acessar EC2 via SSH

- "key_pair"
   - Registra a chave pública na AWS com um par de chaves "${var.projeto}-${var.candidato}-key"

----------

# VPC

- "aws_vpc" "main_vpc"
   - Cria uma VPC "10.0.0.0/16", que define endereços IP
   - "enable_dns_support   = true" / "enable_dns_hostnames = true" habilitam DNS para VPC

- "subnet"  
   - Cria uma sub-rede na VPC "10.0.1.0/24", na região "us-east-1a"

- "internet_gateway"   
   - Cria um Gateway, que libera que a VPC se comunique com a internet pública.

- "route_table" / "route_table_association"
   - Cria uma tabela de rota direcionada com o tráfego "0.0.0.0/0" para a internet Gateway.
   - Associa tabela de rota à sub-rede, fazendo ela pública, permitindo que a EC2 receba um IP público para acesso via SSH.

----------

# Security

- "security_group" 
   - Cria um grupo de segurança para controlar entrada e saída da EC2. Nomeada com "${var.projeto}-${var.candidato}-sg"  
   
- Regras de Entrada (ingress)
   - Libera o tráfego SSH de qualquer lugar com IP público

- Regras de Saída (Egress)
   - Libera o tráfego de saída, podendo acessar de qualquer destino externo na internet.

----------

# Debian

- Usar "data aws_ami" para buscar imagem do Debian 12 na AWS  
- Filtro para buscar imagens com nome que corresponde a "debian-12-amd64-*" e tipo HVM  
- "owners = ["679593333241"]"" para AMIs oficiais da Debian

----------

# Instance EC2 

   - Cria EC2 do tipo "t2.micro" usando a AMI do Debian
   - É criada na sub-rede pública configurada e recebe um IP público para liberar acesso remoto
   - Chave SSH é associada para liberar login via SSH
   - Depois é associada ao grupo de segurança para que o tráfego SSH seja liberado

# Root Block Device  

   - Configura o disco com tamanho de 20 GB de espaço
   - Tipo de volume "gp2"
   - Excluir o volume quando for encerrado


# User Data  

   - Update na inicialização (atualiza os pacotes)
   - Upgrade de todos os pacotes instalados

----------

# Outputs

- "private_key" 
   - Mostra chave privada que é usada para acessar a EC2. É marcada como sensível, para não exibir nos logs ou no console.

- Public_ip
   - Mostra o endereço IP público da EC2

----------

# Interações entre Recursos

1. Keys: A key privada é criada localmente e o par de keys é registrado na AWS
2. VPC e Rede: A VPC é criada, com uma sub-rede pública ligada a uma tabela de rota e conectada ao Internet Gateway
3. Security: O grupo segurança libera SSH e todo o tráfego de saída da instância
4. Instance EC2: É criada na sub-rede com acesso à internet e associada ao par de keys e ao grupo de segurança
5. Outputs: O Terraform mostra a key privada e o IP público da instância

----------

# Conclusão

Este código é uma solução funcional para criar uma infraestrutura básica na AWS, com foco em EC2 Debian com acesso pela internet. Ela mostra conceitos como gerenciamento de VPC, grupos de segurança, e controle de acesso via SSH. 
Foram feitas configurações para ser mais flexivel, porém, seria bom algumas melhorias em segurança, como restringir o acesso SSH a IPs específicos.

-----------

"""

# Salvar em md

file_path = "/mnt/data/descricao_terraform.md"
with open(file_path, "w") as f:
    f.write(md_descr)

file_path



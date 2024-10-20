
md_readme = """


### TAREFA 1 ###


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


--------------------


### CÓDIGO MAIN.TF COM MELHORIAS ###


provider "aws" {
  region = "us-east-1"
}

variable "projeto" {
  description = "Nome do projeto"
  type        = string
  default     = "VExpenses"
}

variable "candidato" {
  description = "Nome do candidato"
  type        = string
  default     = "SeuNome"
}

resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 4096  # Aumentando o tamanho da chave para maior segurança
}

resource "aws_key_pair" "ec2_key_pair" {
  key_name   = "${var.projeto}-${var.candidato}-key"
  public_key = tls_private_key.ec2_key.public_key_openssh
}

resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.projeto}-${var.candidato}-vpc"
  }
}

resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "${var.projeto}-${var.candidato}-subnet"
  }
}

resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id

  tags = {
    Name = "${var.projeto}-${var.candidato}-igw"
  }
}

resource "aws_route_table" "main_route_table" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main_igw.id
  }

  tags = {
    Name = "${var.projeto}-${var.candidato}-route_table"
  }
}

resource "aws_route_table_association" "main_association" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_route_table.id
}

resource "aws_security_group" "main_sg" {
  name        = "${var.projeto}-${var.candidato}-sg"
  description = "Permitir SSH restrito e HTTP, com tráfego de saída completo"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    description = "Allow HTTP traffic"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow SSH only from specific IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    # Alterar "meu_ip" para o IP do seu dispositivo atual.
    cidr_blocks = ["<meu_ip>/32"]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.projeto}-${var.candidato}-sg"
  }
}

data "aws_ami" "debian12" {
  most_recent = true

  filter {
    name   = "name"
    values = ["debian-12-amd64-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["679593333241"]
}

resource "aws_instance" "debian_ec2" {
  ami               = data.aws_ami.debian12.id
  instance_type     = "t2.micro"
  subnet_id         = aws_subnet.main_subnet.id
  key_name          = aws_key_pair.ec2_key_pair.key_name
  vpc_security_group_ids = [aws_security_group.main_sg.id]
  associate_public_ip_address = true

  root_block_device {
    volume_size           = 20
    volume_type           = "gp3"  # Melhor desempenho e custo-benefício
    delete_on_termination = true
  }

  user_data = <<-EOF
    #!/bin/bash
    apt-get update -y
    apt-get upgrade -y
    apt-get install -y nginx
    systemctl start nginx
    systemctl enable nginx
    ufw allow 'Nginx HTTP'
  EOF

  tags = {
    Name = "${var.projeto}-${var.candidato}-ec2"
  }
}

output "private_key" {
  description = "Chave privada para acessar a instância EC2"
  value       = tls_private_key.ec2_key.private_key_pem
  sensitive   = true
}

output "ec2_public_ip" {
  description = "Endereço IP público da instância EC2"
  value       = aws_instance.debian_ec2.public_ip
}


--------------------


### TAREFA 2 ###

# Descrição Técnica das Alterações e Resultados Esperados

1. Aplicação de melhorias de segurança

- Chave RSA de 4096 bits: Substituí a chave de 2048 bits por uma de 4096 bits, aumentando a segurança do par de chaves
- Restrição do SSH a um IP específico: Modificada a regra de entrada para permitir acesso SSH apenas do IP do usuário, evitando ataques por acesso irrestrito
- Uso do volume gp3: Alterado o tipo de volume para gp3, com melhor desempenho e menor custo do que o gp2

2. Automação da Instalação do Nginx

- O script "user_data" foi modificado para:

  - Instalar/iniciar o Nginx automaticamente com a EC2
  - Habilitar o serviço Nginx para iniciar junto com o sistema
  - Abrir a porta 80 no firewall

- Resultado esperado: Quando a instância for criada, o Nginx estará ativo e atuando na porta 80

3. Outras Melhorias

- Ajuste na configuração do grupo de segurança:

  - Agora, somente o tráfego HTTP (porta 80) é permitido de qualquer lugar
  - Todo o tráfego de saída é permitido, garantindo que a instância possa se comunicar com a internet
  - Uso de "vpc_security_group_ids" ao invés de "security_groups": Esta mudança garante maior compatibilidade em casos de VPCs e sub-redes personalizadas

----------

# Conclusão

Com essas alterações, o ambiente se torna mais seguro e funcional:

- O acesso SSH é restrito e a segurança da chave foi melhorada
- O Nginx é instalado e iniciado automaticamente, facilitando o processo
- Melhorias no volume de armazenamento e uso adequado dos recursos da AWS, tendo melhor custo-benefício e desempenho


--------------------


### PASSO A PASSO PARA INICIALIZAR E APLICAR A CONFIGURAÇÃO ###

# Pré-requisitos

1. Conta na AWS: Você precisa de uma conta ativa na AWS para provisionar recursos.

2. Instalação do Terraform:

Você deve ter o Terraform instalado em sua máquina. Você pode baixar a versão mais recente em terraform.io e seguir as instruções de instalação específicas para seu sistema operacional.

3. Configuração das Credenciais da AWS:

Você precisa configurar as credenciais da AWS. As credenciais podem ser definidas de várias maneiras:

- Arquivo de credenciais: Crie um arquivo em ~/.aws/credentials com o seguinte conteúdo:

            [default]
            aws_access_key_id = SUA_CHAVE_DE_ACESSO
            aws_secret_access_key = SUA_CHAVE_SECRETA

- Variáveis de ambiente:

            export AWS_ACCESS_KEY_ID=SUA_CHAVE_DE_ACESSO
            export AWS_SECRET_ACCESS_KEY=SUA_CHAVE_SECRETA


4. Permissões Adequadas: O usuário da AWS que você está utilizando deve ter permissões para criar os recursos definidos no código Terraform (VPC, EC2, Security Groups, etc.).


--------------------


# Passos para Inicializar e Aplicar a Configuração Terraform


1. Criar e Preparar o Diretório do Projeto

Crie um diretório em seu sistema onde você colocará os arquivos de configuração do Terraform. Navegue até esse diretório:

            mkdir meu_projeto_terraform
            cd meu_projeto_terraform

----------

2. Criar um Arquivo de Configuração:

Arraste/cole o arquivo no o diretório

----------

3. Inicializar o Terraform:

Execute o comando terraform init para inicializar o diretório do Terraform. Isso irá baixar os plugins necessários para o provedor AWS e preparar o ambiente.

            terraform init

----------      

4. Verificar o Plano:

Antes de aplicar as mudanças, é uma boa prática verificar o que será criado. Execute o seguinte comando para visualizar o plano de execução:

            terraform plan

* Esse comando mostrará quais recursos o Terraform irá criar, modificar ou destruir. Revise o plano e certifique-se de que tudo está correto.            

----------

5. Aplicar a Configuração:

Após verificar o plano, você pode aplicar as mudanças para criar os recursos na AWS. Execute:

      terraform apply

* O Terraform pedirá confirmação para continuar. Digite yes e pressione Enter.

----------

6. Aguardar a Criação dos Recursos:

O Terraform começará a criar os recursos conforme definido no código. Aguarde até que todos os recursos sejam criados. Você verá mensagens de progresso e, ao final, receberá as saídas definidas no código (por exemplo, o endereço IP público da instância EC2).

----------

7. Verificar os Recursos na AWS:

Após a aplicação bem-sucedida, você pode acessar o Console da AWS para verificar se os recursos foram criados corretamente.

----------


# Verificando a Instalação do Nginx

Acesse o IP público da instância no navegador:

      http://<IP-PUBLICO>

Você deverá ver a página padrão do Nginx.


----------


# Encerrando o Ambiente

Se você não precisar mais dos recursos que criou, é importante destruir esses recursos para evitar custos adicionais. Você pode fazer isso com o seguinte comando:

      terraform destroy

* Assim como no comando apply, o Terraform pedirá confirmação. Digite yes para prosseguir com a destruição dos recursos.


----------      


# Conclusão

Estas instruções garantem que você consiga inicializar, aplicar e verificar a configuração Terraform com sucesso.


----------  """


# Salvando o conteúdo em um arquivo md

caminho_arquivo = "/mnt/data/tarefas.md"
with open(caminho_arquivo, "w") as arquivo:
    arquivo.write(md_readme)

caminho_arquivo


### PASSO A PASSO PARA INICIALIZAR E APLICAR A CONFIGURAÇÃO ###


passo_a_passo_md = """


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

Assim como no comando apply, o Terraform pedirá confirmação. Digite yes para prosseguir com a destruição dos recursos.


----------      


# Conclusão

Estas instruções garantem que você consiga inicializar, aplicar e verificar a configuração Terraform com sucesso.


----------


"""

# Salvando o conteúdo em um arquivo md

caminho_arquivo2 = "/mnt/data/config_aplic.md"
with open(caminho_arquivo2, "w") as arquivo2:
    arquivo2.write(passo_a_passo_md)

caminho_arquivo2

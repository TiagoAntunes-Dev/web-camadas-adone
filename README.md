# ADO 1 - Hospedagem de Banco de Dados: Suki Doces

**Disciplina:** Aplicações Web em Camadas
**Projeto:** E-commerce Suki Doces

---

## 1. O meu banco de dados local
Durante grande parte do desenvolvimento da API do projeto Suki Doces, o meu banco de dados MySQL estava rodando exclusivamente na minha máquina. A configuração original no meu arquivo `.env` apontava diretamente para a porta local, como pode ser visto abaixo:

<img width="1906" height="785" alt="Configuração Local" src="https://github.com/user-attachments/assets/93c9ca12-0e66-4ddb-8b9f-8e14ec2da74d" />


Abaixo está a estrutura principal do meu banco de dados através do arquivo `schema.prisma`, que contém a modelagem de tabelas vitais para o e-commerce, como `produtos`, `pedidos`, `usuario`, `carrinho_itens` e `categorias`:

> <img width="1920" height="826" alt="Schema Prisma" src="https://github.com/user-attachments/assets/b4c99cea-ab51-44d6-8b2c-5bc4ae6920a5" />


E aqui está a comprovação do banco de dados populado e rodando ativamente no meu ambiente local através da interface do banco:

> [COLE AQUI A FOTO 3: O print do seu MySQL Workbench rodando localmente]

---

## 2. Opções de hospedagem gratuita pesquisadas
Para realizar a migração do banco de dados do Suki Doces para a nuvem, pesquisei e avaliei três serviços gratuitos de hospedagem:

1. **Aiven:** Oferece um plano gratuito muito robusto para MySQL, disponibilizando 5GB de armazenamento e 1GB de RAM, sem tempo de "sleep" (o banco não desliga por inatividade). É totalmente compatível com o Prisma. Foi a opção que **escolhi** por atender perfeitamente às demandas do meu e-commerce e pela facilidade de obter a connection string.
2. **Clever Cloud:** Também oferece bancos MySQL gratuitos e compatíveis com Prisma, mas o limite do plano grátis é extremamente restrito (apenas 10MB de dados e 5 conexões simultâneas). **Descartei** essa opção porque 10MB se esgotariam muito rápido ao começar a cadastrar imagens, produtos e registrar pedidos dos clientes.
3. **Neon:** É um serviço excelente, muito rápido e moderno, com bons limites no plano gratuito. No entanto, eu o **descartei** porque ele suporta exclusivamente PostgreSQL. Como o backend do Suki Doces já estava 100% tipado e estruturado em MySQL, fazer a migração de dialeto SQL não faria sentido nesta etapa.

---

## 3. A hospedagem que escolhi: Passo a Passo
A plataforma escolhida foi o **Aiven**. Abaixo detalho o processo real de como migrei meu projeto:

1. Acessei a plataforma e criei um serviço "MySQL" no plano Free.
2. Assim que o servidor iniciou, acessei a aba *Overview* do serviço e localizei a "Service URI", que é a connection string gerada pelo Aiven.
3. Fui até o arquivo `.env` do backend no VS Code, apaguei a URL do localhost e colei essa nova URL fornecida pela nuvem.
4. Pelo terminal do VS Code, rodei o comando `npx prisma db push`. O Prisma leu meu `schema.prisma` e construiu a estrutura de tabelas diretamente no servidor remoto do Aiven.

Abaixo, a captura de tela que comprova as tabelas do meu projeto devidamente criadas no servidor da nuvem:

> [COLE AQUI A FOTO 4: O print das tabelas rodando no Aiven ou no Workbench conectado ao Aiven]

Abaixo está o teste de integração: uma requisição via Thunder Client batendo na minha API e salvando/buscando os dados com sucesso diretamente no banco remoto!

> [COLE AQUI A FOTO 5: O print do Thunder Client com a requisição funcionando no banco remoto]

---

## 4. O que muda do local para o remoto

**A Connection String:**
A connection string não é apenas um link, mas sim a "chave de acesso" completa do banco. Ela especifica o protocolo (ex: `mysql://`), o usuário administrador (ex: `avnadmin`), a senha do banco, o "Host" (que agora é o endereço do servidor remoto na nuvem em vez do nosso "localhost" local), a porta de conexão (ex: 12089) e o nome do banco de dados alvo. 

**A segurança do .env e o GitHub:**
O arquivo `.env` é onde armazenamos dados ultrassecretos, como a connection string. Quando trabalhamos localmente, o risco é menor, mas ao hospedar o banco na nuvem, qualquer pessoa que tiver acesso ao `.env` terá controle total sobre o banco de dados. É por isso que o `.env` é inserido no `.gitignore` e **nunca** é enviado para o GitHub; caso contrário, os dados dos clientes da loja poderiam ser vazados ou deletados facilmente. Na nuvem (como no Render), precisamos preencher essas variáveis de ambiente manualmente no painel de controle.

**O que acontece com os dados locais:**
Ao realizar a migração e rodar o comando do Prisma (`db push` ou `migrate deploy`), os dados que eu havia cadastrado no meu PC (os doces de teste e meus usuários) **não foram transferidos para a nuvem**. O banco remoto começa completamente vazio. Isso ocorre porque o Prisma espelha apenas a arquitetura (as tabelas, colunas e relacionamentos), enquanto os dados em si continuam isolados fisicamente no disco rígido da minha máquina.

**Erros Encontrados e Solução:**
Durante o processo de desenvolvimento e deploy para o banco remoto, enfrentei um erro de conflito de *schema* muito interessante. No meu ambiente local, eu havia refatorado a arquitetura do banco para unificar as tabelas antigas de `cliente` e `usuario` em uma tabela só, visando simplificar as regras de negócio. 

O problema ocorreu ao tentar sincronizar essa nova estrutura local com o banco remoto usando o `prisma db push`. O Prisma detectou que a estrutura estava diferente e que a tabela antiga da nuvem precisaria ser apagada, e bloqueou a ação disparando um aviso de *Data Loss* (Risco de Perda de Dados), para me proteger. Como resolvi: precisei confirmar a ação forçando o Prisma a sobrescrever o banco (aceitando a recriação da tabela unificada) e, em seguida, reiniciei a API no provedor de hospedagem para que o backend passasse a utilizar o novo schema unificado sem falhas.

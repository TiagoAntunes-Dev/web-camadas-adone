ADO 1 · Hospedagem de Banco de Dados

Disciplina: Aplicações Web em Camadas
Projeto: Product CRUD API — Node.js + Express + MongoDB
Aluno: Tiago Antunes Paz de Oliveira


1. Configuração Local do Banco de Dados
Como estava o .env localmente
Durante o desenvolvimento, o banco de dados era acessado localmente através do MongoDB Atlas, mas antes disso eu testei com uma instância local do MongoDB. A variável de ambiente no arquivo .env ficava assim na fase de desenvolvimento local:
env# Versão local (MongoDB rodando na própria máquina)
MONGO_URI=mongodb://localhost:27017/crud_db
O arquivo .env nunca é enviado ao GitHub, porque ele contém credenciais sensíveis — como usuário, senha e o endereço do banco. Por isso ele está listado no .gitignore.
Schema / Model do projeto
Meu projeto não usa Prisma — ele usa Mongoose, que é a biblioteca de modelagem de objetos (ODM) para MongoDB. O equivalente ao schema.prisma no meu projeto é o arquivo product.model.js:
javascriptconst mongoose = require('mongoose');

const ProductSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'O nome do produto é obrigatório.'],
        trim: true
    },
    quantity: {
        type: Number,
        required: true,
        default: 0,
        min: [0, 'A quantidade em stock não pode ser negativa.']
    },
    price: {
        type: Number,
        required: true,
        default: 0,
        min: [0, 'O preço do produto não pode ser negativo.']
    },
    image: {
        type: String,
        required: false,
        trim: true
    }
},
{
    timestamps: true
});

module.exports = mongoose.model('Product', ProductSchema);

📸 [INSIRA AQUI: print do seu VSCode mostrando o arquivo product.model.js aberto]

Banco rodando localmente

📸 [INSIRA AQUI: print do MongoDB Compass (ou terminal) mostrando o banco crud_db rodando localmente, com a coleção products e alguns documentos]


2. Opções de Hospedagem Gratuita Pesquisadas
Como meu projeto usa MongoDB (não MySQL/PostgreSQL), pesquisei opções que suportam banco NoSQL na nuvem. Apresento três serviços que avaliei:

🔹 Opção 1: MongoDB Atlas
O que oferece no plano gratuito:
O MongoDB Atlas oferece um cluster gratuito chamado M0 (Shared Free Tier) com 512 MB de armazenamento, conexões compartilhadas e sem expiração — o cluster não "dorme" como em outros serviços. A limitação principal é o armazenamento e o desempenho compartilhado com outros usuários do plano gratuito.
Compatível com Mongoose (equivalente ao Prisma para MongoDB)?
Sim. O Mongoose se conecta ao Atlas pela connection string padrão mongodb+srv://. O processo é simples: cria-se o cluster, o Atlas gera a URI de conexão, e basta colá-la no .env.
Por que escolhi:
Escolhi o MongoDB Atlas porque ele é o serviço oficial do próprio MongoDB. A integração com Mongoose é nativa, a documentação é excelente e o cluster M0 gratuito não tem data de expiração. Além disso, o Atlas já é amplamente utilizado em tutoriais e cursos, o que facilita encontrar suporte.

🔹 Opção 2: Railway
O que oferece no plano gratuito:
O Railway oferece um plano gratuito com $5 de crédito por mês. Ele suporta vários tipos de banco, incluindo MongoDB, PostgreSQL e MySQL. O limite de crédito mensal significa que o serviço pode ser desligado quando o crédito acabar, o que pode ser um problema para projetos de longo prazo.
Compatível com Mongoose?
Sim. O Railway gera uma connection string que pode ser usada diretamente com Mongoose, da mesma forma que o Atlas.
Por que descartei:
Descartei porque o modelo de créditos mensais é imprevisível para projetos pessoais. Preferi uma opção com plano gratuito permanente e sem a preocupação de que o banco pare de funcionar no meio do mês.

🔹 Opção 3: Supabase
O que oferece no plano gratuito:
O Supabase é focado em PostgreSQL (banco relacional), não em MongoDB. Oferece 500 MB de armazenamento, 2 projetos gratuitos e o projeto "hiberna" após 7 dias de inatividade no plano gratuito.
Compatível com Mongoose?
Não é compatível com Mongoose diretamente, pois o Supabase usa PostgreSQL. Para usá-lo eu teria que migrar toda a estrutura do projeto para um banco relacional e usar o Prisma como ORM — o que representaria uma refatoração grande do projeto.
Por que descartei:
Descartei porque meu projeto é construído sobre MongoDB e Mongoose. Mudar para PostgreSQL exigiria reescrever o model, as validações e a lógica de consulta. Mantive o MongoDB Atlas, que é a solução natural para o stack que já estava usando.

3. A Hospedagem Escolhida: MongoDB Atlas — Passo a Passo
3.1 Criação da conta e do banco

Acessei cloud.mongodb.com e criei uma conta gratuita.
Criei uma organização e um projeto chamado "Project Crud".
Escolhi criar um cluster gratuito (M0 Shared), selecionando a região AWS / Sao Paulo (sa-east-1) para menor latência.
O Atlas criou o cluster Cluster0 automaticamente.


📸 [INSIRA AQUI: print da tela de Clusters do MongoDB Atlas mostrando o Cluster0 ativo (com o ponto verde)]

3.2 Configuração de acesso
Após criar o cluster, configurei dois pontos obrigatórios para conseguir conectar:

Database User: Criei um usuário com nome e senha específicos para a aplicação (não usei minha conta pessoal do Atlas).
Network Access (IP Whitelist): Adicionei 0.0.0.0/0 para permitir conexão de qualquer IP durante o desenvolvimento. Em produção isso deveria ser restrito.


📸 [INSIRA AQUI: print da tela de Network Access mostrando o IP liberado]

3.3 Onde encontrar a connection string
Dentro do painel do Atlas:

Cliquei em "Connect" no cluster.
Selecionei "Drivers" e escolhi Node.js.
O Atlas gerou a connection string no formato:

mongodb+srv://<usuario>:<senha>@cluster0.xxxxxxx.mongodb.net/<banco>
3.4 Atualização do .env
Substituí a URI local pela URI do Atlas no arquivo .env:
env# Antes (local)
MONGO_URI=mongodb://localhost:27017/crud_db

# Depois (remoto - Atlas)
MONGO_URI=mongodb+srv://tiagoantunes1974_db_user:<senha>@cluster0.7v3xmzm.mongodb.net/crud_db

⚠️ A senha nunca aparece diretamente no README — ela fica apenas no .env local, que está no .gitignore.

3.5 Criação das coleções no banco remoto
Como meu projeto usa Mongoose (e não Prisma), não existe um comando prisma migrate deploy. As coleções são criadas automaticamente pelo Mongoose quando o primeiro documento é inserido — esse é o comportamento padrão do MongoDB.
Para garantir que o banco remoto estava funcionando, subi o servidor e fiz uma requisição POST criando um produto. O Atlas criou automaticamente o banco crud_db e a coleção products.

📸 [INSIRA AQUI: print do MongoDB Atlas > Browse Collections mostrando o banco crud_db e a coleção products com documentos]

3.6 Requisição funcionando com o banco remoto
Com o servidor apontando para o Atlas, testei as rotas pelo Insomnia/Postman:

📸 [INSIRA AQUI: print do Insomnia/Postman mostrando uma requisição GET /api/products retornando os produtos do banco remoto, com status 200]


📸 [INSIRA AQUI: print do terminal mostrando o log ✅ Connected to Database! após conectar ao Atlas]


4. O que Muda do Local para o Remoto
O que é uma connection string e o que cada parte significa
Uma connection string é o endereço completo que o código usa para encontrar, autenticar e se conectar ao banco de dados. É como o "endereço postal" do banco. No caso do MongoDB Atlas, ela tem este formato:
mongodb+srv://usuario:senha@cluster0.7v3xmzm.mongodb.net/crud_db
Cada parte tem um significado:
ParteO que émongodb+srv://Protocolo de conexão. O +srv indica que usa DNS para descobrir os endereços dos nós do cluster automaticamenteusuario:senhaCredenciais do usuário criado no Atlas (não é a conta da pessoa, é um usuário do banco)@cluster0.7v3xmzm.mongodb.netEndereço do servidor remoto onde o cluster está hospedado/crud_dbNome do banco de dados específico que será usado dentro do cluster
Por que o .env não vai para o GitHub
O arquivo .env contém dados que, se expostos publicamente, permitiriam que qualquer pessoa acessasse o banco de dados da aplicação — podendo ler, modificar ou deletar todos os dados. Por isso ele é listado no .gitignore, que instrui o Git a ignorá-lo completamente.
A relação direta com a troca de banco é que, ao migrar de local para remoto, a única coisa que muda no código é o valor da variável MONGO_URI dentro do .env. O código da aplicação (index.js, product.controller.js etc.) não precisa ser alterado — ele apenas lê process.env.MONGO_URI, seja qual for o valor. Isso é exatamente o propósito das variáveis de ambiente: separar a configuração do código.
O que acontece com os dados do banco local
Os dados que estavam no banco local não vão automaticamente para o remoto. Quando conecto ao Atlas pela primeira vez, o banco crud_db começa vazio — ele só ganha coleções e documentos quando a aplicação começa a fazer inserções.
Isso acontece porque o banco local e o banco remoto são dois servidores completamente independentes. Não existe sincronização automática entre eles. Se eu quisesse migrar os dados, precisaria exportar os documentos do banco local (usando mongoexport) e importá-los no Atlas (usando mongoimport). Para fins deste projeto, optei por começar com o banco remoto zerado e criar novos registros via requisições HTTP.
Erros encontrados no processo e como resolvi
Erro 1: Timeout na conexão
Logo que troquei a MONGO_URI para a string do Atlas, o servidor não conseguia conectar e ficava em timeout. O erro no terminal era algo como MongoNetworkError: connect ETIMEDOUT.
O problema era que meu IP não estava liberado no Network Access do Atlas. Resolvi acessando o painel do Atlas, indo em Security > Network Access e adicionando meu IP atual (ou liberando todos com 0.0.0.0/0 para desenvolvimento).
Erro 2: Senha com caracteres especiais na URL
Minha senha continha o caractere * (asterisco), que é um caractere especial em URLs. Quando coloquei a string diretamente na URL, o Mongoose não conseguia parsear corretamente.
A solução foi fazer o URL encoding do caractere especial: * vira %2A. Então a senha Thunderbolts*2025 ficou como Thunderbolts%2A2025 dentro da connection string. Isso é um comportamento padrão de URLs — caracteres especiais precisam ser codificados para não quebrarem a estrutura do endereço.

Estrutura do Projeto
crud-api-products/
├── controllers/
│   └── product.controller.js   # Lógica das operações CRUD
├── models/
│   └── product.model.js        # Schema Mongoose (equivalente ao schema.prisma)
├── routes/
│   └── product.routes.js       # Definição das rotas HTTP
├── .env                        # Variáveis de ambiente (NÃO vai ao GitHub)
├── .gitignore                  # Inclui .env e node_modules
├── index.js                    # Ponto de entrada, conexão com o banco
└── package.json

Tecnologias Utilizadas

Node.js — ambiente de execução JavaScript no servidor
Express.js — framework para criação das rotas HTTP
MongoDB — banco de dados NoSQL orientado a documentos
Mongoose — ODM (Object Data Modeling) para modelar os dados no MongoDB
MongoDB Atlas — serviço de hospedagem gerenciada do MongoDB na nuvem
Dotenv — gerenciamento de variáveis de ambiente

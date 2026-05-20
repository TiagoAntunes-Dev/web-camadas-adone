# ADO 1 · Hospedagem de Banco de Dados

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-43853D?style=for-the-badge&logo=node.js&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/Express.js-404D59?style=for-the-badge&logo=express&logoColor=white" alt="Express" />
  <img src="https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white" alt="MongoDB" />
  <img src="https://img.shields.io/badge/Mongoose-880000?style=for-the-badge&logo=mongoose&logoColor=white" alt="Mongoose" />
</p>

**Disciplina:** Aplicações Web em Camadas  
**Projeto:** Product CRUD API  
**Aluno:** Tiago Antunes Paz de Oliveira  

---

## Índice
1. [Configuração Local do Banco de Dados](#1-configuração-local-do-banco-de-dados)
2. [Opções de Hospedagem Gratuita Pesquisadas](#2-opções-de-hospedagem-gratuita-pesquisadas)
3. [A Hospedagem Escolhida: MongoDB Atlas](#3-a-hospedagem-escolhida-mongodb-atlas--passo-a-passo)
4. [O que Muda do Local para o Remoto](#4-o-que-muda-do-local-para-o-remoto)
5. [Estrutura do Projeto](#5-estrutura-do-projeto)
6. [Tecnologias Utilizadas](#6-tecnologias-utilizadas)

---

## 1. Configuração Local do Banco de Dados

### 💻 Como estava o `.env` localmente
Durante o desenvolvimento, o banco de dados era acessado localmente. A variável de ambiente no arquivo `.env` ficava assim:

```env
# Versão local (Localhost rodando na própria máquina)
PORT = 4000

```

O arquivo `.env` **nunca é enviado ao GitHub**, porque ele contém credenciais sensíveis — como usuário, senha e o endereço do banco. Por isso ele está listado no `.gitignore`.

### Banco rodando localmente
 ```
C:\Users\tiago\OneDrive\Documentos\VsCode Journey Begin's\Curso em Video\HTML\crud-api-products>npm run dev
> crud-api-express@1.0.0 dev
> nodemon index.js
 ✅ Connected to Database!
 Server is running on port mongodb://localhost:27017/crud_db
```
---

### Schema / Model do projeto
Meu projeto não usa Prisma — ele usa **Mongoose**, que é a biblioteca de modelagem de objetos (ODM) para MongoDB. O equivalente ao `schema.prisma` no meu projeto é o arquivo `product.model.js`:

```javascript
const mongoose = require('mongoose');

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
```

---

## 2. Entendo por que usar o MongoDB para Hospedagem.

### MongoDB Atlas

**O que oferece no plano gratuito:**  
O MongoDB Atlas oferece um cluster gratuito chamado M0 (Shared Free Tier) com 512 MB de armazenamento, conexões compartilhadas e sem expiração — o cluster não "dorme" como em outros serviços. A limitação principal é o armazenamento e o desempenho compartilhado com outros usuários do plano gratuito.

**Compatível com Mongoose (equivalente ao Prisma para MongoDB)?**  
Sim. O Mongoose se conecta ao Atlas pela connection string padrão `mongodb+srv://`. O processo é simples: cria-se o cluster, o Atlas gera a URI de conexão, e basta colá-la no `.env`.

**Por que escolhi:**  
Escolhi o MongoDB Atlas porque ele é o serviço oficial do próprio MongoDB. A integração com Mongoose é nativa, a documentação é excelente e o cluster M0 gratuito não tem data de expiração. Além disso, o Atlas já é amplamente utilizado em tutoriais e cursos, o que facilita encontrar suporte.

---

## 3. MongoDB Atlas — Passo a Passo para criar minha conta

### 3.1 Criação da conta e do banco

1. Acessei [cloud.mongodb.com](https://cloud.mongodb.com) e criei uma conta gratuita.
2. Criei uma organização e um projeto chamado **"Project Crud"**.
3. Escolhi criar um cluster gratuito (M0 Shared), selecionando a região **AWS / Sao Paulo (sa-east-1)** para menor latência.
4. O Atlas criou o cluster **Cluster0** automaticamente.

<br>

<p align="center">
<img width="521" height="232" alt="image" src="https://github.com/user-attachments/assets/28d53861-74ac-4367-bbce-ee3317e051b7"/>
</p>

<br>

### 3.2 Configuração de acesso

Após criar o cluster, configurei dois pontos obrigatórios para conseguir conectar:

- **Database User:** Criei um usuário com nome e senha específicos para a aplicação (não usei minha conta pessoal do Atlas).
- **Network Access (IP Whitelist):** Adicionei `0.0.0.0/0` para permitir conexão de qualquer IP durante o desenvolvimento. Em produção isso deveria ser restrito.

<br>

<p align="center">
<img width="733" height="135" alt="image" src="https://github.com/user-attachments/assets/c1898526-ecfb-49d5-88b1-b351109686a8" />

</p>

<br>

### 3.3 Onde encontrar a connection string

Dentro do painel do Atlas:
1. Cliquei em **"Connect"** no cluster.
2. Selecionei **"Drivers"** e escolhi Node.js.
3. O Atlas gerou a connection string no formato.
4. Substituí a URI local pela URI do Atlas no arquivo `.env`:

```
mongodb+srv://<usuario>:<senha>@cluster0.xxxxxxx.mongodb.net/<banco>
```


<br>

### 3.4 Criação das coleções no Banco

Como meu projeto usa Mongoose (e não Prisma), não existe um comando `prisma migrate deploy`. As coleções são criadas automaticamente pelo Mongoose quando o primeiro documento é inserido — esse é o comportamento padrão do MongoDB.

Para garantir que o banco remoto estava funcionando, subi o servidor e fiz uma requisição POST criando um produto. O Atlas criou automaticamente o banco `crud_db` e a coleção `products`.

<br>
<br>

<p align="center">
<img width="666" height="212" alt="image" src="https://github.com/user-attachments/assets/94826f5d-daf0-43f6-8586-b3b7ceec5764" />
<p>

<br>

### 3.6 Requisição funcionando com o banco remoto

Com o servidor apontando para o Atlas, testei as rotas VSCode:

<p align="center">
<img width="960" height="484" alt="image" src="https://github.com/user-attachments/assets/5fb4c926-61d8-4c2e-b766-b172b752cd6e" />
<br>
<img width="922" height="445" alt="image" src="https://github.com/user-attachments/assets/7b8b5cbc-a60c-43a9-9798-ba9b90c66e9a" />
</p>

<br>

## 4. O que Muda do Local para o Remoto

### O que é uma connection string e o que cada parte significa

Uma connection string é o endereço completo que o código usa para encontrar, autenticar e se conectar ao banco de dados. É como o "endereço postal" do banco. No caso do MongoDB Atlas, ela tem este formato:

```
mongodb+srv://usuario:senha@cluster0.7v3xmzm.mongodb.net/crud_db
```

Cada parte tem um significado:

| Parte | O que é |
|---|---|
| `mongodb+srv://` | Protocolo de conexão. O `+srv` indica que usa DNS para descobrir os endereços dos nós do cluster automaticamente |
| `usuario:senha` | Credenciais do usuário criado no Atlas (não é a conta da pessoa, é um usuário do banco) |
| `@cluster0.7v3xmzm.mongodb.net` | Endereço do servidor remoto onde o cluster está hospedado |
| `/crud_db` | Nome do banco de dados específico que será usado dentro do cluster |

<br>

### Por que o `.env` não vai para o GitHub

O arquivo `.env` contém dados que, se expostos publicamente, permitiriam que qualquer pessoa acessasse o banco de dados da aplicação — podendo ler, modificar ou deletar todos os dados. Por isso ele é listado no `.gitignore`, que instrui o Git a ignorá-lo completamente.

A relação direta com a troca de banco é que, ao migrar de local para remoto, a única coisa que muda no código é o valor da variável `MONGO_URI` dentro do `.env`. O código da aplicação (`index.js`, `product.controller.js` etc.) não precisa ser alterado — ele apenas lê `process.env.MONGO_URI`, seja qual for o valor. Isso é exatamente o propósito das variáveis de ambiente: separar a configuração do código.

<br>

### Erros encontrados no processo e como resolvi

**Erro 1: Senha com caracteres especiais na URL** 
<br>
Minha senha continha o caractere `*` (asterisco), que é um caractere especial em URLs. Quando coloquei a string diretamente na URL, o Mongoose não conseguia parsear corretamente.

<br>

**Erro 2: Conflito de porta local (Porta 3000 em uso)**
<br>
Ao tentar testar a API no Thunder Client/Postman, recebi mensagens de rota não encontrada (`Cannot GET /api/products`) ou retornos de outro projeto meu.  

* **Solução:** Descobri que outro projeto Node.js (Suki Doces) estava rodando em segundo plano e ocupando a porta `3000`. Como a porta já estava "sequestrada", a requisição nunca chegava na API nova. A solução foi alterar a variável `PORT` no arquivo `.env` para `4000` e reiniciar o servidor, garantindo que a API rodasse de forma isolada.
<br>

---

## Estrutura do Projeto

```
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
```

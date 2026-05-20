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
# Versão local (MongoDB rodando na própria máquina)
MONGO_URI=mongodb://localhost:27017/crud_db

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

<p> .</p>

<p align="center">
<img width="521" height="232" alt="image" src="https://github.com/user-attachments/assets/28d53861-74ac-4367-bbce-ee3317e051b7"/>
</p>



### 3.2 Configuração de acesso

Após criar o cluster, configurei dois pontos obrigatórios para conseguir conectar:

- **Database User:** Criei um usuário com nome e senha específicos para a aplicação (não usei minha conta pessoal do Atlas).
- **Network Access (IP Whitelist):** Adicionei `0.0.0.0/0` para permitir conexão de qualquer IP durante o desenvolvimento. Em produção isso deveria ser restrito.

> 📸 **[INSIRA AQUI: print da tela de Network Access mostrando o IP liberado]**

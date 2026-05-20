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

### Banco rodando localmente

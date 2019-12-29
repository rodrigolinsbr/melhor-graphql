# melhor-graphql
Hello world GraphQL

### https://blog.rocketseat.com.br/iniciando-graphql-nodejs-expressjs/
Já falamos um pouco sobre ferramentas de consumo de dados de API com GraphQL como o Apollo e Relay, agora chegou o momento de mostrarmos um pouco como funciona um serviço com GraphQL.

No post de hoje criaremos um servidor simples com GraphQL e Express.js, mas que servira de base para estruturas mais complexas.

# GraphQL
O GraphQL é uma linguagem de consultas de dados para APIs que visa passar a responsabilidade do pedido de dados para o lado do cliente, o cliente escolhe o que quer e o servidor entrega. No servidor ele é estruturado por 3 pilares principais, o Schema, que define os tipos de dados e quais serão as requisições, o Resolver, é a função que devolve os dados para o cliente, e o Provider, definindo onde os dados estarão guardados.

# Mão na massa
Crie um projeto com o node.js e comece instalando os packages necessários para a brincadeira:

`npm install express graphql express-graphql`

Crie também um arquivo index.js e importe as dependências:

``
const app = require("express")();
const expressGraphql = require("express-graphql");
const { buildSchema } = require("graphql");``

Agora é preciso definir como serão os dados trabalhados dentro do GraphQL e quais requisições existirão, com a função buildSchema é possível criar essa estrutura:

```
const schema = buildSchema('
  type User {
    id: ID
    name: String
    repo: String
    age: Int
  }
  type Query {
    user(id: ID!): User
    users: [User]
  }
  type Mutation {
    createUser(name: String!, repo: String!, age: Int!): User
  }
');

```

O type User define uma estrutura  para o Schema, lembrando que o GraphQL é fortemente tipado, por isso em frente aos campos existe os tipos. No Schema existem 3 types que são especiais: Query, Mutation e Subscription. Neste post trabalharemos apenas com o Query e Mutationque serve para consultar dados e enviar dados respectivamente.

Dentro deles os campos podem ser definidos como se fossem funções, então por isso existe o user(id: ID!) que serve para resgatar um Userpelo id, lembrando que esse ponto de exclamação é pra dizer que é obrigatório ter um argumento. Mas não é obrigado a sempre a ter um argumento, como o caso do users que retorna um Array de usuários, por isso o [].

Talvez para ficar mais claro, compare o que está dentro das requisições Query e Mutation com rotas e os argumentos com os parâmetros delas, isso pensando numa API REST.

Defina o provider:

`
const providers = {
  users: []
};
`

É importante observar que ele é apenas um objeto com um item usersque contém um Array, e com o resolver createUser será povoado com o users. Nesse caso especial, o provider está em memória e você poderia facilmente usar o Sequelize ou o Mongoose para poder trabalhar com banco dados. Observe também que dentro do resolver é que o provider será manipulado.

Ainda é preciso definir quem responderá a essas requisições, e é aqui que entra os resolvers:

```
let id = 0;

const resolvers = {
  user({ id }) {
    return providers.users.find(item => item.id === Number(id));
  },
  users() {
    return providers.users;
  },
  createUser({ name, repo, age }) {
    const user = {
      id: id++,
      name,
      repo,
      age
    };

    providers.users.push(user);

    return user;
  }
};

```
A observação a ser feita aqui é que os resolvers são funções que possuem o mesmo nome das requisições que estão em Mutation e Query, e é possível desestruturar o primeiro parâmetro do resolver para poder pegar os parâmetros definidos lá nas requisições.

A título de comparação o resolver se equipara a um controller.

Para finalizar, é preciso configurar o middleware do GraphQL para funcionar com o express.


```
app.use(
  "/graphql",
  expressGraphql({
    schema,
    rootValue: resolvers,
    graphiql: true
  })
);

app.listen(3000);
```

O middleware será usado na rota /graphql e recebe alguns parâmetros, o schema para definir o Schema o rootValue os resolvers e por fim o graphiql que habilita uma interface gráfica para servir de playground do servidor GraphQL e em modo de produção é recomendado desabilita-la.

`node index.js`

Agora no navegador, http://localhost:3000/graphql você pode adicionar um novo usuário:


Você pode listar também os usuários cadastrados:


Observe que ambos os casos os valores retornados foram queles pedido na requisição. Por isso é dito que o GraphQL passa a responsabilidade para o cliente, os dados que ele pedir virá, logicamente que respeitando o Schema.

# Examples

### Adicionando usuário

```
mutation{
  createUser(
    name:"Rodrigo",
    repo:"rodrigolinsbr",
    age:36
  ){
    id
  }
}
```
### Listando os nomes dos usuários

```
query{
  users{
    name
  }
}

```

### Listando os nomes e idades dos usuários

```
query{
  users{
    name,age
  }
}

```

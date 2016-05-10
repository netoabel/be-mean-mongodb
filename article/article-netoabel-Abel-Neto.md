# MongoDb - Artigo sobre Autenticação no MongoDb
**Autor:** Abel Neto

**Data:** 1462832546308

## Autenticação versus Autorização

O termo _autenticação_ é utilizado para designar o processo de verificação de credenciais do usuário e permitir ou não o seu acesso ao sistema. Esse processo consiste no envio de credenciais (em texto plano ou criptografadas)  para um serviço de autenticação, que irá determinar se o usuário é realmente quem ele diz que é.

_Autorização_, por outro lado, é algo que acontece somente após uma _autenticação_ bem sucedida, ou seja, quando um usuário foi validado como tendo credenciais válidas. O sistema então verifica se o usuário possui _autorização_ para acessar a funcionalidade do sistema à qual está solicitando acesso. Em geral, esquemas de _autorização_ são definidos através de listas de permissões atribuidas a cada usuário do sistema.

## Como criar um usuário no MongoDb

Para criar um usuário no Mongo, utilizamos o método `db.createUser()`, que recebe como parâmetro obrigatório um documento contendo as informações do usuário, com a seguinte estrutura:

```json
{ 
  user: "<nome do usuário>",
  pwd: "<senha em texto plano>",
  customData: { <qualquer informação> },
  roles: [
    { role: "<regra>", db: "<database>" } | "<regra>"
  ]
}
```

Onde:

    user        - Nome do usuário;
    pwd         - Senha;
    customData  - Propriedade opcional que recebe um documento e pode ser usada para armazenar dados adicionais de qualquer tipo;
    roles       - As regras de permissões do usuário. 

A propriedade `roles` deve receber um array, que pode ter, em cada posição, uma string ou um documento. 

No caso em que é armazenada uma string nesse array, esta representa o nome da regra, que terá efeito sobre o novo usuário somente na database selecionada no momento da sua criação. 

Já quando um documento é adicionado ao array, ele especifica o nome da regra e também a database onde ela terá efeito para o usuário criado, da seguinte forma:

```json
{ role: "<regra>", db: "<database>" }
```

No exemplo abaixo, temos um comando de criação de um usuário comum (sem roles):

```json
db.createUser(
{ 
  user : "netoabel",
  pwd: "senha123",
  customData : { twitter: "@_netoabel" }
}, 
{ w: "majority" , wtimeout: 5000 }
)
```

O documento da penúltima linha é o segundo parâmetro, opcional, que o método `db.createUser()` pode receber: o [WriteConcern](https://docs.mongodb.com/manual/reference/write-concern/), que representa o nível de confirmação exigido pelo MongoDB nas operações de escrita e pode ser ajustado visando desempenho ou consistência dos dados.

## Como criar um usuário administrador

Para criar um usuário com permissões de administrador em todas as databases, basta criá-lo com a regra `userAdminAnyDatabase` na database `admin`:

```json
db.createUser({
  user: "netoabel",
  pwd: "senha123",
  roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
})
```

## Regras de administração de cluster

#### clusterAdmin

#### clusterManager

#### clusterMonitor

#### hostManager

## Explique todas as ações de privilégio listadas em Privilege Actions.


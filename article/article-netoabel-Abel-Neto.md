# MongoDb - Artigo sobre Autenticação no MongoDb
**Autor:** Abel Neto

**Data:** 1462832546308

## Autenticação versus Autorização

O termo _autenticação_ é utilizado para designar o processo de verificação de credenciais do usuário e permitir ou não o seu acesso ao sistema. Esse processo consiste no envio de credenciais (em texto plano ou criptografadas)  para um serviço de autenticação, que irá determinar se o usuário é realmente quem ele diz que é.

_Autorização_, por outro lado, é algo que acontece somente após uma _autenticação_ bem sucedida, ou seja, quando um usuário foi validado como tendo credenciais válidas. O sistema então verifica se o usuário possui _autorização_ para acessar a funcionalidade do sistema à qual está solicitando acesso. Em geral, esquemas de _autorização_ são definidos através de listas de permissões atribuidas a cada usuário do sistema.

## Como criar um usuário no MongoDb

Para criar um usuário no Mongo, utilizamos o método `db.createUser()`, que recebe como parâmetro obrigatório um documento contendo as informações do usuário, com a seguinte estrutura:

```javascript
{ 
  user: "<nome do usuário>",
  pwd: "<senha em texto plano>",
  customData: { <qualquer informação> },
  roles: [
    { role: "<nome da permissão>", db: "<database>" } | "<papel>"
  ]
}
```

Onde:

    user        - Nome do usuário;
    pwd         - Senha;
    customData  - Propriedade opcional que recebe um documento e pode ser usada para armazenar dados adicionais de qualquer tipo;
    roles       - As permissões do usuário. 

A propriedade `roles` deve receber um array, que pode ter, em cada posição, uma string ou um documento. 

No caso em que é armazenada uma string nesse array, esta representa o nome da permissão, que terá efeito sobre o novo usuário somente na database selecionada no momento da sua criação. 

Já quando um documento é adicionado ao array, ele especifica o nome da permissão e também a database onde ela terá efeito para o usuário criado, da seguinte forma:

```javascript
{ role: "<nome da permissão>", db: "<database>" }
```

No exemplo abaixo, temos um comando de criação de um usuário comum (sem roles):

```javascript
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

Para criar um usuário com permissões de administrador em todas as databases, basta criá-lo com a permissão `userAdminAnyDatabase` na database `admin`:

```javascript
db.createUser({
  user: "netoabel",
  pwd: "senha123",
  roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
})
```

## Permissões de administração de cluster

Usuários com permissões de administração de cluster são capazes de administrar várias instâncias do MongoDB organizadas em cluster, em vez de somente databases específicas em um servidor. 

#### clusterAdmin
Nível mais alto de gerenciamento de cluster. É equivalente a uma combinação das permissões `clusterManager`, `clusterMonitor` e `hostManager`, além de também conceder permissão para a ação `dropDatabase`.

#### clusterManager
Autoriza ações de monitoramento e gerenciamento no cluster. Um usuário com essa permissão pode acessar as databases `config` e `local`, que são usadas em sharding e replicação, respectivamente.

Provê acesso às seguintes ações no cluster como um todo:

* addShard
* applicationMessage
* cleanupOrphaned
* flushRouterConfig
* listShards
* removeShard
* replSetConfigure
* replSetGetStatus
* replSetStateChange
* resync

Além das seguintes ações em todas as databases do cluster:

* enableSharding
* moveChunk
* splitChunk
* splitVector

Na collection `settings` da database `config`, provê acesso às seguintes ações:

* insert
* remove
* update

Ainda na database `config`, nas collections `system.indexes`, `system.js`, `system.namespaces` e em todas as collections de configuração, provê acesso às seguntes ações:

* collStats
* dbHash
* dbStats
* find
* killCursors

Na database `local`, provê acesso às seguintes acões na collection `replset`:

* collStats
* dbHash
* dbStats
* find
* killCursors

#### clusterMonitor

Autoriza acess somente leitura a ferramentas de monitoração, como o [MongoDB Cloud Manager](https://cloud.mongodb.com/?jmp=docs&_ga=1.132013837.389647211.1462567638).

Provê as seguintes permissões no cluster como um todo:

* connPoolStats
* cursorInfo
* getCmdLineOpts
* getLog
* getParameter
* getShardMap
* hostInfo
* inprog
* listDatabases
* listShards
* netstat
* replSetGetStatus
* serverStatus
* shardingState
* top

Provê as seguintes permissões em todas as databases do cluster:

* collStats
* dbStats
* getShardVersion

Provê a ação `find` em todas as collections `system.profile` do cluster. 

Nas collections `system.indexes`, `system.js`, `system.namespaces` e em todas as collections de configuração da database `config`, provê acesso às seguntes ações:

* collStats
* dbHash
* dbStats
* find
* killCursors

#### hostManager

Autoriza o monitoramento e gerenciamento de servidores.

Provê as seguintes ações no cluster como um todo:* 

* applicationMessage
* closeAllDatabases
* connPoolSync
* cpuProfiler
* diagLogging
* flushRouterConfig
* fsync
* invalidateUserCache
* killop
* logRotate
* resync
* setParameter
* shutdown
* touch
* unlock

Disponibiliza as seguintes ações em todas as databases do cluster:

* killCursors
* repairDatabase

## Ações de privilégio




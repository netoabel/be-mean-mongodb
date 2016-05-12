
# MongoDb - Projeto Final
**Autor:** Abel Neto

**Data:** 1462977026837

## Caso de uso onde seria interessante utilizar o MongoDB

MongoDB é um banco de dados NoSQL, o que significa que ele não segue a implementação mais tradicional e conhecida de bancos de dados em geral, a relacional. Isso pode ser bom ou ruim, a depender do cenário onde ele for utilizado. 

Um caso onde eu o utilizaria (e de fato utilizo) é o meu TCC, que é um projeto de automação residencial, onde eu armazeno informações sobre atualizações de estado dos dispositivos em uma base de dados. Como pode haver um alto volume de dados e operações de escrita e não há a necessidade de garantir transações atômicas ou alta confiabilidade, por exemplo, o MongoDB se mostrou a melhor alternativa para o caso. 

Além disso, diferentes dispositivos possuem diferentes modelagens no banco. Modelar esse tipo de estrutura é muito mais fácil no MongoDB. Incluir novas estruturas e tipos de dispositivos não é um problema, por exemplo.

## Modelagem

Vejamos como ficaria a modelagem relacional a seguir no MongoDB: 

![](https://raw.githubusercontent.com/netoabel/be-mean-instagram-mongodb-projects/master/modeling/relational.jpg)

### Collection users

```javascript
{
    name,
    bio,
    register_date,
    auth: {
        username,
        email,
        password,
        last_access,
        online,
        disabled,
        hash_token
    },
    settings: { 
        system: {
            background_path
        }
    }
}
```

### Collection projects

```javascript
{
    name,
    description,
    date_begin,
    date_dream,
    date_end,
    realocate,
    visible,
    expired,
    visualizable_mod,
    members: [
        { 
            user_id,
            member_type,
            notify
        }
    ],
    tags: [],
    goals: [
        {
            name,
            description,
            date_begin,
            date_dream,
            date_end,
            realocate,
            expired,
            tags: [],
            historic: [
                { date_realocate }
            ],
            activities: [
                { activity_id }
            ]
        }
    ]
}
```

## Collection activites

```javascript
{ 
    name,
    description,
    date_begin,
    date_dream,
    date_end,
    realocate,
    expired,
    tags: [],
    historic: [
        { date_realocate }
    ], 
    members: [
        { 
            user_id,
            member_type,
            notify
        }
    ],
    comments: [
        {
            text,
            date,
            files: [
                { path, weight, name }
            ],
            members: [
                { 
                    user_id,
                    member_type, 
                    notify 
                }
            ]
        }
    ]
}
```

## Algumas considerações

Poderíamos ter criado apenas uma collection `projects` com todos os documentos de `activities` dentro dela, como arrays. Porém, o MongoDB estabelece um limite de 16MB por documento armazenado. No nosso cenário, a probabilidade de a quantidade de atividades crescer consideravelmente é muito grande. Isso poderia facilmente levar um documento da collection `projects` a exceder o limite de tamanho estabelecido pelo Mongo.

Ao criar uma collection `activities` separada, fazemos com que cada atividade seja armazenada como um novo documento. Além de evitar a extrapolação do limite, esse formato evita que um único documento use uma quantidade excessiva de memória RAM ou consuma uma largura de banda muito grande durante uma transmissão.

No caso dos comentários, como é muito pouco provável que a quantidade de comentários em uma única atividade cresça a ponto de fazer com que um documento da collection `activities` se torne muito grande, optamos por mantê-los em um array interno a essa collection, a fim de tornar o acesso a eles mais fácil e performático.

## Inserindo usuários

```javascript
be-mean-project> var users = [];
be-mean-project> 
... for (var i = 0; i < 10; i++) {
...     var user = {
...         name: "User " + i,
...         bio: "Once upon a time...",
...         register_date: new Date(), 
...         auth: {
...             username: "user_" + i,
...             email: "user_" + i + "@gmail.com",
...             password: "password",
...             last_access: new Date(),
...             online: true,
...             disabled: false,
...             hash_token: "572f78aed7193a654cecb74b",
...         },
...         settings: { 
...             system: {
...                 background_path: "images/user_" + i + "_bg.png"
...             }
...         },
...     };
...     users.push(user);
... }
10
be-mean-project> db.users.save(users);
Inserted 1 record(s) in 355ms
BulkWriteResult({
  "writeErrors": [ ],
  "writeConcernErrors": [ ],
  "nInserted": 10,
  "nUpserted": 0,
  "nMatched": 0,
  "nModified": 0,
  "nRemoved": 0,
  "upserted": [ ]
})
```

## Inserindo projetos

Primeiro, criaremos as atividades que serão atribuídas aos projetos:

```javascript
be-mean-project> var activity = { 
...     name: "Activity 1",
...     description: "Description",
...     date_begin: new Date(),
...     date_dream: new Date(),
...     date_end: new Date(),
...     realocate: new Date(),
...     expired: false,
...     tags: ["activity"],
...     historic: [
...         { date_realocate: new Date() }
...     ], 
...     members: [
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48321"),
...             member_type: "type1",
...             notify: true
...         }
...     ],
...     comments: [
...         {
...             text: "Comment!",
...             date: new Date(),
...             files: [
...                 { 
...                     path: "files/file",
...                     weight: 10, 
...                     name: "file" 
...                 }
...             ],
...             members: [
...                 { 
...                     user_id: ObjectId("5734c206a551a48f44c48321"),
...                     member_type: "type1", 
...                     notify: true 
...                 }
...             ]
...         }
...     ]
... };
be-mean-project> db.activities.save(activity);
Inserted 1 record(s) in 3ms
WriteResult({
  "nInserted": 1
})
```

Seguindo o mesmo processo, foram inseridas 8 atividades:

```javascript
be-mean-project> db.activities.count()
8
```

Agora, inserimos os projetos:

```javascript
be-mean-project> var project = {
...     name: "Project 1",
...     description: "Description",
...     date_begin: new Date(),
...     date_dream: new Date(),
...     date_end: new Date(),
...     realocate: new Date(),
...     visible: true,
...     expired: false,
...     visualizable_mod: true,
...     members: [
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48321"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48322"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48323"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48324"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48325"),
...             member_type: "type 1",
...             notify: true
...         }
...     ],
...     tags: ['tag1', 'tag2', 'tag3'],
...     goals: [
...         {
...             name: "Goal",
...             description: "Description",
...             date_begin: new Date(),
...             date_dream: new Date(),
...             date_end: new Date(),
...             realocate: new Date(),
...             expired: false,
...             tags: ['tag1', 'tag2', 'tag3'],
...             historic: [
...                 { date_realocate: new Date() }
...             ],
...             activities: [
...                 { activity_id: ObjectId("5734d192a551a48f44c4832b") },
...                 { activity_id: ObjectId("5734d287a551a48f44c4832d") }
...             ]
...         }
...     ]
... };
be-mean-project> db.projects.save(project);
Inserted 1 record(s) in 4ms
WriteResult({
  "nInserted": 1
})

be-mean-project> var project = {
...     name: "Project 2",
...     description: "Description",
...     date_begin: new Date(),
...     date_dream: new Date(),
...     date_end: new Date(),
...     realocate: new Date(),
...     visible: true,
...     expired: false,
...     visualizable_mod: true,
...     members: [
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48322"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48323"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48324"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48325"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48326"),
...             member_type: "type 1",
...             notify: true
...         }
...     ],
...     tags: ['tag2', 'tag3', 'tag4'],
...     goals: [
...         {
...             name: "Goal",
...             description: "Description",
...             date_begin: new Date(),
...             date_dream: new Date(),
...             date_end: new Date(),
...             realocate: new Date(),
...             expired: false,
...             tags: ['tag1', 'tag2', 'tag3'],
...             historic: [
...                 { date_realocate: new Date() }
...             ],
...             activities: [
...                 { activity_id: ObjectId("5734d2a0a551a48f44c4832e") },
...                 { activity_id: ObjectId("5734d2bda551a48f44c4832f") }
...             ]
...         }
...     ]
... };
be-mean-project> db.projects.save(project);
Inserted 1 record(s) in 3ms
WriteResult({
  "nInserted": 1
})

be-mean-project> var project = {
...     name: "Project 3",
...     description: "Description",
...     date_begin: new Date(),
...     date_dream: new Date(),
...     date_end: new Date(),
...     realocate: new Date(),
...     visible: true,
...     expired: false,
...     visualizable_mod: true,
...     members: [
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48323"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48324"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48325"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48326"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48327"),
...             member_type: "type 1",
...             notify: true
...         }
...     ],
...     tags: ['tag3', 'tag4', 'tag5'],
...     goals: [
...         {
...             name: "Goal",
...             description: "Description",
...             date_begin: new Date(),
...             date_dream: new Date(),
...             date_end: new Date(),
...             realocate: new Date(),
...             expired: false,
...             tags: ['tag1', 'tag2', 'tag3'],
...             historic: [
...                 { date_realocate: new Date() }
...             ],
...             activities: [
...                 { activity_id: ObjectId("5734d2daa551a48f44c48330") },
...                 { activity_id: ObjectId("5734d2ffa551a48f44c48331") }
...             ]
...         }
...     ]
... };
be-mean-project> db.projects.save(project);
Inserted 1 record(s) in 3ms
WriteResult({
  "nInserted": 1
})

be-mean-project> var project = {
...     name: "Project 4",
...     description: "Description",
...     date_begin: new Date(),
...     date_dream: new Date(),
...     date_end: new Date(),
...     realocate: new Date(),
...     visible: true,
...     expired: false,
...     visualizable_mod: true,
...     members: [
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48324"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48325"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48326"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48327"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48328"),
...             member_type: "type 1",
...             notify: true
...         }
...     ],
...     tags: ['tag4', 'tag5', 'tag6'],
...     goals: [
...         {
...             name: "Goal",
...             description: "Description",
...             date_begin: new Date(),
...             date_dream: new Date(),
...             date_end: new Date(),
...             realocate: new Date(),
...             expired: false,
...             tags: ['tag1', 'tag2', 'tag3'],
...             historic: [
...                 { date_realocate: new Date() }
...             ],
...             activities: [
...                 { activity_id: ObjectId("5734d319a551a48f44c48332") },
...                 { activity_id: ObjectId("5734d332a551a48f44c48333") }
...             ]
...         }
...     ]
... };
be-mean-project> db.projects.save(project);
Inserted 1 record(s) in 2ms
WriteResult({
  "nInserted": 1
})

be-mean-project> var project = {
...     name: "Project 5",
...     description: "Description",
...     date_begin: new Date(),
...     date_dream: new Date(),
...     date_end: new Date(),
...     realocate: new Date(),
...     visible: true,
...     expired: false,
...     visualizable_mod: true,
...     members: [
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48325"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48326"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48327"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48328"),
...             member_type: "type 1",
...             notify: true
...         },
...         { 
...             user_id: ObjectId("5734c206a551a48f44c48329"),
...             member_type: "type 1",
...             notify: true
...         }
...     ],
...     tags: ['tag5', 'tag6', 'tag7'],
...     goals: [
...         {
...             name: "Goal",
...             description: "Description",
...             date_begin: new Date(),
...             date_dream: new Date(),
...             date_end: new Date(),
...             realocate: new Date(),
...             expired: false,
...             tags: ['tag1', 'tag2', 'tag3'],
...             historic: [
...                 { date_realocate: new Date() }
...             ]
...         }
...     ]
... };
be-mean-project> db.projects.save(project);
Inserted 1 record(s) in 3ms
WriteResult({
  "nInserted": 1
})
```

## Retrieve - busca

## Update - alteração

## Delete - remoção

## Sharding
// coloque aqui todos os comandos que você executou

## Replica
// coloque aqui todos os comandos que você executou

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

### Collection `users`

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
        hash_token,
    }
    settings: { 
        system: {
            background_path
        }
    }
    projects: [
        { 
            project_id , 
            member_type
        }
    ]
}
```

### Collection `projects`

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
        { goal_id }
    ]
}
```


## Collection `goals`

```javascript
{
    name,
    description,
    project_id,
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
    ]
}
```

## Create - cadastro

## Retrieve - busca

## Update - alteração

## Delete - remoção

## Sharding
// coloque aqui todos os comandos que você executou

## Replica
// coloque aqui todos os comandos que você executou
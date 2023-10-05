# people-management
Cadastro de pessoas e suas stacks.

**Observação:** Diversas das ideias utilizadas nesse projeto foram baseadas no repositório [rinha-de-backend-2023-q3](https://raw.githubusercontent.com/zanfranceschi/rinha-de-backend-2023-q3).

# Instruções

Abaixo serão apresentadas as instruções necessárias para criação da aplicação.

## Endpoints

Resumo dos endpoints:

- `POST /people` – para criar um recurso pessoa.
- `GET /people/[:id]` – para consultar um recurso criado com a requisição anterior.
- `GET /people?t=[:termo da busca]` – para fazer uma busca por pessoas.
- `GET /people/count` – endpoint especial para contagem de pessoas cadastradas.


### Criação de Pessoas
`POST /people`

Deverá aceitar uma requisição em formato JSON com os seguintes parâmetros:

| atributo | descrição |
| --- | --- |
| **nickname** | obrigatório, único, string de até 32 caracteres. |
| **name** | obrigatório, string de até 100 caracteres. |
| **nascimento** | obrigatório, string para data no formato YYYY-MM-DD (ano, mês, dia). |
| **stack** | opcional, vetor de string com cada elemento sendo obrigatório e de até 32 caracteres. |

Para requisições válidas, sua API deverá retornar status code 201 - created junto com o header "Location: /people/[:id]" onde [:id] é o id – em formato UUID com a versão a seu critério – da pessoa que acabou de ser criada. O conteúdo do corpo fica a seu critério; retorne o que quiser. 

Exemplos de requisições válidas:
```json
{
    "nickname" : "josé",
    "name" : "José Roberto",
    "birthDate" : "2000-10-01",
    "stack" : ["C#", "Node", "Oracle"]
}
```

```json
{
    "nickname" : "ana",
    "name" : "Ana Barbosa",
    "birthDate" : "1985-09-23",
    "stack" : null
}
```
Para requisições inválidas, o status code deve ser 422 - Unprocessable Entity/Content. Aqui, novamente, o conteúdo do corpo fica a seu critério.

Exemplos de requisições inválidas:
```json
{
    "nickname" : "josé", // caso "josé" já tenha sido criado em outra requisição
    "name" : "José Roberto",
    "birthDate" : "2000-10-01",
    "stack" : ["C#", "Node", "Oracle"]
}
```

```json
{
    "nickname" : "ana",
    "name" : null, // não pode ser null
    "birthDate" : "1985-09-23",
    "stack" : null
}
```

```json
{
    "nickname" : null, // não pode ser null
    "name" : "Ana Barbosa",
    "birthDate" : "1985-01-23",
    "stack" : null
}
```

Para o caso de requisições sintaticamente inválidas, a resposta deverá ter o status code para 400 - bad request. Exemplos:

```json
{
    "nickname" : "nickname",
    "name" : 1, // nome deve ser string e não número
    "birthDate" : "1985-01-01",
    "stack" : null
}
```

```json
{
    "nickname" : "nickname",
    "name" : "name",
    "birthDate" : "1985-01-01",
    "stack" : [1, "PHP"] // stack deve ser um array de apenas strings
}
```

### Detalhe de uma Pessoa
`GET /people/[:id]`

Deverá retornar os detalhes de uma pessoa caso esta tenha sido criada anteriormente. O parâmetro [:id] deve ser do tipo UUID na versão que escolher. O retorno deve ser como os exemplos a seguir.


```json
{
    "id" : "f7379ae8-8f9b-4cd5-8221-51efe19e721b",
    "nickname" : "josé",
    "name" : "José Roberto",
    "birthDate" : "2000-10-01",
    "stack" : ["C#", "Node", "Oracle"]
}
```

```json
{
    "id" : "5ce4668c-4710-4cfb-ae5f-38988d6d49cb",
    "nickname" : "ana",
    "name" : "Ana Barbosa",
    "birthDate" : "1985-09-23",
    "stack" : null
}
```

Note que a resposta é praticamente igual ao payload de criação com o acréscimo de `id`. O status code para pessoas que existem deve ser 200 - Ok. Para recursos que não existem, deve-se retornar 404 - Not Found.


### Busca de Pessoas
`GET /people?t=[:termo da busca]`

Dado o `termo da busca`, a resposta deverá ser uma lista que satisfaça o termo informado estar contido nos atributos `nickname`, `nome`, e/ou elementos de `stack`. A busca não precisa ser paginada e poderá retornar apenas os 50 primeiros registros resultantes da filtragem para facilitar a implementação.

O status code deverá ser sempre 200 - Ok, mesmo para o caso da busca não retornar resultados (vazio).

Exemplos: Dado os recursos seguintes existentes em sua aplicação:

```json
[{
    "id" : "f7379ae8-8f9b-4cd5-8221-51efe19e721b",
    "nickname" : "josé",
    "name" : "José Roberto",
    "birthDate" : "2000-10-01",
    "stack" : ["C#", "Node", "Oracle"]
},
{
    "id" : "5ce4668c-4710-4cfb-ae5f-38988d6d49cb",
    "nickname" : "ana",
    "name" : "Ana Barbosa",
    "birthDate" : "1985-09-23",
    "stack" : ["Node", "Postgres"]
}]
```

Uma requisição `GET /people?t=node`, deveria retornar o seguinte:
```json
[{
    "id" : "f7379ae8-8f9b-4cd5-8221-51efe19e721b",
    "nickname" : "josé",
    "name" : "José Roberto",
    "birthDate" : "2000-10-01",
    "stack" : ["C#", "Node", "Oracle"]
},
{
    "id" : "5ce4668c-4710-4cfb-ae5f-38988d6d49cb",
    "nickname" : "ana",
    "name" : "Ana Barbosa",
    "birthDate" : "1985-09-23",
    "stack" : ["Node", "Postgres"]
}]
```

Uma requisição `GET /people?t=berto`, deveria retornar o seguinte:
```json
[{
    "id" : "f7379ae8-8f9b-4cd5-8221-51efe19e721b",
    "nickname" : "josé",
    "name" : "José Roberto",
    "birthDate" : "2000-10-01",
    "stack" : ["C#", "Node", "Oracle"]
}]
```

Uma requisição `GET /people?t=Python`, deveria retornar o seguinte:
```json
[]
```

Se a query string `t` não for informada, a resposta deve ter seu status code para 400 - bad request com o corpo que quiser. Ou seja, informar `t` é obrigatório.

### Contagem de Pessoas - Endpoint Especial
`GET /contagem-pessoas`

Deverá retornar a quantidade de pessoas cadastradas na base, a resposta deve ter status code 200.

# Referências

- [Exemplo Docker Compose para PostgreSQL](https://github.com/khezen/compose-postgres/blob/master/docker-compose.yml)
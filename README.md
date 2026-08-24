# 🐳 Aplicação de Três Camadas com Docker

Projeto desenvolvido na disciplina de **Computação em Nuvem** com o objetivo de aplicar, na prática, conceitos de arquitetura em três camadas utilizando Docker.

## 👥 Alunos

- **Jonas Gabriel Amorim Ribeiro**
- **Lucas Leal Ferreira**

## 📚 Sobre o projeto

A aplicação utiliza uma arquitetura de três camadas, separando o **frontend**, a **API** e o **banco de dados** em serviços distintos.

O objetivo é simular com Docker uma arquitetura semelhante à utilizada em ambientes de nuvem, mantendo apenas o frontend acessível externamente e os demais serviços isolados em uma rede interna.

## 🛠️ Tecnologias utilizadas

- Docker
- Docker Compose
- Nginx
- Traefik Whoami
- Redis
- MinIO
- HTML
- GitHub Actions *(CI/CD - próxima etapa)*

## 🏗️ Arquitetura

A aplicação foi organizada para manter apenas o frontend exposto ao usuário, enquanto os demais serviços permanecem na rede interna.

```text
                    USUÁRIO
                       │
                       │ localhost:8080
                       ▼
                 ┌───────────┐
                 │ FRONTEND  │
                 │   Nginx   │
                 └─────┬─────┘
                       │
                    /api/
                       │
                       ▼
                 ┌───────────┐
                 │    API    │
                 │  Whoami   │
                 └───────────┘
                       │
                 REDE INTERNA
                       │
              ┌────────┴────────┐
              ▼                 ▼
          ┌───────┐         ┌───────┐
          │ Redis │         │ MinIO │
          │ Banco │         │Objetos│
          └───────┘         └───────┘
```

O **frontend** é o serviço exposto ao usuário. A **API**, o **Redis** e os demais serviços permanecem isolados na rede interna do Docker.

## 🧪 Desenvolvimento da atividade

### 1. Conhecendo o serviço da API

A primeira etapa consistiu em executar a imagem `traefik/whoami` como um contêiner para conhecer o funcionamento da API.

O `whoami` devolve informações do contêiner que respondeu à requisição, como o **hostname**, o **endereço IP** e alguns dados da requisição HTTP.

O hostname é útil porque identifica qual contêiner executou a resposta, permitindo comprovar qual instância atendeu determinada requisição.

### 2. Publicação da vitrine

O frontend da aplicação foi criado utilizando **Nginx** e representa a camada pública da arquitetura.

A vitrine pode ser acessada através de:

```text
http://localhost:8080
```

Somente o frontend possui uma porta publicada diretamente para o computador.

> 📷 Inserir aqui o print da vitrine funcionando em `localhost:8080`.

### 3. Publicação da API pela vitrine

A API não possui uma porta publicada diretamente para o computador.

O acesso é realizado através do frontend pela rota:

```text
http://localhost:8080/api/
```

O Nginx funciona como um **proxy reverso**, recebendo a requisição do usuário e encaminhando-a para a API através da rede interna do Docker.

```text
Usuário
   │
   ▼
localhost:8080/api/
   │
   ▼
Nginx
   │
   │ Rede interna
   ▼
API
```

A API sem porta publicada imita o funcionamento de uma **sub-rede privada**, pois ela não pode ser acessada diretamente pelo computador de fora.

O acesso acontece passando pelo frontend, utilizando a rota `/api/` e o proxy reverso do Nginx. Assim, somente o frontend fica exposto e a API continua protegida dentro da rede interna.

> 📷 Inserir aqui o print da API funcionando através de `/api/`.

### 4. Banco de dados privado

O banco de dados utilizado na aplicação é o **Redis**.

Assim como a API, o banco permanece na rede interna e não possui uma porta publicada diretamente para o computador.

Para testar a comunicação com o Redis, pode ser utilizado:

```bash
redis-cli ping
```

Quando o serviço está funcionando corretamente, a resposta esperada é:

```text
PONG
```

Isso demonstra que os serviços conseguem se comunicar internamente sem a necessidade de deixar o banco exposto externamente.

> 📷 Inserir aqui o print mostrando a resposta `PONG`.

### 5. Teste de isolamento de falhas por camada

Nesta etapa, a API foi interrompida propositalmente para observar o comportamento das diferentes camadas da aplicação.

Mesmo com a API indisponível, a vitrine continuou funcionando normalmente.

Por outro lado, ao tentar acessar `/api/`, a requisição apresentou erro porque o Nginx não conseguiu se comunicar com a API.

```text
Frontend → funcionando
API      → indisponível
Banco    → isolado
```

Esse comportamento demonstra o **isolamento entre as camadas**, pois uma falha na API não necessariamente derruba o frontend da aplicação.

> 📷 Inserir aqui o print da vitrine funcionando enquanto `/api/` apresenta erro.

## 💾 Persistência do banco

Foi utilizado um volume para manter os dados do banco separados do ciclo de vida do contêiner.

### O que aconteceria com os dados sem o volume?

Sem o volume persistente, os dados armazenados dentro do contêiner poderiam ser perdidos quando ele fosse removido.

O volume funciona de forma semelhante a um **armazenamento de bloco**, pois fornece um espaço persistente separado da vida útil do contêiner.

```text
Contêiner do banco
        │
        ▼
      Volume
        │
        ▼
Dados persistentes
```

Assim, mesmo que o contêiner do banco seja destruído e recriado, os dados permanecem armazenados, aumentando a durabilidade das informações.

## 🪣 Armazenamento de objetos com MinIO

As imagens dos produtos são armazenadas utilizando **MinIO**.

As imagens são arquivos não estruturados e, por isso, são mais adequadas ao **armazenamento de objetos**.

Esse tipo de armazenamento foi projetado para arquivos como:

- Imagens
- Vídeos
- Documentos
- Backups

No MinIO, esses arquivos são armazenados como objetos dentro de **buckets**.

O banco de dados fica responsável pelos dados estruturados do produto, como:

```text
Produto
├── Nome
├── Preço
├── Estoque
└── Referência da imagem
             │
             ▼
        Bucket MinIO
             │
             ▼
      Imagem do produto
```

Dessa forma, o banco pode guardar os dados do produto e apenas uma referência para a imagem armazenada no MinIO.

## 🔄 Versionamento dos objetos

O versionamento permite manter versões anteriores de um objeto quando ele é alterado ou sobrescrito.

Por exemplo:

```text
produto.jpg
│
├── Versão 1
├── Versão 2
└── Versão 3
```

Caso uma imagem seja substituída por engano, é possível recuperar uma versão anterior.

Isso ajuda a proteger os arquivos contra erros humanos, como sobrescritas ou alterações acidentais.

Como as versões antigas também ocupam espaço, pode ser utilizada uma **política de ciclo de vida (Lifecycle)**.

Essa política pode mover versões antigas para classes de armazenamento mais baratas e excluí-las depois de determinado período.

```text
Versão atual
     │
     ▼
Armazenamento padrão
     │
     ▼
Versões antigas
     │
     ▼
Armazenamento frio
     │
     ▼
Arquivo
     │
     ▼
Exclusão após período definido
```

## 📝 Análise escrita

### 1. Se você levasse esta aplicação para a nuvem, quais recursos ficariam na sub-rede pública e quais na privada? Que portas você abriria no grupo de segurança do frontend?

O frontend ficaria na **sub-rede pública**, porque é a parte que os usuários precisam acessar pela internet.

Já a API e o banco de dados ficariam em **sub-redes privadas**, sem acesso direto externo.

No grupo de segurança do frontend seriam abertas:

- Porta **80** para HTTP
- Porta **443** para HTTPS

Em um ambiente de produção, o ideal seria utilizar principalmente HTTPS.

A API receberia apenas conexões dos componentes autorizados da aplicação, enquanto o banco receberia somente conexões dos serviços internos que realmente precisassem acessá-lo.

### 2. Pelo modelo de responsabilidade compartilhada, manter o banco fechado ao mundo é responsabilidade do provedor ou do cliente?

Essa configuração é responsabilidade do **cliente**.

O provedor cuida da segurança da infraestrutura física e dos serviços oferecidos, enquanto o cliente é responsável por configurar corretamente sua aplicação.

Isso inclui:

- Configuração da rede
- Permissões
- Grupos de segurança
- Controle de acesso
- Definição dos recursos expostos à internet

Portanto, deixar o banco em uma rede privada e sem uma porta aberta para a internet é responsabilidade de quem configura a aplicação.

### 3. Na Tarefa 5, a vitrine continuou no ar e somente `/api/` caiu. O que poderia evitar que a queda de uma única instância da API derrubasse a rota?

Seria possível adicionar um **Load Balancer** e executar mais de uma instância da API ao mesmo tempo.

```text
                    ┌──► API 1
Frontend ──► LB ────┤
                    ├──► API 2
                    │
                    └──► API 3
```

O balanceador distribuiria as requisições entre as instâncias disponíveis.

Se uma delas parasse de funcionar, as próximas requisições poderiam ser encaminhadas para outra instância saudável, mantendo a rota `/api/` disponível.

### 4. Cite duas vantagens de ter isolado a API e o banco em vez de publicá-los diretamente.

A primeira vantagem é a **segurança**, pois o isolamento reduz a superfície de ataque.

Como a API e o banco não ficam diretamente expostos, usuários externos não conseguem acessá-los da mesma forma que acessam o frontend.

A segunda vantagem é seguir o **princípio do menor privilégio**, fazendo com que cada camada se comunique apenas com aquilo que realmente precisa.

```text
Usuário
   │
   ▼
Frontend
   │
   ▼
API
   │
   ▼
Banco
```

Não existe necessidade de deixar todos os componentes diretamente acessíveis pela internet.

### 5. Por que o volume do banco corresponde a um armazenamento de bloco, e o que aconteceria com os dados sem ele?

O volume do banco corresponde a um armazenamento semelhante ao **armazenamento de bloco**, porque funciona como um espaço persistente ligado ao serviço do banco e mantém os dados separados do ciclo de vida do contêiner.

Sem esse volume, ao remover o contêiner, os dados armazenados somente nele poderiam ser perdidos.

Com o volume, é possível destruir e recriar o contêiner sem perder as informações que já haviam sido gravadas.

### 6. Por que as imagens dos produtos vão para o armazenamento de objetos (MinIO) e não para o banco de dados?

As imagens são arquivos não estruturados e se encaixam melhor no **armazenamento de objetos**, representado neste projeto pelo MinIO.

Nesse modelo, os arquivos ficam armazenados como objetos dentro de buckets.

O banco fica responsável pelos dados estruturados do produto, como:

- Nome
- Preço
- Estoque

O banco pode armazenar apenas uma referência para a imagem existente no MinIO.

### 7. Como o versionamento no bucket protege contra um erro humano, e que política de ciclo de vida pode ser utilizada?

O versionamento mantém versões anteriores de um objeto quando ele é sobrescrito.

Assim, se uma imagem for substituída por engano, é possível recuperar uma versão anterior.

Para evitar que versões antigas permaneçam ocupando espaço e gerando custos indefinidamente, pode ser utilizada uma **política de ciclo de vida**.

Essa política pode:

1. Manter versões recentes no armazenamento padrão
2. Mover versões antigas para armazenamento mais barato
3. Mover versões ainda mais antigas para armazenamento de arquivo
4. Excluir automaticamente versões que não são mais necessárias

## 📁 Estrutura do projeto

A estrutura principal do projeto é:

```text
aplicacao-tres-camadas-docker/
│
├── web/
│   ├── html/
│   │   └── index.html
│   │
│   └── nginx.conf
│
├── docker-compose.yml
└── README.md
```

Com a implementação do CI/CD, também será adicionada a estrutura:

```text
.github/
└── workflows/
    └── ci.yml
```

## ⚙️ CI/CD

A próxima etapa do projeto será a implementação de uma pipeline de **CI/CD utilizando GitHub Actions**.

A ideia é automatizar etapas do projeto sempre que novas alterações forem enviadas ao repositório.

A configuração será adicionada posteriormente conforme o desenvolvimento da atividade.

## 🚧 Status do projeto

**Projeto em desenvolvimento.**

### Implementado

- Arquitetura de três camadas
- Docker Compose
- Frontend com Nginx
- Proxy reverso
- API interna
- Redis
- Isolamento de rede
- Persistência de dados
- MinIO
- Versionamento de objetos

### Próxima etapa

- CI/CD com GitHub Actions

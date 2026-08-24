# 🐳 Aplicação de Três Camadas com Docker

Projeto desenvolvido na disciplina de **Computação em Nuvem**, com o objetivo de aplicar na prática conceitos de arquitetura em três camadas utilizando Docker.

## 👥 Alunos

- **Jonas Gabriel Amorim Ribeiro**
- **Lucas Leal Ferreira**

---

## 📚 Sobre o projeto

A aplicação utiliza uma arquitetura de três camadas, separando frontend, API e banco de dados em serviços distintos.

A proposta é simular, utilizando Docker, uma arquitetura semelhante à encontrada em ambientes de nuvem, mantendo apenas o frontend acessível externamente e isolando os demais serviços em uma rede interna.

### Tecnologias utilizadas

- Docker
- Docker Compose
- Nginx
- Traefik Whoami
- Redis
- MinIO
- HTML

---

## 🏗️ Arquitetura

A aplicação foi organizada da seguinte forma:

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

              REDE INTERNA DOCKER
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        ┌─────────┐         ┌─────────┐
        │  Redis  │         │  MinIO  │
        │  Banco  │         │ Objetos │
        └─────────┘         └─────────┘
```

O frontend é o serviço exposto ao computador do usuário. A API, o banco de dados e os demais serviços permanecem isolados na rede interna.

---

# 🧪 Desenvolvimento da atividade

## 1. Conhecendo o serviço da API

A primeira etapa consistiu em executar a imagem `traefik/whoami` como um contêiner para conhecer o funcionamento da API.

O `whoami` devolve informações do contêiner que respondeu à requisição, como o **hostname**, o **endereço IP** e alguns dados da requisição HTTP.

O hostname é útil porque identifica qual contêiner executou a resposta. Dessa forma, é possível comprovar qual instância atendeu determinada requisição.

---

## 2. Publicação da vitrine

Nesta etapa foi criado o frontend da aplicação utilizando **Nginx**.

A vitrine é a camada pública da aplicação e pode ser acessada pelo computador através de:

```text
http://localhost:8080
```

O frontend é o único serviço da arquitetura principal que precisa ficar diretamente acessível pelo usuário.

### Resultado

> 📷 Inserir aqui o print da vitrine funcionando em `localhost:8080`.

---

## 3. Publicação da API pela vitrine

A API foi mantida sem uma porta publicada diretamente para o computador.

O acesso acontece através do frontend utilizando:

```text
/api/
```

O Nginx funciona como um **proxy reverso**, recebendo a requisição do usuário e encaminhando-a para a API através da rede interna do Docker.

### Fluxo

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

A API sem porta publicada imita o funcionamento de um serviço localizado em uma **sub-rede privada**, porque ela não pode ser acessada diretamente pelo computador de fora.

Assim, somente o frontend fica exposto, enquanto a API continua protegida dentro da rede interna.

### Resultado

> 📷 Inserir aqui o print da resposta da API através de `localhost:8080/api/`.

---

## 4. Banco de dados privado

O banco de dados utilizado na aplicação é o **Redis**.

Assim como a API, o banco permanece na rede interna e não possui uma porta publicada diretamente para o computador.

Isso significa que o banco pode ser utilizado pelos serviços autorizados dentro da arquitetura sem precisar ficar exposto externamente.

### Teste de comunicação

A comunicação com o Redis pode ser verificada através do comando:

```bash
redis-cli ping
```

Quando o serviço está funcionando corretamente, a resposta esperada é:

```text
PONG
```

### Resultado

> 📷 Inserir aqui o print mostrando a resposta `PONG`.

---

## 5. Teste de isolamento de falhas por camada

Nesta etapa, a API foi interrompida propositalmente para observar o comportamento das diferentes camadas da aplicação.

Mesmo com a API indisponível, a vitrine continuou funcionando normalmente.

Por outro lado, ao tentar acessar:

```text
/api/
```

a requisição apresentou erro, pois o Nginx não conseguiu encontrar a API.

Esse comportamento demonstra o **isolamento entre as camadas**.

```text
Frontend → funcionando ✅

API → indisponível ❌

Banco → isolado 🔒
```

A falha de uma camada não necessariamente significa que todas as outras camadas precisam parar de funcionar.

### Resultado

> 📷 Inserir aqui o print mostrando a vitrine funcionando e `/api/` apresentando erro.

---

# 💾 Persistência do banco

Foi utilizado um volume para manter os dados do banco separados do ciclo de vida do contêiner.

## O que aconteceria sem o volume?

Sem o volume persistente, os dados armazenados dentro do contêiner poderiam ser perdidos quando ele fosse removido.

O volume funciona de forma semelhante a um **armazenamento de bloco**, pois fornece um espaço persistente separado da vida útil do contêiner.

Assim, mesmo que o contêiner do banco seja destruído e recriado, os dados permanecem armazenados, aumentando a durabilidade das informações.

### Fluxo simplificado

```text
Contêiner do banco
        │
        ▼
     Volume
        │
        ▼
Dados persistentes
```

---

# 🪣 Armazenamento de objetos com MinIO

As imagens dos produtos são armazenadas utilizando **MinIO**.

O MinIO representa o armazenamento de objetos da arquitetura.

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
             └── imagem-do-produto
```

## Por que a imagem vai para o armazenamento de objetos e não para o banco?

As imagens dos produtos são arquivos e, por isso, são mais adequadas ao **armazenamento de objetos**.

Esse tipo de armazenamento foi projetado para guardar arquivos como:

- imagens;
- vídeos;
- documentos;
- backups.

Os arquivos são armazenados como objetos dentro de **buckets**.

O banco de dados é mais adequado para informações estruturadas, como:

- nome;
- preço;
- estoque.

Dessa forma, o banco pode armazenar os dados do produto e uma referência para a imagem armazenada no MinIO.

---

# 🔄 Versionamento dos objetos

O versionamento permite manter versões anteriores de um objeto quando ele é alterado ou sobrescrito.

Exemplo:

```text
produto.jpg
│
├── Versão 1
├── Versão 2
└── Versão 3
```

## Como o versionamento protege contra erro humano?

O versionamento protege contra erros humanos porque mantém versões anteriores de um objeto quando ele é sobrescrito ou alterado.

Dessa forma, caso uma imagem seja substituída por engano, é possível recuperar a versão anterior.

Entretanto, as versões antigas também ocupam espaço e podem gerar custos.

Para evitar que elas permaneçam armazenadas indefinidamente, pode ser utilizada uma **política de ciclo de vida (Lifecycle)**.

Por exemplo:

```text
Versão atual
    │
    ▼
Armazenamento quente
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

Assim, versões antigas podem ser movidas para classes de armazenamento mais baratas e posteriormente excluídas quando não forem mais necessárias.

---

# 📝 Análise escrita

## 1. Se você levasse esta aplicação para a nuvem, quais recursos ficariam na sub-rede pública e quais na privada? Que portas você abriria no grupo de segurança do frontend?

O frontend ficaria na **sub-rede pública**, porque é a parte que os usuários precisam acessar pela internet.

Já a API e o banco de dados ficariam em **sub-redes privadas**, sem acesso direto externo.

No grupo de segurança do frontend seriam abertas:

- **Porta 80:** HTTP
- **Porta 443:** HTTPS

Em um ambiente de produção, o ideal seria utilizar principalmente HTTPS.

A API receberia conexões apenas dos componentes autorizados da aplicação, enquanto o banco receberia somente conexões dos serviços internos que realmente precisassem acessá-lo.

---

## 2. Pelo modelo de responsabilidade compartilhada, manter o banco fechado ao mundo é responsabilidade do provedor ou do cliente?

Essa configuração é responsabilidade do **cliente**.

O provedor é responsável pela segurança da infraestrutura física e dos serviços oferecidos, enquanto o cliente é responsável por configurar corretamente sua aplicação.

Isso inclui:

- configuração da rede;
- permissões;
- grupos de segurança;
- controle de acesso;
- definição dos recursos que serão expostos à internet.

Portanto, deixar o banco em uma rede privada e sem uma porta aberta para a internet é responsabilidade de quem configura a aplicação.

---

## 3. Na Tarefa 5, a vitrine continuou no ar e somente `/api/` caiu. O que poderia evitar que a queda de uma única instância da API derrubasse a rota?

Seria possível adicionar um **Load Balancer (balanceador de carga)** e executar mais de uma instância da API simultaneamente.

Exemplo:

```text
                    ┌──► API 1
Usuário → Nginx → LB│
                    ├──► API 2
                    │
                    └──► API 3
```

O balanceador distribuiria as requisições entre as instâncias disponíveis.

Se uma delas parasse de funcionar, as próximas requisições poderiam ser encaminhadas para outra instância saudável, mantendo a rota `/api/` disponível.

---

## 4. Cite duas vantagens de isolar a API e o banco em vez de publicá-los diretamente.

### Segurança

O isolamento reduz a **superfície de ataque**.

Como a API e o banco não ficam diretamente expostos, usuários externos não conseguem acessá-los da mesma forma que acessam o frontend.

### Princípio do menor privilégio

Cada camada se comunica apenas com aquilo que realmente precisa.

Por exemplo:

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

Não existe necessidade de deixar todos os componentes acessíveis diretamente pela internet.

---

## 5. Por que o volume do banco corresponde a um armazenamento de bloco, e o que aconteceria com os dados sem ele?

O volume do banco corresponde a um armazenamento semelhante ao **armazenamento de bloco**, porque funciona como um espaço persistente ligado ao serviço do banco e mantém os dados separados do ciclo de vida do contêiner.

Sem esse volume, ao remover o contêiner, os dados armazenados somente nele poderiam ser perdidos.

Com o volume, é possível destruir e recriar o contêiner sem perder as informações que já haviam sido gravadas.

---

## 6. Por que as imagens dos produtos vão para o armazenamento de objetos (MinIO) e não para o banco de dados?

As imagens são arquivos não estruturados e se encaixam melhor no **armazenamento de objetos**, representado neste projeto pelo MinIO.

Nesse modelo, os arquivos ficam armazenados como objetos dentro de buckets, sendo adequado para:

- imagens;
- vídeos;
- documentos;
- outros arquivos.

O banco fica responsável pelos dados estruturados do produto, como nome, preço e estoque, podendo armazenar uma referência para a imagem que está no MinIO.

---

## 7. Como o versionamento protege contra erro humano e qual política pode evitar custos desnecessários com versões antigas?

O versionamento mantém versões anteriores de um objeto quando ele é sobrescrito.

Assim, se uma imagem for substituída por engano, é possível recuperar uma versão anterior.

Para evitar que versões antigas permaneçam ocupando espaço e gerando custos indefinidamente, pode ser utilizada uma **política de ciclo de vida**.

Essa política pode:

1. manter versões recentes no armazenamento padrão;
2. mover versões antigas para armazenamento frio;
3. mover versões ainda mais antigas para armazenamento de arquivo;
4. excluir automaticamente versões que não são mais necessárias.

---

# 📁 Estrutura do projeto

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

---

# 🔐 Resumo do isolamento

| Serviço | Acesso externo | Função |
|---|---|---|
| Frontend / Nginx | ✅ Sim | Interface pública e proxy reverso |
| API / Whoami | ❌ Não | Serviço interno da aplicação |
| Redis | ❌ Não | Banco de dados |
| MinIO | 🔒 Controlado | Armazenamento de objetos |

A arquitetura busca manter exposto apenas aquilo que realmente precisa receber acesso externo, enquanto os demais componentes permanecem isolados.

---

## 🎯 Conceitos aplicados

Durante o desenvolvimento foram trabalhados conceitos de:

- Arquitetura de três camadas
- Contêineres
- Docker Compose
- Redes Docker
- Isolamento de serviços
- Proxy reverso
- Persistência de dados
- Armazenamento de bloco
- Armazenamento de objetos
- Buckets
- Versionamento
- Políticas de ciclo de vida
- Responsabilidade compartilhada
- Sub-redes públicas e privadas
- Grupos de segurança
- Balanceamento de carga
- Alta disponibilidade

---

## 🚧 Status do projeto

Projeto desenvolvido para atividade prática da disciplina de **Computação em Nuvem**.

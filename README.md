# Aplicação de Três Camadas com Docker

Projeto desenvolvido na disciplina de **Computação em Nuvem** com o objetivo de aplicar na prática conceitos de arquitetura em três camadas utilizando Docker.

## 👥 Alunos

- Jonas Gabriel
- Lucas Leal

## 📚 Sobre o projeto

A aplicação será composta por três serviços:

- **Frontend:** Nginx
- **API:** Traefik Whoami
- **Banco de dados:** Redis

A arquitetura utiliza uma rede interna do Docker para manter a API e o banco de dados isolados, permitindo acesso externo apenas ao frontend.

## 🐳 Tecnologias utilizadas

- Docker
- Docker Compose
- Nginx
- Traefik Whoami
- Redis
- HTML

## 📁 Estrutura do projeto

```text
aplicacao-tres-camadas-docker/
│
├── web/
│   ├── html/
│   │   └── index.html
│   └── nginx.conf
│
├── docker-compose.yml
└── README.md

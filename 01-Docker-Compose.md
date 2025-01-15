# Docker Compose

## Sumário
1. [O que é Docker Compose](#o-que-é-docker-compose)
2. [Estrutura e Sintaxe](#estrutura-e-sintaxe)
3. [Componentes Principais](#componentes-principais)
4. [Casos de Uso](#casos-de-uso)
5. [Comandos Essenciais](#comandos-essenciais)
6. [Boas Práticas](#boas-práticas)
7. [Exemplos Práticos](#exemplos-práticos)
8. [Configurações Avançadas](#configurações-avançadas)

## O que é Docker Compose

Docker Compose é uma ferramenta que permite definir e executar aplicações Docker multi-container. Com ele, você usa um arquivo YAML para configurar os serviços da sua aplicação e, com um único comando, cria e inicia todos os serviços a partir da sua configuração.

### Principais Benefícios

1. **Simplificação da Configuração**
   - Define toda a stack em um único arquivo
   - Elimina comandos Docker longos e complexos
   - Mantém consistência entre ambientes

2. **Gerenciamento de Dependências**
   - Controla a ordem de inicialização dos serviços
   - Gerencia conexões entre containers
   - Compartilha recursos entre serviços

3. **Portabilidade**
   - Mesmo ambiente em diferentes máquinas
   - Fácil compartilhamento de configurações
   - Versionamento de infraestrutura

## Estrutura e Sintaxe

### Anatomia de um docker-compose.yml

```yaml
version: '3.8'                 # Versão do formato do arquivo

services:                      # Definição dos serviços
  api:                        # Nome do serviço
    build: .                  # Configuração de build
    ports:                    # Mapeamento de portas
      - "3000:3000"
    environment:              # Variáveis de ambiente
      - NODE_ENV=development
    volumes:                  # Volumes
      - .:/app
    depends_on:              # Dependências
      - db

  db:                        # Outro serviço
    image: postgres:13       # Imagem base
    environment:
      - POSTGRES_PASSWORD=senha

volumes:                     # Volumes nomeados
  pgdata:

networks:                    # Redes personalizadas
  backend:
```

### Principais Elementos

1. **version**
   - Define a versão do formato do arquivo
   - Influencia recursos disponíveis
   - Compatibilidade com versão do Docker Engine

2. **services**
   - Containers que compõem a aplicação
   - Configurações específicas de cada serviço
   - Relacionamentos entre serviços

3. **volumes**
   - Armazenamento persistente
   - Compartilhamento de dados
   - Backup e restauração

4. **networks**
   - Comunicação entre containers
   - Isolamento de rede
   - Configurações de rede personalizadas

## Componentes Principais

### Services

```yaml
services:
  web:
    build:
      context: ./dir
      dockerfile: Dockerfile.dev
      args:
        - ENV=development
    image: myapp:latest
    container_name: web-app
    ports:
      - "80:80"
    environment:
      - API_KEY=secret
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - db
      - redis
    restart: unless-stopped
    networks:
      - frontend
      - backend
```

### Volumes

```yaml
volumes:
  # Volume nomeado
  dbdata:
    driver: local
    
  # Volume com configurações específicas
  cached:
    driver: local
    driver_opts:
      type: none
      device: /path/on/host
      o: bind

services:
  db:
    volumes:
      # Volume nomeado
      - dbdata:/var/lib/postgresql/data
      # Bind mount
      - ./init:/docker-entrypoint-initdb.d
      # Volume anônimo
      - /var/tmp
```

### Networks

```yaml
networks:
  frontend:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
  
  backend:
    internal: true
    driver: overlay
    
services:
  api:
    networks:
      backend:
        aliases:
          - api.local
      frontend:
        ipv4_address: 172.28.1.1
```

## Casos de Uso

### 1. Ambiente de Desenvolvimento

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    command: npm run dev
    ports:
      - "3000:3000"
      - "9229:9229"  # debugging
    environment:
      - NODE_ENV=development
    
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=dev
      - POSTGRES_PASSWORD=dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 2. Ambiente de Teste

```yaml
version: '3.8'

services:
  test:
    build:
      context: .
      target: test
    volumes:
      - .:/app
    command: npm test
    environment:
      - NODE_ENV=test
      - DB_HOST=test-db
    depends_on:
      - test-db

  test-db:
    image: postgres:13
    environment:
      - POSTGRES_USER=test
      - POSTGRES_PASSWORD=test
```

### 3. Ambiente de Produção

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: production
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
```

## Comandos Essenciais

### Básicos

```bash
# Iniciar serviços
docker-compose up -d

# Parar serviços
docker-compose down

# Visualizar logs
docker-compose logs -f [serviço]

# Listar serviços em execução
docker-compose ps

# Executar comando em um serviço
docker-compose exec serviço comando
```

### Build e Deploy

```bash
# Construir imagens
docker-compose build

# Reconstruir serviços
docker-compose up -d --build

# Pull de imagens atualizadas
docker-compose pull

# Push de imagens construídas
docker-compose push
```

### Escala e Gestão

```bash
# Escalar serviço
docker-compose up -d --scale web=3

# Reiniciar serviços
docker-compose restart

# Pausar serviços
docker-compose pause

# Retomar serviços
docker-compose unpause
```

## Boas Práticas

### 1. Organização de Arquivos

```plaintext
projeto/
├── docker-compose.yml           # Configuração base
├── docker-compose.override.yml  # Desenvolvimento
├── docker-compose.prod.yml      # Produção
├── .env                        # Variáveis de ambiente
├── Dockerfile                  # Build da aplicação
└── config/
    ├── nginx/                  # Configurações Nginx
    └── redis/                  # Configurações Redis
```

### 2. Variáveis de Ambiente

```yaml
# .env
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
REDIS_URL=redis://redis:6379

# docker-compose.yml
services:
  db:
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
```

### 3. Multi-stage Builds

```dockerfile
# Dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM node:18-slim
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/main"]
```

### 4. Healthchecks

```yaml
services:
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## Exemplos Práticos

### 1. Stack MERN (MongoDB, Express, React, Node)

```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      target: development
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
    environment:
      - REACT_APP_API_URL=http://localhost:4000

  backend:
    build:
      context: ./backend
      target: development
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
    depends_on:
      - mongo
    environment:
      - MONGODB_URI=mongodb://mongo:27017/myapp

  mongo:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

### 2. Microserviços com Message Broker

```yaml
version: '3.8'

services:
  auth-service:
    build: ./auth
    environment:
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - rabbitmq

  payment-service:
    build: ./payments
    environment:
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - rabbitmq

  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
```

## Configurações Avançadas

### 1. Redes Customizadas

```yaml
networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  backend:
    internal: true
    driver: overlay

services:
  api:
    networks:
      - frontend
      - backend
```

### 2. Secrets Management

```yaml
secrets:
  db_password:
    file: ./secrets/db_password.txt
  ssl_cert:
    external: true

services:
  api:
    secrets:
      - db_password
      - ssl_cert
```

### 3. Deploy Configs

```yaml
services:
  api:
    deploy:
      mode: replicated
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
```

### 4. Logging

```yaml
services:
  api:
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

Sempre importante consultar a [documentação oficial do Docker Compose](https://docs.docker.com/compose/).

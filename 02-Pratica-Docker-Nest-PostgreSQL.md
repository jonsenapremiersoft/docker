# Exercício Prático de Docker

## Introdução
Este exercício irá guiá-lo na criação de uma aplicação web usando Docker. Vamos construir uma API simples com NestJS e PostgreSQL.

## Por que usar Docker?

- **Consistência**: Sua aplicação funcionará igual em qualquer lugar
- **Isolamento**: Cada parte da aplicação fica separada e organizada
- **Facilidade**: Não precisa instalar banco de dados ou outras ferramentas localmente

## Pré-requisitos

### 1. Docker
- O Docker é o container engine que vamos usar
- Baixe em: https://www.docker.com/products/docker-desktop
- Após instalar, abra o terminal e execute:
  ```bash
  docker --version
  ```
  Se aparecer a versão, está tudo certo!

### 2. Docker Compose
- Ferramenta para gerenciar múltiplos containers
- Geralmente já vem com o Docker Desktop
- Verifique com:
  ```bash
  docker-compose --version
  ```

### 3. Node.js
- Necessário para criar o projeto inicial
- Baixe em: https://nodejs.org/
- Versão recomendada: 18.x ou superior

### 4. Editor de Código
- Recomendamos VS Code: https://code.visualstudio.com/
- Instale as extensões:
  - Docker
  - Remote Containers
  - TypeScript

## Passo 1: Criando o Projeto Base

### Por que NestJS?
NestJS é um framework Node.js moderno que facilita a criação de APIs organizadas. Vamos usá-lo porque:
- Tem boa integração com TypeScript
- Estrutura bem organizada
- Fácil de adicionar banco de dados

### Criando o Projeto

```bash
# Instalar o CLI do NestJS globalmente
npm install -g @nestjs/cli

# Criar novo projeto
nest new docker-exercise
# Selecione 'npm' quando perguntado sobre o gerenciador de pacotes

# Entrar na pasta do projeto
cd docker-exercise

# Instalar dependências para banco de dados
npm install @nestjs/typeorm typeorm pg
```

**O que cada dependência faz?**
- `@nestjs/typeorm`: Integração do NestJS com banco de dados
- `typeorm`: ORM para trabalhar com banco de dados
- `pg`: Driver do PostgreSQL

## Passo 2: Criando o Dockerfile

### O que é um Dockerfile?
Um Dockerfile é como uma receita que diz ao Docker como construir sua aplicação. Cada linha é uma instrução que cria uma camada na imagem.

```dockerfile
# Usar Node.js 18 com Alpine Linux (uma versão leve)
FROM node:18-alpine

# Definir a pasta de trabalho dentro do container
WORKDIR /app

# Copiar arquivos de dependências
# Copiamos primeiro para aproveitar o cache do Docker
COPY package*.json ./

# Instalar dependências
RUN npm install

# Copiar todo o código
COPY . .

# Dizer qual porta o container vai usar
EXPOSE 3000

# Comando para iniciar a aplicação
CMD ["npm", "run", "start:dev"]
```

**Explicação de cada instrução:**
- `FROM`: Imagem base que vamos usar
- `WORKDIR`: Pasta onde vamos trabalhar no container
- `COPY`: Copia arquivos do seu computador para o container
- `RUN`: Executa comandos durante a construção da imagem
- `EXPOSE`: Indica qual porta será usada
- `CMD`: Comando que será executado quando o container iniciar

## Passo 3: Configurando o Docker Compose

### Por que usar Docker Compose?
O Docker Compose ajuda a gerenciar múltiplos containers. No nosso caso, precisamos de:
- Container para a API (NestJS)
- Container para o banco de dados (PostgreSQL)

```yaml
version: '3.8'

services:
  # Nossa API
  api:
    build: .  # Usa o Dockerfile na pasta atual
    ports:
      - "3000:3000"  # Mapeia porta 3000 do container para seu computador
    volumes:
      - .:/app  # Sincroniza arquivos entre seu computador e o container
      - /app/node_modules  # Volume especial para node_modules
    environment:  # Variáveis de ambiente
      - DATABASE_HOST=db  # Nome do serviço do banco de dados
      - DATABASE_PORT=5432
      - DATABASE_USER=postgres
      - DATABASE_PASSWORD=postgres
      - DATABASE_NAME=docker_exercise
    depends_on:  # Indica que precisa do banco de dados
      - db

  # Banco de Dados
  db:
    image: postgres:13  # Usa imagem oficial do PostgreSQL
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=docker_exercise
    volumes:
      - postgres_data:/var/lib/postgresql/data  # Guarda dados do banco

# Define volumes que serão criados
volumes:
  postgres_data:
```

**Explicação detalhada:**
1. **services**: Define os containers que vamos usar
   - `api`: Nossa aplicação NestJS
   - `db`: Banco de dados PostgreSQL

2. **volumes**: 
   - Mantém os dados mesmo se o container for removido
   - `.:/app`: Sincroniza seu código com o container
   - `postgres_data`: Guarda dados do PostgreSQL

3. **environment**:
   - Variáveis de ambiente necessárias
   - Configurações do banco de dados

## Passo 4: Configurando a Conexão com o Banco

### Atualizando o Módulo Principal
Arquivo `src/app.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    // Configuração do TypeORM
    TypeOrmModule.forRoot({
      type: 'postgres',  // Tipo do banco de dados
      host: process.env.DATABASE_HOST,  // Pega valor da variável de ambiente
      port: parseInt(process.env.DATABASE_PORT),
      username: process.env.DATABASE_USER,
      password: process.env.DATABASE_PASSWORD,
      database: process.env.DATABASE_NAME,
      autoLoadEntities: true,  // Carrega entidades automaticamente
      synchronize: true,  // Cria tabelas automaticamente (não use em produção!)
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**O que cada configuração faz?**
- `type`: Indica que usamos PostgreSQL
- `host`: Endereço do banco de dados
- `port`: Porta do banco de dados
- `autoLoadEntities`: Carrega modelos automaticamente
- `synchronize`: Cria tabelas baseadas nos modelos

## Passo 5: Criando um Modelo de Dados

### O que é uma Entidade?
Uma entidade representa uma tabela no banco de dados. Vamos criar uma entidade simples para tarefas.

Arquivo `src/task/task.entity.ts`:
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()  // Marca classe como uma entidade do banco
export class Task {
  @PrimaryGeneratedColumn()  // Cria ID automático
  id: number;

  @Column()  // Campo normal da tabela
  title: string;

  @Column({ default: false })  // Campo com valor padrão
  completed: boolean;
}
```

## Passo 6: Rodando a Aplicação

### Comandos Principais
```bash
# Construir e iniciar containers
docker-compose up -d

# Ver status dos containers
docker-compose ps

# Ver logs da aplicação
docker-compose logs -f api
```

**O que cada comando faz?**
- `up -d`: Inicia os containers em segundo plano
- `ps`: Mostra status dos containers
- `logs -f`: Mostra logs em tempo real

### Como saber se deu certo?
1. Execute `docker-compose ps`
   - Deve mostrar dois containers rodando
2. Acesse `http://localhost:3000`
   - Deve ver a página inicial do NestJS
3. Verifique os logs com `docker-compose logs -f api`
   - Não deve mostrar erros

## Problemas Comuns e Soluções

### 1. "A porta 3000 já está em uso"
```bash
# Verifique o que está usando a porta
lsof -i :3000  # Linux/Mac
netstat -ano | findstr :3000  # Windows

# Pare o processo ou mude a porta no docker-compose.yml
```

### 2. "Não consegue conectar ao banco de dados"
```bash
# Verifique se o container do banco está rodando
docker-compose ps

# Veja logs do banco
docker-compose logs db

# Verifique se as variáveis de ambiente estão corretas
docker-compose config
```

### 3. "Mudanças no código não aparecem"
```bash
# Reinicie o container da API
docker-compose restart api

# Ou reconstrua a imagem
docker-compose up -d --build api
```

## Dicas Importantes

1. **Sempre verifique os logs**
   - Os logs mostram o que está acontecendo
   - Use `docker-compose logs -f` frequentemente

2. **Use o VS Code**
   - A extensão Docker ajuda muito
   - Você pode ver logs, containers e imagens na interface

3. **Mantenha o terminal aberto**
   - É mais fácil ver problemas em tempo real
   - Você aprende mais vendo os logs

## Como Testar se Está Funcionando

1. **Teste o NestJS**
   ```bash
   # Deve mostrar "Hello World"
   curl http://localhost:3000
   ```

2. **Teste o Banco de Dados**
   ```bash
   # Entrar no container do banco
   docker-compose exec db psql -U postgres -d docker_exercise
   
   # Dentro do psql, liste as tabelas
   \dt
   ```

3. **Teste os Volumes**
   ```bash
   # Faça uma mudança no código
   # Veja se a mudança aparece sem reconstruir
   ```

## Próximos Passos Sugeridos

1. **Adicione mais funcionalidades**
   - Crie rotas para adicionar tarefas
   - Implemente sistema de usuários
   - Adicione validação de dados

2. **Melhore a configuração**
   - Separe ambientes (dev/prod)
   - Adicione variáveis de ambiente
   - Configure backups do banco


## Recursos

1. **Documentação Oficial**
   - [Docker](https://docs.docker.com/)
   - [NestJS](https://docs.nestjs.com/)
   - [TypeORM](https://typeorm.io/)

# Docker

## Introdução ao Docker

### Por que usar Docker?

O Docker resolve vários problemas comuns no desenvolvimento de software:

1. **"Funciona na minha máquina"**
   - Elimina diferenças entre ambientes
   - Garante consistência entre desenvolvimento, teste e produção
   - Facilita a colaboração entre equipes

2. **Isolamento de Recursos**
   - Cada aplicação roda em seu próprio container
   - Evita conflitos de dependências
   - Melhor gestão de recursos

3. **Escalabilidade**
   - Facilita a replicação de ambientes
   - Permite escalar horizontalmente
   - Simplifica o processo de deploy

### Como o Docker Funciona

O Docker utiliza recursos do kernel Linux para criar ambientes isolados (containers):

- **Namespaces**: Isolam processos, rede, sistemas de arquivos
- **Control Groups (cgroups)**: Controlam recursos (CPU, memória, I/O)
- **Union File System**: Sistema de arquivos em camadas

## Instalação e Configuração

### Windows

1. **Requisitos**
   - Windows 10/11 Pro, Enterprise ou Education
   - Virtualização habilitada no BIOS
   - WSL2 instalado

```powershell
# Instalar WSL2
wsl --install

# Reiniciar o computador
```

2. **Instalação Docker Desktop**
   - Baixe o instalador em [Docker Desktop](https://www.docker.com/products/docker-desktop)
   - Execute o instalador
   - Marque "Use WSL2 instead of Hyper-V"
   - Finalize a instalação e reinicie

3. **Verificação**
```powershell
docker --version
docker-compose --version
docker run hello-world
```

### Linux (Ubuntu/Debian)

1. **Remover versões antigas**
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

2. **Instalar dependências**
```bash
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

3. **Adicionar repositório oficial**
```bash
# Adicionar chave GPG
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Adicionar repositório
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4. **Instalar Docker Engine**
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

5. **Configurar usuário**
```bash
# Adicionar usuário ao grupo docker
sudo usermod -aG docker $USER

# Aplicar alterações (fazer logout e login)
```

6. **Verificação**
```bash
docker --version
docker run hello-world
```

## Conceitos Fundamentais

### 1. Containers

Um container é uma unidade padrão de software que empacota código e todas as dependências para que a aplicação seja executada de forma rápida e confiável em diferentes ambientes.

**Características principais:**
- Leve e portátil
- Isolado do sistema host
- Compartilha o kernel do host
- Início rápido
- Recursos controlados

**Anatomia de um container:**
```plaintext
Container
├── Aplicação
├── Dependências
├── Bibliotecas
├── Binários
└── Configurações
```

### 2. Imagens

Uma imagem é um template somente leitura com instruções para criar um container.

**Características:**
- Imutável
- Construída em camadas
- Pode ser baseada em outras imagens
- Compartilhável via registries (como Docker Hub)

### 3. Volumes

Volumes são o mecanismo preferido para persistir dados gerados e usados por containers Docker.

**Tipos de volumes:**
1. **Named Volumes**
   ```bash
   # Criar volume
   docker volume create meu-volume
   
   # Usar volume
   docker run -v meu-volume:/data nginx
   ```

2. **Bind Mounts**
   ```bash
   # Montar diretório local
   docker run -v $(pwd):/app nginx
   ```

3. **tmpfs Mounts** (apenas Linux)
   ```bash
   # Dados em memória
   docker run --tmpfs /tmp nginx
   ```

## Comandos Docker Essenciais

### Gestão de Containers

```bash
# Listar containers em execução
docker ps
docker container ls

# Listar todos os containers
docker ps -a
docker container ls -a

# Criar e iniciar container
docker run [opções] imagem [comando]
  -d          # Modo detached (background)
  -p 80:80    # Mapear porta
  -v vol:/dir # Montar volume
  --name web  # Nome do container
  --rm        # Remover após parar
  -e VAR=val  # Variável de ambiente

# Exemplo prático
docker run -d \
  --name minha-api \
  -p 3000:3000 \
  -v $(pwd):/app \
  -e NODE_ENV=development \
  node:18

# Iniciar container existente
docker start container-id

# Parar container
docker stop container-id

# Reiniciar container
docker restart container-id

# Remover container
docker rm container-id
docker rm -f container-id  # Força remoção

# Logs do container
docker logs container-id
docker logs -f container-id  # Follow logs

# Executar comando em container
docker exec -it container-id bash
```

### Gestão de Imagens

```bash
# Listar imagens
docker images
docker image ls

# Baixar imagem
docker pull node:18

# Construir imagem
docker build -t app:1.0 .
docker build --no-cache -t app:1.0 .

# Remover imagem
docker rmi imagem-id
docker image rm imagem-id

# Inspecionar imagem
docker image inspect imagem-id

# Histórico de camadas
docker history imagem-id
```

### Network

```bash
# Listar redes
docker network ls

# Criar rede
docker network create minha-rede

# Conectar container à rede
docker network connect minha-rede container-id

# Inspecionar rede
docker network inspect minha-rede
```


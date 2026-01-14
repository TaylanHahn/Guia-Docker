## <img src="https://www.docker.com/wp-content/uploads/2022/03/Moby-logo.png" width="45" /> DOCKER 

### 1. Conceitos Fundamentais 🐳

Para um desenvolvedor **Java**, a melhor forma de entender **Docker** é através da **Programação Orientada a Objetos (POO)** 🧠:

- 📄 **Dockerfile ≈ Código Fonte (`.java`)**  
  É a *receita* 🧾. Onde você define tudo o que a aplicação precisa para funcionar.

- 📦 **Imagem ≈ Classe (`.class`)**  
  É o binário **imutável** 🔒 gerado a partir do Dockerfile.  
  👉 Você não “roda” uma imagem, você a **instancia**.

- ▶️ **Container ≈ Objeto (Instância)**  
  É a imagem em execução 🚀.  
  Você pode ter vários containers (objetos) rodando a partir da mesma imagem (classe), todos **isolados entre si** 🧱.

- 🌐 **Registry ≈ Maven Repository / Nexus**  
  Local onde as imagens são armazenadas 📚 (ex: **Docker Hub**).

---

### 🛠️ 2. O Dockerfile: Criando a Imagem Java Perfeita ☕🐳 

A prática moderna exige o uso de **Multi-Stage Builds** 🧩.  
Isso evita que o código fonte e as ferramentas de build (**Maven/Gradle**) fiquem na imagem final de produção, reduzindo o tamanho de **800MB+ ➜ ~150MB** 📉.

### 🎯 Cenário  
Uma aplicação **Spring Boot** 🌱 simples.


**🧪 Exemplo de Dockerfile (Multi-Stage)**

```dockerfile
# --- 🏗️ Estágio 1: Build (Compilação) ---
# Usamos uma imagem com Maven para gerar o .jar
FROM maven:3.9.6-eclipse-temurin-17 AS build
WORKDIR /app

# 📄 Copia apenas o pom.xml primeiro (otimização de cache de camadas)
COPY pom.xml .
# ⬇️ Baixa as dependências (se o pom não mudou, o Docker reutiliza essa camada)
RUN mvn dependency:go-offline

# 📂 Copia o código fonte e faz o build
COPY src ./src
RUN mvn clean package -DskipTests

# --- 🚀 Estágio 2: Runtime (Execução) ---
# Usamos uma imagem JRE leve (apenas o necessário para rodar)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# 📦 Copia apenas o JAR gerado no estágio anterior
COPY --from=build /app/target/*.jar app.jar

# 🌐 Define a porta que a aplicação expõe
EXPOSE 8080

# ▶️ Comando para iniciar a aplicação
# 💡 Dica Java: Otimização de memória para containers
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
````

> 💎🔥 **Dica de Ouro**
>  
> O uso de:
>  
> ```text
> -XX:MaxRAMPercentage=75.0
> ```
>  
> informa à JVM ☕ para usar **75% da RAM alocada ao container** 🐳,  
> e **não** da máquina host 🖥️.
>  
> ✅ Isso previne o erro **`OOMKilled (Out of Memory)`** ❌💥,  
> muito comum em ambientes Docker.

---

### 🧩 3. Comandos Essenciais (CLI) 🖥️🐳
Aqui estão os comandos que você usará **90% do tempo** ⏱️.

**🔁 Ciclo de Vida**

- **1. 🏗️ Construir a imagem:** `docker build -t meu-java-app:v1` . *(O ponto final indica que o Dockerfile está na pasta atual)*

- **2. 🚀 Rodar o container:** `docker run -d -p 8080:8080 --name app-java --memory="512m" meu-java-app:v1`
  - `-d`: Detached mode (roda em background) 🕶️.
  - `-p`: Mapeia porta (PortaHost:PortaContainer) 🌐.
  - `--memory`: Limita a RAM do container 🧠🐳.

**🛠️ Gerenciamento e Debug 🐞**

- **3. 📜 Ver logs (System.out.println):** `docker logs -f app-java`
  - `-f`: Follow (acompanha em tempo real) 📡.

- **4. 🧪 Acessar o terminal do container:** `docker exec -it app-java sh` *(Útil para verificar se arquivos de configuração foram copiados corretamente 📂).*

- **5. 🧹 Listar e Limpar:*
  - `docker ps`: Lista containers rodando 📋.
  - `docker system prune -a`: Limpa containers parados e imagens não utilizadas *(economiza espaço em disco 💾✨)*.

---

## 🧩 4. Orquestração Local: Docker Compose 🐳🧩
No mundo real 🌍, sua aplicação **Java** ☕ precisa de um **Banco de Dados** 🗄️.  
O **Docker Compose** permite subir múltiplos containers 🐳🐳 definindo-os em um arquivo **YAML** 📄.

### 🎯 Cenário
Aplicação **Spring Boot** 🌱 conectando ao **PostgreSQL** 🐘.

Arquivo: `docker-compose.yml` 📦
````yaml
version: '3.8'

services:
  # Serviço da Aplicação Java
  app:
    build: . # Constrói a imagem localmente usando o Dockerfile
    ports:
      - "8080:8080"
    environment:
      # Conecta usando o NOME do serviço do banco (db) como host
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/meubanco
      - SPRING_DATASOURCE_USERNAME=user
      - SPRING_DATASOURCE_PASSWORD=password
    depends_on:
      - db # Espera o container do banco iniciar primeiro
    networks:
      - java-network

  # Serviço do Banco de Dados
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=meubanco
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres-data:/var/lib/postgresql/data # Persistência
    networks:
      - java-network

# Definição de Volumes (Persistência)
volumes:
  postgres-data:

# Definição de Redes (Isolamento)
networks:
  java-network:
````
> **⚙️ Comandos do Compose**
> 
> ▶️ Subir tudo: `docker-compose up -d`
> 
> ⛔ Derrubar tudo: `docker-compose down`
> 
> 🔁 Rebuildar (após mudar código Java ☕): `docker-compose up -d --build`

---

## 🧩 5. Persistência de Dados (Volumes) 💾🐳
Containers são **efêmeros** ⏳.  
Se você deletar o container do **PostgreSQL** 🐘 sem um volume, **perderá os dados** ❌📉.

- **📂 Bind Mount:** Mapeia uma pasta do seu computador 🖥️ para o container 🐳.  
  Útil para **desenvolvimento** *(ex: código fonte ou configs)*.
  - `./configs:/app/config`

- **📦 Volume Gerenciado (Recomendado para DB):** O Docker gerencia automaticamente a área de armazenamento 🔒.
  - No exemplo acima: `postgres-data:/var/lib/postgresql/data`.  
    Mesmo que você destrua o container `db`, o volume `postgres-data` **permanece** ✅🛡️.

---

## 🧩 6. Networking (Redes) 🌐🐳
No **Docker Compose**, os serviços se comunicam pelo **nome do serviço** 📛.

- Se sua aplicação **Java** ☕ precisa chamar o banco de dados 🗄️, o host **não** é `localhost` ❌.
- O host correto é `db` ✅ *(nome definido no `docker-compose.yml` 📄)*.
- O Docker possui um **DNS interno** 🌐 que resolve `db` para o **IP interno** do container do *PostgreSQL* 🐘.

---

## 🧩 7. Boas Práticas para Java ☕🐳

- **1. 📄 Arquivo `.dockerignore`:**  
  Crie este arquivo na raiz *(igual ao `.gitignore`)*.  
  Adicione `target/`, `.git/`, `.idea/`.  
  Isso evita copiar **lixo** para dentro da imagem 🗑️, acelerando o build ⚡.

- **2. 📦 Imagens "Distroless" ou "Alpine":**  
  Prefira imagens base **`alpine`** *(ex: `eclipse-temurin:17-jre-alpine`)* por serem **menores** 📉 e **mais seguras** 🔐 *(menos superfície de ataque)*.

- **3. 🚫 Não rode como Root:**  
  Em produção, crie um **usuário específico** dentro do Dockerfile para rodar o JAR ☕, aumentando a segurança 🛡️.

- **4. 🔁 Graceful Shutdown:**  
  O **Spring Boot** 🌱 intercepta o sinal `SIGTERM` do Docker 🐳 para desligar suavemente.  
  Certifique-se de que seu `ENTRYPOINT` permite passar esses sinais:
  - ✅ Formato array: `["java", ...]` *(permite sinais)*  
  - ❌ Formato string: `java ...` *(não permite)*


---



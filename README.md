# 1. Conceitos Fundamentais 🐳

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

# 🛠️ 2. O Dockerfile: Criando a Imagem Java Perfeita ☕🐳  

A prática moderna exige o uso de **Multi-Stage Builds** 🧩.  
Isso evita que o código fonte e as ferramentas de build (**Maven/Gradle**) fiquem na imagem final de produção, reduzindo o tamanho de **800MB+ ➜ ~150MB** 📉.

### 🎯 Cenário  
Uma aplicação **Spring Boot** 🌱 simples.

---

## 🧪 Exemplo de Dockerfile (Multi-Stage)

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



### 📝 Wiki Completa: Spring Boot + MySQL Docker Setup

### Indice
1. [Introduzione](#introduzione)
2. [Prerequisiti](#prerequisiti)
3. [Struttura del progetto](#struttura-del-progetto)
4. [Creazione del database MySQL con Docker](#creazione-del-database-mysql-con-docker)
5. [Creazione del backend Spring Boot](#creazione-del-backend-spring-boot)
6. [Dockerizzazione del backend](#dockerizzazione-del-backend)
7. [Docker Compose per ambiente completo](#docker-compose-per-ambiente-completo)
8. [Problemi comuni e soluzioni](#problemi-comuni-e-soluzioni)
9. [Test dell’ambiente](#test-dellambiente)
10. [Best practice e configurazioni consigliate](#best-practice-e-configurazioni-consigliate)

---

### Introduzione
Questo progetto mostra come configurare un ambiente di sviluppo completo con:

- **Backend:** Spring Boot 4.0.3 + Spring Data JPA
- **Database:** MySQL 8.x
- **Containerizzazione:** Docker + Docker Compose
- **Goal:** avere backend e database pronti per sviluppo locale e test automatici.

---

### Prerequisiti
- Windows / Linux / macOS
- [Docker](https://www.docker.com/) installato
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Java 17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)
- [Maven](https://maven.apache.org/download.cgi)
- Powershell (se si usa il repo `spreengboot-prj-via-powershell`)

---

### Struttura del progetto
```text
spreengboot-prj-via-powershell/
│
├─ backend-service/
│   ├─ src/
│   ├─ pom.xml
│   └─ Dockerfile
│
├─ docker-compose.yml
└─ README.md
```

### Creazione del database MySQL con Docker

### Avvio del container MySQL

Esegui il comando seguente per creare un container MySQL per sviluppo:

```bash
docker run -d \
  --name mysql-dev \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=appdb \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=password \
  -p 3306:3306 \
  mysql:8.0

```

### Configurazione Spring Boot e Docker Compose

### Configurazione `application.properties`

```properties
spring.datasource.url=jdbc:mysql://mysql:3306/appdb
spring.datasource.username=appuser
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
spring.jpa.open-in-view=false
spring.security.user.name=admin
spring.security.user.password=admin
```

### Configurazione `docker-compose.yaml`

```properties
version: '3.9'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql-dev
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: appdb
      MYSQL_USER: appuser
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network

  backend:
    image: backend-service:latest
    container_name: backend-1
    build:
      context: ./backend-service
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/appdb
      SPRING_DATASOURCE_USERNAME: appuser
      SPRING_DATASOURCE_PASSWORD: password
    depends_on:
      - mysql
    networks:
      - app-network

volumes:
  db_data:

networks:
  app-network:
    driver: bridge
```


---


  

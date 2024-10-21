## 1 ##################################################################################
# Scarica il progetto
$uri = "https://start.spring.io/starter.zip"
$body = @{
    dependencies = "web,data-jpa,security,mysql,actuator"
    groupId = "com.example"
    artifactId = "backend-service-1"
    name = "backend-service-1"
    description = "Backend service for Spring Boot"
    packageName = "com.example.backendservice1"
    javaVersion = "17"
    springBootVersion = "3.1.0"
    type = "maven-project"
}

Invoke-WebRequest -Uri $uri -Method Post -Body $body -OutFile "backend-service-1.zip"

## 2 ##################################################################################
# Estrai il file ZIP
Expand-Archive -Path "backend-service-1.zip" -DestinationPath "backend-service-1"

## 3 ##################################################################################
# Rimuovi il file ZIP
Remove-Item -Path "backend-service-1.zip" -Force

## 4 ##################################################################################
# Crea il Dockerfile
$dockerfileContent = @"
# Usa un'immagine JDK come base
FROM openjdk:17-jdk-slim
# Imposta la cartella di lavoro
WORKDIR /app
# Copia il file JAR dell'applicazione nella cartella di lavoro
COPY target/backend-service-1-0.0.1-SNAPSHOT.jar app.jar
# Comando per eseguire l'applicazione
ENTRYPOINT ["java", "-jar", "app.jar"]
"@

# Scrivi il Dockerfile nella cartella del progetto
$dockerfilePath = "backend-service-1/Dockerfile"
$dockerfileContent | Out-File -FilePath $dockerfilePath -Encoding utf8

Write-Host "Progetto creato in: backend-service-1"
Write-Host "Dockerfile creato in: $dockerfilePath"

FRONT-END
ng new front-end-app
ng g s service/api


start docker 
docker-compose up --build 

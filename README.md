# gestor-frontend

Frontend web del Gestor de Proyectos (login, registro, proyectos, tareas y comentarios). Consume la API REST [gestor-api](https://github.com/Pedro-Uriel-Perez/gestor-api).

| | Versión |
|---|---|
| Java | **21** |
| Spring Boot | **3.5.16** (Thymeleaf, Spring Web) |
| Maven | 3.9 o superior (incluye `mvnw`) |
| Pruebas de navegador | Playwright 1.63 (Chromium) |
| Puerto | 8080 por defecto (si está ocupado, ver abajo) |

## Requisitos
- JDK 21 (`JAVA_HOME` apuntando a él). El build falla con un mensaje claro si se usa otra versión (maven-enforcer).
- La API `gestor-api` corriendo en http://localhost:8081 (configurable con `api.url`).

## Ejecutar
```
mvnw spring-boot:run
mvnw spring-boot:run "-Dspring-boot.run.arguments=--server.port=8082"     (si el 8080 está ocupado)
```

## Pruebas y cobertura
```
mvnw test          → 26 pruebas: JUnit + MockRestServiceServer + Playwright (requiere la API levantada)
mvnw -Pci test     → sin las pruebas de navegador (lo que ejecuta Jenkins)
```
Reporte de cobertura: `target/site/jacoco/index.html`.

## Jenkins
El pipeline está en `Jenkinsfile` y usa las herramientas **jdk-21** y **maven-3.9** de Jenkins.
El Jenkins en Docker con esa configuración está en el repositorio gestor-api (carpeta `jenkins/`); el job se llama **gestor-frontend**.

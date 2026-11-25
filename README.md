# gestor-frontend

[![CI/CD gestor-frontend](https://github.com/Pedro-Uriel-Perez/gestor-frontend/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/Pedro-Uriel-Perez/gestor-frontend/actions/workflows/ci-cd.yml)
[![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=Pedro-Uriel-Perez_gestor-frontend&metric=alert_status)](https://sonarcloud.io/project/overview?id=Pedro-Uriel-Perez_gestor-frontend)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=Pedro-Uriel-Perez_gestor-frontend&metric=coverage)](https://sonarcloud.io/component_measures?id=Pedro-Uriel-Perez_gestor-frontend&metric=coverage)

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
- La API `gestor-api` corriendo en http://localhost:8081 (configurable con `api.url` o `API_URL`).

## Ejecutar
```
mvnw spring-boot:run
mvnw spring-boot:run "-Dspring-boot.run.arguments=--server.port=8082"     (si el 8080 está ocupado)
```

## Pruebas y cobertura
```
mvnw test          → 29 pruebas: JUnit + MockRestServiceServer + Playwright (requiere la API levantada)
mvnw -Pci test     → sin las pruebas de navegador (lo que ejecutan GitHub Actions y Jenkins)
```
Reporte de cobertura: `target/site/jacoco/index.html`.

## CI/CD en la nube (GitHub Actions + SonarQube Cloud)
[`ci-cd.yml`](.github/workflows/ci-cd.yml): compila y prueba (JUnit + JaCoCo) → SonarQube Cloud con Quality Gate
→ Trivy (dependencias, secretos y Dockerfile, sin `target/`) → Semgrep (SAST) → despliega en Render el mismo
commit y comprueba `/actuator/health`. Las acciones están fijadas por SHA y los secretos son del repositorio.

## Despliegue y observabilidad
- `Dockerfile`: JRE 21, usuario sin privilegios, CDS y el agente de OpenTelemetry (apagado por defecto); en
  Render lo enciende el grupo de variables `otel-grafana-cloud` creado con Terraform
  ([gestor-infra](https://github.com/Pedro-Uriel-Perez/gestor-infra)). El agente propaga el contexto de traza a
  la API, así cada petición se ve completa en Grafana Cloud: frontend → API → PostgreSQL.

## Jenkins local (alternativa)
El pipeline local está en `Jenkinsfile`; el Jenkins en Docker está en el repositorio gestor-api (carpeta `jenkins/`)
y el job se llama **gestor-frontend**.

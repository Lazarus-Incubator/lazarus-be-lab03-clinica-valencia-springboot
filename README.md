# tpl-be-springboot

Plantilla base para laboratorios de Backend con **Spring Boot 4.0.3** y **Java 25**.

Incluye:
- **Swagger UI** para explorar y probar la API desde el navegador
- Endpoint **Ping** para verificar que el servicio está vivo

---

## Requisitos
- **Java 25** instalado y configurado (JDK)
- Git
- No necesitas Maven instalado: el proyecto usa **Maven Wrapper** (`mvnw.cmd`)

> Si `.\mvnw.cmd -v` muestra Java 17 u otra versión, tu sistema/IDE no está usando Java 25.

---

## Ejecutar el proyecto (Windows)
En la raíz del proyecto:

```powershell
.\mvnw.cmd spring-boot:run
```

El servicio levanta por defecto en:
- http://localhost:8080

---

## Probar rápido

### Ping (sanity check)
- Endpoint: **GET** http://localhost:8080/api/v1/ping
- Respuesta esperada:

```json
{ "status": "ok" }
```

---

## Swagger / OpenAPI

### Swagger UI (interfaz web)
Abre en el navegador:
- http://localhost:8080/swagger-ui/index.html

### OpenAPI JSON
- http://localhost:8080/v3/api-docs

---

## Convenciones de trabajo
- No trabajes directo en `main`. Crea una rama:
  - `feat/<algo-corto>` (ej. `feat/tipo-cambio-get`)
- Commits pequeños y claros.
- Entrega por Pull Request (PR).

---

## Checklist antes de entregar
- [ ] El proyecto arranca con `.\mvnw.cmd spring-boot:run`
- [ ] Swagger UI abre: http://localhost:8080/swagger-ui/index.html
- [ ] Ping responde: **GET** http://localhost:8080/api/v1/ping

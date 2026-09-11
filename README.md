# Escuela Colombiana de Ingeniería Julio Garavito
## Arquitectura de Software – ARSW
### Laboratorio – Parte 2: BluePrints API con Seguridad JWT (OAuth 2.0)

Este laboratorio extiende la **Parte 1** ([Lab_P1_BluePrints_Java21_API](https://github.com/DECSIS-ECI/Lab_P1_BluePrints_Java21_API)) agregando **seguridad a la API** usando **Spring Boot 3, Java 21 y JWT (OAuth 2.0)**.  
El API se convierte en un **Resource Server** protegido por tokens Bearer firmados con **RS256**.  
Incluye un endpoint didáctico `/auth/login` que emite el token para facilitar las pruebas.

---

## Objetivos
- Implementar seguridad en servicios REST usando **OAuth2 Resource Server**.
- Configurar emisión y validación de **JWT**.
- Proteger endpoints con **roles y scopes** (`blueprints.read`, `blueprints.write`).
- Integrar la documentación de seguridad en **Swagger/OpenAPI**.

---

## Requisitos
- JDK 21
- Maven 3.9+
- Git

---

## Ejecución del proyecto
1. Clonar o descomprimir el proyecto:
   ```bash
   git clone https://github.com/DECSIS-ECI/Lab_P2_BluePrints_Java21_API_Security_JWT.git
   cd Lab_P2_BluePrints_Java21_API_Security_JWT
   ```
   ó si el profesor entrega el `.zip`, descomprimirlo y entrar en la carpeta.

2. Ejecutar con Maven:
   ```bash
   mvn -q -DskipTests spring-boot:run
   ```

3. Verificar que la aplicación levante en `http://localhost:8080`.

---

## Endpoints principales

### 1. Login (emite token)
```
POST http://localhost:8080/auth/login
Content-Type: application/json

{
  "username": "student",
  "password": "student123"
}
```
Respuesta:
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### 2. Consultar blueprints (requiere scope `blueprints.read`)
```
GET http://localhost:8080/api/blueprints
Authorization: Bearer <ACCESS_TOKEN>
```

### 3. Crear blueprint (requiere scope `blueprints.write`)
```
POST http://localhost:8080/api/blueprints
Authorization: Bearer <ACCESS_TOKEN>
Content-Type: application/json

{
  "name": "Nuevo Plano"
}
```

---

## Swagger UI
- URL: [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)
- Pulsa **Authorize**, ingresa el token en el formato:
  ```
  Bearer eyJhbGciOi...
  ```

---

## Estructura del proyecto
```
src/main/java/co/edu/eci/blueprints/
  ├── api/BlueprintController.java       # Endpoints protegidos
  ├── auth/AuthController.java           # Login didáctico para emitir tokens
  ├── config/OpenApiConfig.java          # Configuración Swagger + JWT
  └── security/
       ├── SecurityConfig.java
       ├── MethodSecurityConfig.java
       ├── JwtKeyProvider.java
       ├── InMemoryUserService.java
       └── RsaKeyProperties.java
src/main/resources/
  └── application.yml
```

---

## Actividades propuestas
1. Revisar el código de configuración de seguridad (`SecurityConfig`) e identificar cómo se definen los endpoints públicos y protegidos.

- Endpoints públicos: se marcan las rutas de login y las de documentación (Swagger) como accesibles sin token, porque justo para pedir el token no puedes tenerlo aún, y la documentación debe poder verse libremente.

- Endpoints protegidos por scope: todo lo que es parte de la API de negocio exige que el token traiga uno de los permisos de lectura o escritura definidos. Estos permisos no se configuran a mano en la seguridad: se generan automáticamente a partir de lo que el token dice que el usuario puede hacer (su "scope"), con un prefijo estándar que Spring agrega solo.

- Regla por defecto: cualquier ruta que no esté explícitamente clasificada igual exige estar autenticado, aunque sin pedir un permiso específico.

- Validación del token: se activa el modo de "servidor de recursos", que intercepta las peticiones, revisa el token que viene en la cabecera, lo valida contra la clave pública correspondiente y, si es válido, le asigna al usuario los permisos que traía ese token para que las reglas anteriores puedan aplicarse.

2. Explorar el flujo de login y analizar las claims del JWT emitido.

Flujo:

- El cliente envía usuario y contraseña al endpoint de login.
- Se valida esa credencial contra un servicio de usuarios en memoria (contraseñas guardadas ya hasheadas, no en texto plano).
- Si es válida, se arma el conjunto de claims del token y se firma con la clave privada RSA que la app genera al arrancar.
- Se responde con el token, su tipo (Bearer) y cuánto dura.

Claims que trae el token emitido:

- iss (issuer): identifica quién emitió el token, tomado de la configuración de la app.
- sub (subject): el usuario que inició sesión.
- iat / exp: momento de emisión y momento de expiración (el TTL también viene de configuración).
- scope: los permisos que tiene ese usuario, en este caso, tanto lectura como escritura de blueprints se le asignan de forma fija a cualquiera que haga login, sin distinguir por usuario.

3. Extender los scopes (`blueprints.read`, `blueprints.write`) para controlar otros endpoints de la API, del laboratorio P1 trabajado.
4. Modificar el tiempo de expiración del token y observar el efecto.
5. Documentar en Swagger los endpoints de autenticación y de negocio.

---

## Lecturas recomendadas
- [Spring Security Reference – OAuth2 Resource Server](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html)
- [Spring Boot – Securing Web Applications](https://spring.io/guides/gs/securing-web/)
- [JSON Web Tokens – jwt.io](https://jwt.io/introduction)

---

## Licencia
Proyecto educativo con fines académicos – Escuela Colombiana de Ingeniería Julio Garavito.

# Frontend — Aplicación Angular

Documentación orientativa del cliente web que consume el backend en **Python (FastAPI)**. El objetivo es una aplicación con **autenticación** y **gestión CRUD** de **ocho entidades**, con navegación por **sidebar** y **grillas (grids)** por recurso.

---

## 1. ¿Qué es el frontend?

El **frontend** es la parte de la aplicación que el usuario ve y usa en el navegador: pantallas, formularios, tablas, menús y mensajes. No ejecuta la lógica de negocio ni accede directamente a la base de datos; **pide datos y acciones al backend** mediante **HTTP** (por ejemplo, peticiones `GET`, `POST`, `PUT`, `DELETE`) y muestra las respuestas.

En este proyecto, el frontend será una **SPA** (Single Page Application): una sola página cargada y el resto de “pantallas” se cambia sin recargar todo el documento, usando el framework **Angular**.

---

## 2. ¿Qué es Angular?

**Angular** es un framework de **TypeScript** para construir aplicaciones web estructuradas. Ofrece:

| Concepto | Uso en este proyecto |
|----------|----------------------|
| **Componentes** | Pantallas reutilizables (login, layout con sidebar, cada grid, formularios de alta/edición). |
| **Servicios** | Llamadas HTTP al API de FastAPI, manejo de tokens de sesión. |
| **Routing** | Rutas protegidas tras el login; una ruta por entidad (o por vista CRUD). |
| **Forms** | Formularios reactivos para crear y editar registros. |
| **HttpClient** | Cliente HTTP para consumir el backend REST. |

La documentación oficial está en [angular.dev](https://angular.dev).

---

## 3. Cómo se consume el backend (FastAPI)

El backend expone una **API REST** (típicamente JSON). El frontend **no** importa código Python; solo envía peticiones a URLs base del servidor, por ejemplo:

- `https://api.ejemplo.com` o `http://localhost:8000` (según entorno).

### Flujo habitual

1. **Login**: el usuario envía credenciales (`POST /auth/login` o ruta equivalente que defina el backend).
2. **Respuesta**: el backend devuelve un **token** (JWT u otro esquema) y datos del usuario.
3. **Peticiones siguientes**: el frontend adjunta el token en el header `Authorization: Bearer <token>`.
4. **CRUD por entidad**: para cada recurso, el frontend usa los verbos HTTP acordados con el API:

| Acción | HTTP | Ejemplo de ruta (convención REST) |
|--------|------|-----------------------------------|
| Listar / leer muchos | `GET` | `/entidades` |
| Leer uno | `GET` | `/entidades/{id}` |
| Crear | `POST` | `/entidades` |
| Actualizar | `PUT` o `PATCH` | `/entidades/{id}` |
| Eliminar | `DELETE` | `/entidades/{id}` |

Las rutas exactas deben coincidir con las que exponga tu proyecto FastAPI (OpenAPI/Swagger suele estar en `/docs`).

### En Angular

- Se centraliza la **URL base** del API (por ejemplo en `environment.ts`).
- Un **interceptor HTTP** puede inyectar el token y manejar errores 401 (cerrar sesión o redirigir al login).
- Los **servicios** por dominio (por entidad) encapsulan `HttpClient` y devuelven `Observable` o se adaptan a señales según la versión de Angular.

---

## 4. Alcance funcional previsto

- **Login**: pantalla de inicio de sesión; tras éxito, acceso al área principal.
- **Ocho entidades**: cada una con operaciones **CRUD** completas.
- **Sidebar**: menú lateral con una entrada por entidad (y opcionalmente inicio/cierre de sesión).
- **Por entidad**: una vista tipo **grid** (tabla con paginación/filtros según se defina) con acciones listar, crear, editar y eliminar.

La lista concreta de las 8 entidades debe documentarse cuando esté definida en el backend (nombres de modelos y endpoints).

---

## 5. Estructura de carpetas propuesta (Angular)

Estructura orientativa, alineada con buenas prácticas y fácil de escalar a 8 módulos de entidad:

```
src/
├── app/
│   ├── core/                    # Singletons: auth, interceptors, guards
│   │   ├── auth/
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.guard.ts
│   │   │   └── login/
│   │   └── interceptors/
│   │       └── auth.interceptor.ts
│   ├── shared/                  # Componentes y pipes reutilizables
│   │   ├── components/
│   │   └── models/              # Interfaces TypeScript alineadas al API
│   ├── layout/                  # Shell: sidebar + router-outlet
│   │   └── main-layout/
│   ├── features/                # Una carpeta por entidad + login
│   │   ├── login/
│   │   ├── entidad-1/
│   │   ├── entidad-2/
│   │   └── ...                  # hasta entidad-8
│   ├── app.routes.ts
│   ├── app.config.ts
│   └── app.component.ts
├── environments/
│   ├── environment.ts           # apiUrl, etc.
│   └── environment.development.ts
└── main.ts
```

- **`core/`**: servicios que deben existir una sola vez (autenticación, interceptor).
- **`shared/`**: tablas genéricas, diálogos de confirmación, modelos compartidos.
- **`layout/`**: marco visual con **sidebar** y área donde se cargan las rutas hijas.
- **`features/<entidad>/`**: componente de **grid**, diálogo o página de formulario, y `*.service.ts` que llama al FastAPI para esa entidad.

---

## 6. Rutas sugeridas

| Ruta | Descripción |
|------|-------------|
| `/login` | Formulario de acceso (pública). |
| `` | Área autenticada con layout + sidebar. |
| `/entidad-1`, `/entidad-2`, … | Vista grid + CRUD de cada entidad (nombres según negocio). |

El **guard** de autenticación protege las rutas hijas del layout y redirige a `/login` si no hay sesión válida.

---

## 7. Pasos para arrancar el proyecto (cuando se cree el código)

1. Instalar [Node.js](https://nodejs.org/) (LTS recomendado).
2. Instalar Angular CLI: `npm install -g @angular/cli`.
3. Crear el proyecto: `ng new nombre-proyecto` (opciones: routing sí, estilos a elección).
4. Configurar `environment*.ts` con la **URL base** del API FastAPI.
5. Implementar **login** y **servicio de auth** según el contrato del backend.
6. Añadir **HttpClient** y el **interceptor** de token en `app.config.ts`.
7. Crear el **layout** con **sidebar** y rutas lazy-loaded por feature (`loadChildren`).
8. Por cada una de las **8 entidades**: servicio HTTP + pantalla grid + formularios/modales CRUD.
9. Probar contra el backend en local o desplegado; revisar CORS en FastAPI si el front y el API están en orígenes distintos.

---

## 8. Notas de integración con FastAPI

- **CORS**: el backend debe permitir el origen del frontend (puerto de `ng serve` o dominio de producción).
- **Esquema de datos**: los DTOs del API deben reflejarse en **interfaces TypeScript** en `shared/models` o por feature.
- **Documentación**: usar `/docs` (Swagger) del FastAPI como contrato de referencia para URLs, cuerpos y códigos de respuesta.

---

## 9. Próximos pasos

- Sustituir `entidad-1` … `entidad-8` por los **nombres reales** de dominio.
- Añadir al README la **URL del repositorio del backend** y variables de entorno necesarias cuando existan.

---

*README generado como guía del frontend Angular para el curso; actualizar conforme evolucione el API y el código.*

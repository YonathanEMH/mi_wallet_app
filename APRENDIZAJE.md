# Mi Wallet App - Proyecto de Aprendizaje

## Meta
Dejar de depender de IA para tomar decisiones de arquitectura web.

## Visión del Proyecto
Cada usuario tiene un perfil con sesión, registra sus gastos personales, y ve paneles con insights:
- Gastos más altos
- Categorías que más generan gastos
- Recomendaciones de qué podría disminuir

---

## Stack Tecnológico
| Capa | Tecnología | Notas |
|------|------------|-------|
| Backend | FastAPI + SQLAlchemy + Pydantic | API REST |
| Frontend | Next.js + TypeScript + Zod | App web |
| Base de Datos | PostgreSQL | Base de datos relacional |
| Autenticación | JWT (JSON Web Token) | Tokens de acceso |
| Hashing | bcrypt | Para passwords |
| Control Versiones | Git + GitFlow | Registro en GitHub |
| Deploy | Gratuito (Railway/Vercel) | Aprender a desplegar |

## Estructura del Proyecto
```
mi_wallet_app/
├── backend/       # FastAPI (Python)
├── frontend/      # Next.js (React)
└── APRENDIZAJE.md # Este documento
```

---

## Decisiones Arquitectónicas y su Por Qué

### 1. ¿Por qué la Foreign Key va en la tabla Gasto y no en otra?
- Razón: Un gasto es la tabla "hija" en la relación. Es el evento que ocurre y necesita referencias a quién lo hizo y de qué tipo es.
- Alternativa considerada: Poner FK en Usuario apuntando a Gastos (no tiene sentido - un usuario no "apunta" a gastos, los gastos apuntan al usuario)

### 2. ¿Por qué el monto es DECIMAL(10,2) y no FLOAT?
- Razón: Para manejar dinero con precisión (0.01 pesos). Float puede tener errores de redondeo en operaciones aritméticas.

### 3. ¿Por qué password_hash con bcrypt?
- Razón: Si la base de datos se filtra, los passwords reales NO quedan expuestos. bcrypt está diseñado para ser lento y resistente a ataques de fuerza bruta.
- Si guardáramos passwords en texto plano y alguien hackea la DB, tendría todos los passwords reales de los usuarios.

### 4. ¿Por qué JWT en vez de sesiones tradicionales?
- Razón: El token se guarda del lado del cliente (localStorage o cookies httpOnly). Cada request lo envía y el servidor lo verifica con una clave secreta. Es escalable y no requiere almacenar sesiones en el servidor.
- Alternativa considerada: Cookies de sesión (el servidor guarda el estado). Más simple pero menos escalable.

### 5. ¿Por qué PostgreSQL y no SQLite?
- Razón: PostgreSQL es una base de datos relacional robusta, gratuita, y apta para producción. SQLite es mejor para testing o apps muy simples.
- Aprender PostgreSQL es más valioso para el mercado laboral.

---

## Modelo de Datos (Completado)

### Tablas Definidas

#### Usuario
| Campo | Tipo | Notas |
|-------|------|-------|
| id | SERIAL PRIMARY KEY | Identificador único |
| username | VARCHAR(50) | Unique, not null |
| email | VARCHAR(100) | Unique, not null |
| password_hash | VARCHAR(255) | bcrypt, nunca guardar plaintext |
| created_at | TIMESTAMP | Default: now() |

#### Categoria
| Campo | Tipo | Notas |
|-------|------|-------|
| id | SERIAL PRIMARY KEY | Identificador único |
| nombre | VARCHAR(50) | Unique, not null |
| created_at | TIMESTAMP | Default: now() |

**Categorías predefinidas:** Comida, Transporte, Ocio

#### Gasto
| Campo | Tipo | Notas |
|-------|------|-------|
| id | SERIAL PRIMARY KEY | Identificador único |
| usuario_id | INTEGER FK → Usuario | Relación 1:N con Usuario |
| categoria_id | INTEGER FK → Categoria | Relación 1:N con Categoria |
| monto | DECIMAL(10,2) | Positive, not null |
| descripcion | VARCHAR(200) | Nullable |
| fecha | DATE | Not null |
| created_at | TIMESTAMP | Default: now() |

### Diagrama de Relaciones
```
┌─────────────┐       ┌─────────────┐
│  Usuario    │       │  Categoria  │
└──────┬──────┘       └──────┬──────┘
       │                     │
       │    ┌────────────┐   │
       └───►│   Gasto    │◄──┘
            │            │
            │ usuario_id │
            │ categoria_id│
            │ monto      │
            │ fecha      │
            └────────────┘
```

---

## Fases del Proyecto

### Fase 1: Modelo de Datos ✅
- [x] Definir 3 tablas con campos y relaciones
- [x] Entender decisiones arquitectónicas

### Fase 2: Git + Control de Versiones
- [ ] Inicializar repositorio Git
- [ ] Crear .gitignore
- [ ] Primer commit
- [ ] Flujo GitFlow: main, develop, feature branches

### Fase 3: Backend - Autenticación
- [ ] Setup proyecto FastAPI
- [ ] Crear venv con dependencias
- [ ] Modelo Usuario con password_hash
- [ ] Endpoint POST /auth/register
- [ ] Endpoint POST /auth/login → devolver JWT
- [ ] Configurar JWT con python-jose
- [ ] Access token de 24 horas

### Fase 4: Backend - CRUD Gastos
- [ ] Modelo Gasto con FK a Usuario
- [ ] Endpoint GET /gastos (solo gastos del usuario autenticado)
- [ ] Endpoint POST /gastos
- [ ] Endpoint PUT /gastos/{id} (validar propiedad del gasto)
- [ ] Endpoint DELETE /gastos/{id} (validar propiedad del gasto)
- [ ] Validaciones: monto > 0, fecha válida

### Fase 5: Backend - Categorías y Resumen
- [ ] Endpoint GET /categorias
- [ ] Endpoint GET /gastos/resumen-mensual?mes=XX&anio=YYYY
- [ ] Lógica: agrupar por categoría, devolver totales

### Fase 6: Frontend - Autenticación
- [ ] Setup Next.js + TypeScript
- [ ] Formulario registro
- [ ] Formulario login
- [ ] Guardar JWT en localStorage
- [ ] Proteger rutas privadas

### Fase 7: Frontend - Gastos
- [ ] Listar gastos del usuario
- [ ] Formulario crear gasto
- [ ] Formulario editar gasto
- [ ] Validación con Zod antes de enviar

### Fase 8: Frontend - Dashboard
- [ ] Resumen mensual por categoría
- [ ] Mostrar gráfico o tabla de totales

### Fase 9: Deploy
- [ ] Deploy backend en Railway
- [ ] Deploy frontend en Vercel
- [ ] Configurar variables de entorno

---

## Reglas de Negocio
- Un usuario solo puede ver/modificar/eliminar SUS propios gastos
- Intento de modificar gasto de otro usuario → 403 Forbidden
- Validación de monto positivo
- Si no hay gastos en el mes: devolver diccionario vacío, no error

---

## Registro de Sesiones

| Fecha | Fase | Decisiones tomadas |
|-------|------|-------------------|
| 2026-05-07 | 1 | Se definieron las 3 tablas con sus campos y relaciones |
| 2026-05-10 | Plan | Se define scope MVP con auth JWT, PostgreSQL, deploy gratuito, Git |
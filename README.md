# 🗄️ Learning Inventory — Fase 6

Aplicación fullstack de gestión de inventario construida con PostgreSQL (Neon), Next.js y React/Vite. Desplegada en Vercel.

## 🌐 Demo en producción

- **Frontend**: https://learning-inventory-frontend.vercel.app/
- **Backend API**: https://learning-inventory-backend.vercel.app/api/products

---

## 🏗️ Arquitectura

React/Vite (Frontend) → Next.js API Routes (Backend) → PostgreSQL en Neon

---

## 📦 Tecnologías

| Capa | Tecnología |
|------|-----------|
| Base de datos | PostgreSQL 17 (Neon serverless) |
| Backend | Next.js 15 + TypeScript |
| Driver DB | @neondatabase/serverless |
| Frontend | React + Vite + TypeScript |
| Despliegue | Vercel |

---

## 🗃️ Esquema de base de datos

### Tabla `categories`
| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID | Clave primaria generada automáticamente |
| name | VARCHAR(100) | Nombre único de la categoría |
| description | TEXT | Descripción opcional |

### Tabla `products`
| Columna | Tipo | Descripción |
|---------|------|-------------|
| id | UUID | Clave primaria generada automáticamente |
| name | VARCHAR(150) | Nombre del producto |
| price | NUMERIC(10,2) | Precio mayor que 0 |
| stock | INTEGER | Stock disponible, por defecto 0 |
| category_id | UUID | Foreign key a categories |

---

## 🚀 Instalación y uso local

### Backend

```bash
cd backend
npm install
```

Crea un archivo `.env.local` con tu connection string de Neon:

DATABASE_URL=postgresql://usuario:contraseña@host/neondb?sslmode=require

Arranca el servidor:

```bash
npm run dev
```

La API estará disponible en `http://localhost:3000/api/products`

### Frontend

```bash
cd frontend
npm install
npm run dev
```

El frontend estará en `http://localhost:5173`

---

## 📡 API Endpoints

### GET `/api/products`
Devuelve todos los productos con su categoría mediante un INNER JOIN.

**Respuesta:**
```json
[
  {
    "id": "uuid",
    "name": "Portátil Gaming",
    "price": "1299.99",
    "stock": 9,
    "category": "Electrónica"
  }
]
```

### POST `/api/products`
Inserta un nuevo producto usando consultas parametrizadas.

**Body:**
```json
{
  "name": "Nuevo producto",
  "price": 99.99,
  "stock": 10,
  "category_id": "uuid-de-categoria"
}
```

---

## 🔒 Seguridad

Todas las consultas usan **parámetros preparados** para prevenir inyección SQL:

```typescript
// ✅ Seguro - parámetros separados del SQL
const result = await sql`
  INSERT INTO products (name, price, stock, category_id)
  VALUES (${name}, ${price}, ${stock}, ${category_id})
`;

// ❌ Vulnerable - concatenación directa
const query = "SELECT * FROM products WHERE id = " + userId;
```

---

## 📁 Estructura del proyecto
learning-inventory/
├── sql/
│   ├── schema.sql        # Definición de tablas
│   └── seed.sql          # Datos de prueba
└── docs/
├── arquitectura-datos.md
├── analisis-sql.md
└── seguridad-db.md
backend/
├── src/
│   ├── app/
│   │   └── api/
│   │       └── products/
│   │           └── route.ts
│   └── lib/
│       └── db.ts
└── .env.local
frontend/
└── src/
└── App.tsx

---

## 🧩 ORM: Drizzle ORM

En proyectos grandes se usan ORMs para abstraer las consultas SQL. Drizzle ORM permite definir el esquema en TypeScript y hacer consultas con tipado completo.

### Ventajas frente a SQL puro
- El esquema se define en TypeScript con autocompletado y validación
- Las consultas son type-safe, el compilador detecta errores antes de ejecutar
- Migraciones automáticas al cambiar el esquema
- Más mantenible en proyectos grandes con muchas tablas

### Ejemplo con Drizzle

```typescript
// Definición del esquema
const products = pgTable('products', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: varchar('name', { length: 150 }).notNull(),
  price: numeric('price').notNull(),
});

// Consulta tipada
const result = await db.select().from(products);
```
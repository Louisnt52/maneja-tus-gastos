# 💰 Maneja tus Gastos — Monorepo

Proyecto de ejemplo para enseñar **Docker**, **GitHub Actions** y **CI/CD** con un stack moderno.

## 🏗️ Arquitectura

```
maneja-tus-gastos/
├── frontend/        → Angular 19 + Firebase Hosting
├── backend/         → Node.js + Express + PostgreSQL (Sequelize)
├── docker-compose.yml
└── .github/workflows/
    ├── deploy-frontend.yml   → CI/CD → Firebase Hosting
    └── deploy-backend.yml    → CI/CD → Render.com
```

## 🚀 Levantar en local (con Docker)

### Pre-requisitos
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Git](https://git-scm.com/)

### 1. Clonar el repositorio
```bash
git clone https://github.com/TU_USUARIO/maneja-tus-gastos.git
cd maneja-tus-gastos
```

### 2. Copiar variables de entorno del backend
```bash
cp backend/.env.example backend/.env
```

### 3. Levantar todo con Docker Compose
```bash
docker-compose up --build
```

| Servicio   | URL                        |
|------------|----------------------------|
| Frontend   | http://localhost:4200       |
| Backend API| http://localhost:3000       |
| Health     | http://localhost:3000/health|

### 4. Bajar los contenedores
```bash
docker-compose down
```

---

## 🧪 Correr Tests

### Backend (Jest)
```bash
cd backend
npm install
npm test
```

### Frontend (Karma/Jasmine)
```bash
cd frontend
npm install
npm test
```

---

## 📡 API REST

Base URL: `http://localhost:3000/api`

| Método | Endpoint          | Descripción            |
|--------|-------------------|------------------------|
| GET    | /expenses         | Listar todos los gastos|
| GET    | /expenses/:id     | Obtener gasto por ID   |
| POST   | /expenses         | Crear nuevo gasto      |
| PUT    | /expenses/:id     | Actualizar gasto       |
| DELETE | /expenses/:id     | Eliminar gasto         |

### Ejemplo body (POST/PUT):
```json
{
  "description": "Supermercado",
  "amount": 50.00,
  "category": "Alimentación",
  "date": "2026-03-08"
}
```

### Categorías válidas:
`Alimentación` · `Transporte` · `Entretenimiento` · `Salud` · `Educación` · `Hogar` · `Otros`

---

## ⚙️ GitHub Actions Secrets requeridos

Ve a tu repo → **Settings → Secrets and variables → Actions** y agrega:

| Secret | Descripción |
|--------|-------------|
| `FIREBASE_SERVICE_ACCOUNT` | JSON de cuenta de servicio de Firebase |
| `FIREBASE_PROJECT_ID` | ID del proyecto Firebase |
| `RENDER_DEPLOY_HOOK_URL` | URL del deploy hook de Render.com |

---

## 🔥 Deploy Frontend → Firebase Hosting

Ver guía completa al final de este README: [Paso a Paso Firebase](#-paso-a-paso-firebase-hosting)

## 🟢 Deploy Backend → Render.com

Ver guía completa al final de este README: [Paso a Paso Render](#-paso-a-paso-rendercom)

---

## 🔥 Paso a Paso: Firebase Hosting

### 1. Crear proyecto en Firebase
1. Ve a [console.firebase.google.com](https://console.firebase.google.com)
2. Click **"Agregar proyecto"**
3. Ingresa nombre: `maneja-tus-gastos`
4. Desactiva Google Analytics (no es necesario)
5. Click **"Crear proyecto"**

### 2. Instalar Firebase CLI
```bash
npm install -g firebase-tools
```

### 3. Login en Firebase
```bash
firebase login
```

### 4. Inicializar Hosting en el proyecto
```bash
cd frontend
firebase init hosting
```
Responde las preguntas así:
- **Project**: selecciona `maneja-tus-gastos` (el que creaste)
- **Public directory**: `dist/frontend/browser`
- **Single-page app**: `Yes`
- **GitHub automatic deploys**: `No` (lo hacemos manual con Actions)
- **Overwrite index.html**: `No`

Esto actualiza `.firebaserc` con tu project ID real.

### 5. Generar Service Account para GitHub Actions
1. En Firebase Console → ⚙️ **Configuración del proyecto** → **Cuentas de servicio**
2. Click **"Generar nueva clave privada"** → descarga el JSON
3. En GitHub → tu repo → **Settings → Secrets → New repository secret**
4. Nombre: `FIREBASE_SERVICE_ACCOUNT`, valor: pega todo el contenido del JSON
5. Nombre: `FIREBASE_PROJECT_ID`, valor: el ID de tu proyecto (ej: `maneja-tus-gastos-abc12`)

### 6. Actualizar la URL del backend en producción
Edita `frontend/src/environments/environment.prod.ts`:
```typescript
export const environment = {
  production: true,
  apiUrl: 'https://TU-SERVICIO.onrender.com', // ← tu URL de Render
};
```

### 7. Verificar el workflow
El archivo `.github/workflows/deploy-frontend.yml` se activa automáticamente al hacer push a `main` con cambios en `frontend/`.

---

## 🟢 Paso a Paso: Render.com

### 1. Crear cuenta en Render
Ve a [render.com](https://render.com) y regístrate con tu cuenta de GitHub.

### 2. Crear la base de datos PostgreSQL
1. Dashboard → **New → PostgreSQL**
2. Nombre: `gastos-db`
3. Plan: **Free**
4. Click **Create Database**
5. Copia el valor de **"External Database URL"** (lo usarás abajo)

### 3. Crear el Web Service del backend
1. Dashboard → **New → Web Service**
2. Conecta tu repositorio de GitHub
3. Configura:
   - **Name**: `maneja-tus-gastos-api`
   - **Root Directory**: `backend`
   - **Runtime**: `Docker` ← (usa tu Dockerfile automáticamente)
   - **Plan**: **Free**
4. En **Environment Variables** agrega:
   - `DATABASE_URL` → pega la URL de PostgreSQL del paso anterior
   - `NODE_ENV` → `production`
   - `PORT` → `3000`
5. Click **Create Web Service**
6. Espera que despliegue (primera vez toma ~5 min)
7. Copia la URL del servicio: `https://maneja-tus-gastos-api.onrender.com`

### 4. Obtener el Deploy Hook
1. En tu Web Service en Render → **Settings**
2. Baja hasta **"Deploy Hook"**
3. Copia la URL (algo como `https://api.render.com/deploy/srv-xxx?key=yyy`)
4. En GitHub → repo → **Settings → Secrets → New repository secret**
5. Nombre: `RENDER_DEPLOY_HOOK_URL`, valor: la URL copiada

### 5. Verificar el workflow
El archivo `.github/workflows/deploy-backend.yml` se activa al hacer push a `main` con cambios en `backend/`.

> ⚠️ **Nota para alumnos**: El free tier de Render.com "duerme" el servicio tras 15 min de inactividad. La primera request después puede tardar ~30 segundos en responder. Esto es normal en el plan gratuito.

---

## 🐳 Docker — Conceptos clave

| Concepto | Dónde se aplica |
|----------|----------------|
| **Multi-stage build** | `frontend/Dockerfile` — Stage 1: build Angular, Stage 2: nginx |
| **Alpine images** | Ambos Dockerfiles usan `node:20-alpine` y `nginx:alpine` para imágenes livianas |
| **Health checks** | `docker-compose.yml` verifica que PostgreSQL esté listo antes de iniciar el backend |
| **Volumes** | `postgres_data` persiste los datos entre reinicios del contenedor |
| **Depends on** | El backend espera que la DB esté healthy antes de arrancar |

---

## 📚 Stack Tecnológico

| Capa | Tecnología |
|------|------------|
| Frontend | Angular 19 (Standalone Components) |
| Backend | Node.js 20 + Express 4 |
| Base de datos | PostgreSQL 16 + Sequelize ORM |
| Contenedores | Docker + Docker Compose |
| CI/CD Frontend | GitHub Actions → Firebase Hosting |
| CI/CD Backend | GitHub Actions → Render.com |
| Tests Frontend | Karma + Jasmine |
| Tests Backend | Jest + Supertest |


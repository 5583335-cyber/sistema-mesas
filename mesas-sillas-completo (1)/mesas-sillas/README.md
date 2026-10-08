# Sistema de Gestión de Mesas y Sillas

Node.js + Express + PostgreSQL (Neon) + frontend HTML/CSS/JS. Una sola app: Express sirve `frontend/`.

## Estructura
```
database/schema.sql
backend/ (package.json, .env.example, src/{config,controllers,middleware,routes,services,utils,app.js})
frontend/ (html, css/, js/)
```

## 1. Crear la base en Neon
1. Entra a https://neon.tech y crea un proyecto.
2. En **Connection Details** copia la cadena (`postgresql://...?sslmode=require`).

## 2. Ejecutar schema.sql
En Neon: **SQL Editor** → pega todo `database/schema.sql` → **Run**.
(El script borra y recrea las tablas; incluye datos de prueba.)

## 2b. (Opcional) Datos para las gráficas
Ejecuta `database/seed_demo.sql` en Neon para tener historial de 7 días.

## 3. Configurar .env
```
cd backend
cp .env.example .env
```
Edita `.env`:
```
DATABASE_URL=<cadena de Neon>
JWT_SECRET=<texto largo aleatorio>
PORT=10000
NODE_ENV=production
```

## 4. Ejecutar local
```
cd backend
npm install
npm start
```
Abre http://localhost:10000

## 5. GitHub
```
git init
git add .
git commit -m "Sistema de mesas y sillas"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
git push -u origin main
```
`.env` está en `.gitignore`; no se sube.

## 6. Desplegar en Render
1. Render → **New → Web Service** → conecta el repo.
2. **Root Directory:** `backend`
3. **Build Command:** `npm install`
4. **Start Command:** `npm start`
5. **Environment Variables:**
   - `DATABASE_URL` = cadena de Neon
   - `JWT_SECRET` = secreto largo
   - `NODE_ENV` = `production`
   - (`PORT` lo asigna Render)
6. Deploy. Health check: `/health`.

## 7. Probar la API
```
# login
curl -X POST $URL/api/auth/login -H "Content-Type: application/json" \
  -d '{"email":"admin@demo.com","password":"Admin123!"}'

# inventario / dashboard
curl $URL/api/inventario -H "Authorization: Bearer TOKEN"
curl $URL/api/dashboard  -H "Authorization: Bearer TOKEN"

# reserva confirmada (admin)
curl -X POST $URL/api/reservas -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" \
  -d '{"usuario_id":2,"fecha":"2030-01-10","hora_inicio":"08:00","hora_fin":"12:00","mesas":2,"sillas":30,"confirmar":true}'

# préstamo desde reserva
curl -X POST $URL/api/prestamos -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" \
  -d '{"reserva_id":4,"fecha_prevista_devolucion":"2030-01-12T12:00:00Z"}'

# devolución parcial
curl -X POST $URL/api/devoluciones -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" \
  -d '{"prestamo_id":4,"mesas":1,"sillas":15}'
```

## Endpoints
| Método | Ruta | Rol |
|---|---|---|
| POST | /api/auth/login, /api/auth/register | público |
| GET | /api/inventario | todos |
| POST/PUT | /api/inventario(/:id) | ADMIN |
| GET/POST | /api/reservas | todos (USER ve solo las suyas) |
| POST | /api/reservas/:id/confirmar | ADMIN |
| POST/DELETE | /api/reservas/:id/cancelar, /api/reservas/:id | dueño o ADMIN |
| GET | /api/prestamos(/:id) | todos (USER solo suyos) |
| POST/PUT | /api/prestamos(/:id) | ADMIN |
| GET | /api/devoluciones | todos (USER solo suyas) |
| POST | /api/devoluciones | ADMIN |
| GET | /api/historial | ADMIN |
| GET | /api/dashboard | todos (USER: datos propios) |
| GET/POST/PUT | /api/usuarios | ADMIN |

## Reglas de inventario
- Reserva PENDIENTE no descuenta stock. Al CONFIRMAR: disponible → reservada.
- Cancelar confirmada: reservada → disponible.
- Préstamo desde reserva: reservada → prestada (reserva pasa a FINALIZADA). Préstamo directo: disponible → prestada.
- Devolución (total o parcial): prestada → disponible.
- Todo en transacción con `SELECT … FOR UPDATE`; `CHECK` en PostgreSQL garantiza `total = disponible + reservada + prestada`.
- `movimientos` es inmutable (trigger).

## Usuarios de prueba
| Correo | Contraseña | Rol |
|---|---|---|
| admin@demo.com | Admin123! | ADMIN |
| user1@demo.com | User123! | USER |
| user2@demo.com | User123! | USER |

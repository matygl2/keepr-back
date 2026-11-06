# Keepr — Backend

API de Keepr, la app móvil para guardar tus productos y no perderte ninguna garantía. Acá se guardan las cuentas, los productos y las fotos de los documentos.

**Tecnología:** Express 5, Prisma 7, SQLite.

La app del celular está en otro repositorio: https://github.com/matygl2/keepr-front. Para usarla, este servidor tiene que estar corriendo.

---

## Requisitos

- **Node.js 20.19 o superior** (o 22.12+ / 24+). Se comprueba con `node --version`.

Todos los comandos se corren **en la raíz de este repositorio** (donde está `package.json`). Ahí se crea el archivo de la base de datos.

---

## Variables de entorno

Se leen del archivo `.env`, que **no se sube a Git**. Hay un modelo en `.env.example`.

| Variable | Qué es | Valor |
|---|---|---|
| `DATABASE_URL` | Dónde está la base SQLite | `"file:./dev.db"` |
| `JWT_SECRET` | Clave con la que se firman los tokens de sesión | Al menos **32 caracteres** |
| `PORT` | Puerto del servidor | `3000` (por defecto) |

El servidor lee el `.env` solo al arrancar: si lo cambiás, reinicialo.

---

## 1. Primera vez

```bash
git clone https://github.com/matygl2/keepr-back
cd keepr-back
npm install
```

`npm install` también genera el cliente de Prisma solo (script `postinstall`).

**Crear el archivo `.env`.** Copiá el de ejemplo:

```bash
# Windows
copy .env.example .env

# macOS / Linux
cp .env.example .env
```

Abrí `.env` y completá `JWT_SECRET` con un valor de **al menos 32 caracteres**. Para generar uno:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Las otras dos líneas se dejan como están (`DATABASE_URL="file:./dev.db"` y `PORT=3000`).

**Crear la base de datos** aplicando las migraciones que ya están en el repositorio:

```bash
npm run db:deploy
```

Eso crea el archivo `dev.db` con todas las tablas. **No usar `db:migrate`**: ese comando solo se usa cuando cambia el esquema (ver sección 3).

**Arrancar el servidor:**

```bash
npm run dev
```

Debería mostrar algo así:

```
Keepr API listening on port 3000
  Local:      http://localhost:3000
  Android emu: http://10.0.2.2:3000
  Red local:  http://192.168.1.12:3000   <- usá esta en un celular real
```

Para comprobar que anda, abrí `http://localhost:3000/health` en el navegador: tiene que responder `{"status":"ok"}`.

### Conectar la app desde un celular

La dirección que la app necesita es la de la línea **"Red local"**. Se pone en el `.env.local` de la app (ver el README del repo del frontend).

Si la PC tiene varias redes (VPN, WSL, VirtualBox), el servidor imprime una línea por cada una. La correcta es la IPv4 del adaptador **Wi-Fi**; en Windows la ves con `ipconfig`. Además, el firewall de Windows tiene que permitir a Node en redes privadas, y el celular y la PC tienen que estar en la misma red Wi-Fi.

---

## 2. Después de traer cambios (`git pull`)

Mirá qué archivos cambiaron y hacé lo que corresponda:

| Si cambió… | Hacé |
|---|---|
| `package.json` o `package-lock.json` | `npm install` |
| `prisma/schema.prisma` o hay carpetas nuevas en `prisma/migrations/` | `npm run db:deploy` → `npm run db:generate` → **reiniciar el servidor** |
| `.env.example` | Revisá si hay variables nuevas y agregalas a tu `.env` |

Si no sabés qué cambió, la rutina completa es segura y no borra datos:

```bash
npm install
npm run db:deploy
npm run db:generate
npm run dev
```

---

## 3. Si vos cambiás la base de datos

Cada vez que modifiques `prisma/schema.prisma` (una tabla, una columna):

1. Editá `schema.prisma`.
2. Creá y aplicá la migración, con un nombre corto que diga qué hace:

   ```bash
   npm run db:migrate -- --name agregar_telefono_usuario
   ```

3. Regenerá el cliente:

   ```bash
   npm run db:generate
   ```

4. Reiniciá el servidor y corré las pruebas: `npm test`.
5. **Commiteá juntos** `schema.prisma` y la carpeta nueva que apareció en `prisma/migrations/`. Sin la carpeta de la migración, nadie más puede actualizar su base.

Para ver las tablas y comprobar que el cambio se aplicó, `npm run db:studio` abre Prisma Studio en el navegador.

---

## 4. Empezar con una base de datos limpia

Si se quiere arrancar de cero (**esto borra todas las cuentas y productos de tu máquina**):

1. Frená el servidor.
2. Borrá `dev.db` (y, si querés borrar también las fotos, el contenido de `uploads/`).
3. Corré `npm run db:deploy` y volvé a arrancar con `npm run dev`.

---

## 5. Pruebas y chequeos

```bash
npm test            # usa una base temporal (test.db) que se crea y se borra sola
npm run typecheck
```

Las pruebas aplican las migraciones sobre `test.db`, así que también sirven para comprobar que tus migraciones funcionan. Si dicen que no hay migraciones, es que falta la carpeta `prisma/migrations`.

---

## 6. Scripts disponibles

| Comando | Qué hace |
|---|---|
| `npm run dev` | Arranca el servidor y lo reinicia solo al guardar cambios |
| `npm start` | Arranca el servidor sin reinicio automático |
| `npm test` | Corre las pruebas de la API |
| `npm run typecheck` | Revisa los tipos de TypeScript |
| `npm run db:deploy` | Aplica las migraciones existentes (no borra datos) |
| `npm run db:migrate -- --name <nombre>` | Crea y aplica una migración nueva (solo cuando cambia el esquema) |
| `npm run db:generate` | Regenera el cliente de Prisma |
| `npm run db:studio` | Abre Prisma Studio para ver la base |

---

## 7. Problemas frecuentes

| Qué ves | Causa y solución |
|---|---|
| `Invalid .env configuration` al arrancar | Falta el `.env` o `JWT_SECRET` tiene menos de 32 caracteres. |
| `PrismaClientValidationError: Unknown argument …` | Falta `npm run db:generate`. Corrélo y reiniciá el servidor. |
| `no such table` / `no such column` / `The column … does not exist` | La base no tiene las migraciones nuevas. Corré `npm run db:deploy`. |
| Cambiaste el `.env` y no pasa nada | El servidor lee el `.env` solo al arrancar: reinicialo. |
| `npm install` falla con un error de versión de Node | Actualizá Node a 20.19 o superior. |

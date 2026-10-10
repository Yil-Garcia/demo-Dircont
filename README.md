# 📱 Directorio de Contactos (Plantilla Educativa)

Plantilla web de nivel profesional pero con arquitectura minimalista, diseñada para que estudiantes de ingeniería practiquen el desarrollo Full-Stack, el despliegue serverless en **Vercel** y la integración con una base de datos relacional en la nube (**Azure SQL Database**).

---

## 🎯 Objetivo de Aprendizaje

1. Comprender la arquitectura cliente-servidor mediante una SPA (Single Page Application) minimalista y una API REST en Node.js.
2. Desplegar una aplicación web completa en **Vercel** usando Serverless Functions.
3. Migrar de una persistencia en memoria (Mock Array) a una base de datos empresarial (**Azure SQL**) utilizando consultas SQL nativas y buenas prácticas de seguridad (consultas parametrizadas contra inyección SQL).

---

## 📁 Estructura del Proyecto

```text
U3/
├── api/
│   └── index.js         # API REST en Node.js con endpoints CRUD y marcadores TODO
├── public/
│   └── index.html       # Frontend en un solo archivo: HTML5, CSS3 puro y JavaScript
├── .env.example         # Plantilla de variables de entorno para Azure SQL
├── .gitignore           # Exclusión de node_modules y variables sensibles
├── vercel.json          # Enrutamiento de Serverless Functions para Vercel
├── package.json         # Dependencias (express, cors, dotenv, mssql)
└── README.md            # Documentación e instrucciones
```

---

## 🚀 Inicio Rápido (Desarrollo Local)

### 1. Clonar o descargar el repositorio
Abre una terminal en la carpeta del proyecto.

### 2. Instalar dependencias
```bash
npm install
```

### 3. Iniciar el servidor local
```bash
npm start
```
Abre tu navegador en: [http://localhost:3000](http://localhost:3000)

> **Nota:** La aplicación funcionará inmediatamente utilizando datos simulados en memoria (Mock Data). Puedes agregar, editar y eliminar contactos para probar el flujo.

---

## ☁️ Despliegue en Vercel

### Opción A: Desde la interfaz web de Vercel (Recomendado con GitHub)

1. Sube este proyecto a tu repositorio de **GitHub**.
2. Entra a [vercel.com](https://vercel.com) e inicia sesión con tu cuenta de estudiante o GitHub.
3. Haz clic en **"Add New..."** > **"Project"**.
4. Importa tu repositorio `U3`.
5. En la configuración del proyecto:
   - **Framework Preset:** `Other`.
   - **Root Directory:** `./` (dejar por defecto).
6. Haz clic en **"Deploy"**. En menos de un minuto tu aplicación estará disponible en una URL pública `https://tu-proyecto.vercel.app`.

### Opción B: Usando Vercel CLI
```bash
npm i -g vercel
vercel login
vercel
```

---

## 🗄️ Guía de Práctica: Integración con Azure SQL

Una vez desplegada y probada la aplicación con los datos simulados, sigue estos pasos para conectar tu base de datos Azure SQL:

### Paso 1: Crear la Base de Datos en el Portal de Azure
1. Crea un recurso de **Azure SQL Database**.
2. Elige un nivel de servicio económico (por ejemplo, Serverless o Basic para pruebas).
3. En la pestaña de **Redes (Networking)** del servidor SQL:
   - Habilita la opción: **"Permitir que los servicios y recursos de Azure accedan a este servidor"** (*Allow Azure services and resources to access this server*). Esto es indispensable para que Vercel pueda conectarse.
   - Agrega tu IP pública actual para que puedas conectarte desde tu máquina.

### Paso 2: Crear la tabla `Contactos` (DDL)
Abre el **Editor de consultas (Query editor)** en Azure Portal o conéctate mediante Azure Data Studio / SQL Server Management Studio (SSMS) y ejecuta el siguiente script:

```sql
-- ============================================================================
-- SCRIPT DDL: CREACIÓN DE LA TABLA "Contactos"
-- ============================================================================

IF NOT EXISTS (SELECT * FROM sys.tables WHERE name = 'Contactos')
BEGIN
    CREATE TABLE Contactos (
        Id INT IDENTITY(1,1) PRIMARY KEY,
        Nombre NVARCHAR(100) NOT NULL,
        Email NVARCHAR(100) NOT NULL,
        Telefono NVARCHAR(20) NOT NULL,
        FechaCreacion DATETIME DEFAULT GETDATE()
    );
END
GO

-- Datos iniciales de prueba (Opcional):
INSERT INTO Contactos (Nombre, Email, Telefono)
VALUES 
    ('Ana García', 'ana.garcia@universidad.edu', '+52 55 1234 5678'),
    ('Carlos Mendoza', 'carlos.mendoza@universidad.edu', '+52 55 8765 4321'),
    ('Lucía Fernández', 'lucia.f@universidad.edu', '+52 55 5555 1234');
GO

-- Verificar datos insertados:
SELECT * FROM Contactos;
GO
```

---

### Paso 3: Configurar Variables de Entorno en Vercel
En el panel de tu proyecto en Vercel:
1. Dirígete a **Settings** > **Environment Variables**.
2. Añade las siguientes 4 variables:
   - `DB_USER`: Nombre del usuario administrador de Azure SQL.
   - `DB_PASSWORD`: Contraseña del servidor SQL.
   - `DB_SERVER`: Dirección del servidor (ej. `mi-servidor.database.windows.net`).
   - `DB_NAME`: Nombre de la base de datos (ej. `DirectorioDB` o la que hayas asignado).

Para pruebas locales, crea un archivo `.env` en la raíz copiando `.env.example`.

---

### Paso 4: Completar los marcadores en `api/index.js`
Abre el archivo [`api/index.js`](api/index.js) y busca los bloques señalados con:
```javascript
// // TODO: [PRÁCTICA AZURE SQL]
```

1. **Descomenta** la importación de `mssql` y la función `getPool()` al inicio del archivo.
2. En cada una de las 5 operaciones CRUD:
   - **GET /**: Reemplaza el array con `SELECT Id, Nombre, Email, Telefono FROM Contactos`.
   - **GET /:id**: Reemplaza el array con `SELECT ... WHERE Id = @id`.
   - **POST /**: Reemplaza `contactos.push(...)` con el `INSERT INTO Contactos ... OUTPUT INSERTED.*`.
   - **PUT /:id**: Reemplaza la actualización en memoria con `UPDATE Contactos SET ... WHERE Id = @id`.
   - **DELETE /:id**: Reemplaza `contactos.splice(...)` con `DELETE FROM Contactos WHERE Id = @id`.

> ⚠️ **Principio de Seguridad:** Observa cómo las consultas nativas utilizan `.input('parametro', sql.Tipo, valor)` en lugar de concatenar cadenas. Esto evita ataques de **Inyección SQL (SQLi)**.

---

## 📡 Referencia de la API REST

| Método | Endpoint | Descripción | Body (JSON) |
|---|---|---|---|
| `GET` | `/api/contactos` | Obtener todos los contactos | N/A |
| `GET` | `/api/contactos/:id` | Obtener un contacto por ID | N/A |
| `POST` | `/api/contactos` | Crear un nuevo contacto | `{"nombre":"...", "email":"...", "telefono":"..."}` |
| `PUT` | `/api/contactos/:id` | Actualizar un contacto existente | `{"nombre":"...", "email":"...", "telefono":"..."}` |
| `DELETE` | `/api/contactos/:id` | Eliminar un contacto por ID | N/A |
| `GET` | `/api/health` | Estado del servicio | N/A |

# 🎥 Sistema de Vigilancia con WebRTC - Versión Segura

Sistema de vigilancia en tiempo real con autenticación, usando WebRTC para streaming de video peer-to-peer.

## 🔐 Características de Seguridad

- ✅ **Autenticación con contraseña** - Login obligatorio para acceder
- ✅ **Tokens de sesión** - Sesiones válidas por 24 horas
- ✅ **Contraseñas hasheadas** - SHA-256 para almacenamiento seguro
- ✅ **Control de acceso basado en roles** - Cámaras vs Viewers
- ✅ **Timeout de autenticación** - 10 segundos para autenticarse
- ✅ **Limpieza automática de sesiones** - Sesiones expiradas se eliminan

## 📦 Instalación

```bash
# Clonar o crear el proyecto
mkdir sistema-vigilancia
cd sistema-vigilancia

# Instalar dependencias
npm install express ws uuid

# Crear estructura de carpetas
mkdir public

# Colocar archivos:
# - server.js en la raíz
# - index.html en /public
```

## 🚀 Uso

### 1. Iniciar el servidor

```bash
npm start
```

El servidor mostrará las credenciales de prueba al iniciar:

```
=== CREDENCIALES DE PRUEBA ===
Cámaras:
  - usuario: camera1, contraseña: cam123
  - usuario: camera2, contraseña: cam456
Viewers:
  - usuario: viewer1, contraseña: view123
  - usuario: admin, contraseña: admin123
```

### 2. Acceder a la aplicación

Abre tu navegador en: `http://localhost:3000`

### 3. Iniciar sesión

**Para transmitir video (Cámara):**
1. Selecciona "📹 Cámara" en el tipo de usuario
2. Usa credenciales de cámara (ej: camera1 / cam123)
3. Haz clic en "Iniciar Sesión"

**Para ver cámaras (Viewer):**
1. Selecciona "👁️ Visualización" en el tipo de usuario
2. Usa credenciales de viewer (ej: viewer1 / view123)
3. Haz clic en "Iniciar Sesión"

### 4. Usar el sistema

**Modo Cámara:**
- Configura nombre y calidad
- Ajusta brillo, contraste y zoom
- Activa modo nocturno si es necesario
- Haz clic en "Iniciar Cámara"
- Permite acceso a la webcam

**Modo Visualización:**
- Haz clic en "Conectar"
- Selecciona una cámara de la lista
- Visualiza el stream en tiempo real

## 🔑 Gestión de Usuarios

### Agregar nuevos usuarios

Edita el objeto `USERS` en `server.js`:

```javascript
const USERS = {
  cameras: {
    'mi_camara': hashPassword('mi_password'),
    'camara_entrada': hashPassword('pass123')
  },
  viewers: {
    'vigilante1': hashPassword('vigil123'),
    'admin': hashPassword('admin123')
  }
};
```

### Generar hash de contraseña

Puedes usar este código en Node.js:

```javascript
const crypto = require('crypto');
function hashPassword(password) {
  return crypto.createHash('sha256').update(password).digest('hex');
}
console.log(hashPassword('tu_password'));
```

## 🔒 Configuración de Seguridad en Producción

### 1. Variables de entorno

Nunca dejes contraseñas en el código. Usa variables de entorno:

```javascript
// .env file
CAMERA_USER_1=camera1
CAMERA_PASS_1=contraseña_segura_aqui
VIEWER_USER_1=viewer1
VIEWER_PASS_1=otra_contraseña_segura

// En server.js
require('dotenv').config();

const USERS = {
  cameras: {
    [process.env.CAMERA_USER_1]: hashPassword(process.env.CAMERA_PASS_1)
  },
  viewers: {
    [process.env.VIEWER_USER_1]: hashPassword(process.env.VIEWER_PASS_1)
  }
};
```

### 2. Base de datos

Para producción, usa una base de datos real:

```javascript
// Ejemplo con PostgreSQL
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL
});

async function verifyUser(username, password, role) {
  const hashedPassword = hashPassword(password);
  const result = await pool.query(
    'SELECT * FROM users WHERE username = $1 AND password = $2 AND role = $3',
    [username, hashedPassword, role]
  );
  return result.rows.length > 0;
}
```

### 3. HTTPS obligatorio

```javascript
// Redirigir HTTP a HTTPS
app.use((req, res, next) => {
  if (req.header('x-forwarded-proto') !== 'https' && process.env.NODE_ENV === 'production') {
    res.redirect(`https://${req.header('host')}${req.url}`);
  } else {
    next();
  }
});
```

### 4. Rate limiting

```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutos
  max: 5, // 5 intentos máximo
  message: 'Demasiados intentos de login, intente más tarde'
});

app.post('/api/login', loginLimiter, async (req, res) => {
  // ... código de login
});
```

## 📡 Despliegue

### Render.com (Recomendado)

1. Crea cuenta en [Render.com](https://render.com)
2. Conecta tu repositorio de GitHub
3. Crea un nuevo "Web Service"
4. Configura:
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
5. Agrega variables de entorno en el panel de Render

### Otras opciones

- **Heroku:** Similar a Render
- **Railway:** Deployment automático desde GitHub
- **DigitalOcean:** Droplets con Node.js
- **AWS:** EC2 o Elastic Beanstalk

## 🛡️ Mejores Prácticas de Seguridad

1. ✅ **Nunca** commits contraseñas en el código
2. ✅ Usa **contraseñas fuertes** (mínimo 12 caracteres)
3. ✅ Implementa **2FA** para usuarios administrativos
4. ✅ Mantén **logs** de accesos
5. ✅ Implementa **backup** de sesiones críticas
6. ✅ Usa **HTTPS** en producción
7. ✅ Actualiza **dependencias** regularmente
8. ✅ Implementa **rate limiting** agresivo
9. ✅ Usa **WebSockets seguros** (wss://)
10. ✅ Implementa **expiración de sesiones**

## 📊 Monitoreo

Agrega logs para auditoría:

```javascript
// En server.js
function logAccess(userId, action) {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${userId} - ${action}`);
  // Opcional: guardar en base de datos o archivo
}

// Usar en eventos importantes
logAccess(session.userId, 'LOGIN_SUCCESS');
logAccess(session.userId, 'CAMERA_STARTED');
logAccess(session.userId, 'VIEWER_CONNECTED');
```

## 🐛 Solución de Problemas

### "Token inválido o expirado"
- La sesión caducó después de 24 horas
- Cierra sesión y vuelve a iniciar sesión

### "No autorizado para transmitir"
- Estás usando credenciales de viewer
- Usa credenciales de cámara para transmitir

### "No autorizado para ver"
- Estás usando credenciales de cámara
- Usa credenciales de viewer para visualizar

## 📝 Estructura del Proyecto

```
sistema-vigilancia/
├── server.js           # Servidor con autenticación
├── package.json        # Dependencias
├── .env               # Variables de entorno (no commitear)
├── .gitignore         # Ignorar .env y node_modules
└── public/
    └── index.html     # Frontend con login
```

## 🤝 Contribuir

1. Fork el proyecto
2. Crea una rama para tu feature
3. Commit tus cambios
4. Push a la rama
5. Abre un Pull Request

## 📄 Licencia

MIT License - Usa como quieras, pero sin garantías.

## ⚠️ Disclaimer

Este es un sistema de ejemplo. Para uso en producción:
- Implementa una base de datos real
- Usa bcrypt en lugar de SHA-256
- Implementa 2FA
- Agrega logs completos
- Implementa backups
- Contrata auditoría de seguridad
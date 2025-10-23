# 🎥 Sistema de Vigilancia con WebRTC - V3

Plataforma de monitoreo en tiempo real con autenticación, para streaming de video peer-to-peer.

## 🔐 Características de Seguridad

- ✅ **Autenticación con contraseña** - Login obligatorio para acceder
- ✅ **Tokens de sesión** - Sesiones válidas por 24 horas
- ✅ **Control de acceso basado en roles** - Cámaras vs Viewers
- ✅ **Timeout de autenticación** - 10 segundos para autenticarse
- ✅ **Limpieza automática de sesiones** - Sesiones expiradas se eliminan

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


# 🔍 Debugging de Errores - Client Gateway

## 🎯 **Mejoras Implementadas**

He agregado logging detallado y mejor manejo de errores para identificar exactamente qué está fallando.

---

## 📋 **Nuevos Endpoints de Prueba**

### **1. Health Check (Verificar que el gateway esté funcionando)**
```http
GET http://localhost:3000/auth/health
```

**Respuesta esperada:**
```json
{
  "status": "ok",
  "message": "Auth controller is working", 
  "timestamp": "2025-06-09T15:30:00.000Z"
}
```

### **2. Test Connection (Verificar conexión con NATS/Auth-MS)**
```http
GET http://localhost:3000/auth/test-connection
```

**Si funciona:** Respuesta del auth-ms
**Si falla:** Error detallado de conexión NATS

---

## 🔧 **Pasos de Debugging**

### **Paso 1: Verificar que el Gateway responde**
```http
GET http://localhost:3000/auth/health
```
✅ **Si funciona:** Gateway está bien
❌ **Si falla:** Problema con Docker/puerto 3000

### **Paso 2: Verificar conexión con Auth-MS**
```http
GET http://localhost:3000/auth/test-connection
```
✅ **Si funciona:** NATS y Auth-MS están conectados
❌ **Si falla:** Problema de conectividad entre servicios

### **Paso 3: Probar registro con logs detallados**
```http
POST http://localhost:3000/auth/register-subscription
Content-Type: application/json

{
  "airline": {
    "airline_name": "Test Airlines",
    "alias": "test-air",
    "country": "Bolivia", 
    "contact_email": "test@testairlines.com",
    "phone_number": "+591 2 1234567"
  },
  "admin": {
    "admin_name": "Test Admin",
    "admin_email": "admin@testairlines.com", 
    "admin_password": "TestPass123!",
    "admin_phone": "+591 70123456"
  },
  "payment": {
    "card_number": "1234567890123456",
    "cardholder_name": "Test Admin",
    "expiry_date": "12/25",
    "cvv": "123",
    "plan": "premium"
  }
}
```

**Ahora obtendrás errores más descriptivos como:**
```json
{
  "status": "error",
  "message": "Ya existe una aerolínea con el alias: test-air",
  "details": "Error en el registro de suscripción"
}
```

O si es un error de validación:
```json
{
  "status": "error", 
  "message": "Error al procesar la suscripción",
  "details": {
    "field": "admin_email",
    "error": "El formato del correo electrónico no es válido"
  }
}
```

---

## 📊 **Ver Logs en Docker**

Para ver logs detallados mientras pruebas:

```bash
# En otra terminal, ver logs del client-gateway
docker compose logs -f client-gateway

# Ver logs del auth-ms
docker compose logs -f auth-ms

# Ver logs de ambos
docker compose logs -f client-gateway auth-ms
```

---

## 🔍 **Posibles Causas del "Internal Server Error"**

### **1. Problemas de Conectividad:**
- NATS server no está funcionando
- Auth-MS no se puede conectar a MongoDB
- Client-Gateway no puede comunicarse con Auth-MS

### **2. Problemas de Validación:**
- Datos del JSON con formato incorrecto
- Campos requeridos faltantes
- Validaciones que fallan

### **3. Problemas de Base de Datos:**
- MongoDB Atlas no accesible
- Problemas de autenticación en MongoDB
- Timeout de conexión

---

## 🚀 **Reiniciar y Probar**

```bash
# 1. Parar todo
docker compose down

# 2. Limpiar (opcional)
docker system prune -f

# 3. Volver a levantar con logs
docker compose up --build

# 4. En otra terminal, seguir logs
docker compose logs -f

# 5. Probar endpoints paso a paso
```

---

## 💡 **Ejemplo de Request Simplificado**

Si el JSON completo falla, prueba con datos más simples:

```json
{
  "airline": {
    "airline_name": "Test",
    "alias": "test",
    "country": "Bolivia", 
    "contact_email": "test@test.com",
    "phone_number": "1234567"
  },
  "admin": {
    "admin_name": "Test",
    "admin_email": "admin@test.com", 
    "admin_password": "Test123!",
    "admin_phone": "1234567"
  },
  "payment": {
    "card_number": "1234567890123456",
    "cardholder_name": "Test",
    "expiry_date": "12/25",
    "cvv": "123",
    "plan": "premium"
  }
}
```

**Ahora deberías obtener errores mucho más específicos y útiles para debugging!** 🔧

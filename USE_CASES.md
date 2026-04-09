# 📱 Casos de Uso del Plugin VoIpUSSD

## ¿Para qué puedes usar este plugin?

El plugin **VoIpUSSD** te permite automatizar interacciones con servicios USSD de operadoras telefónicas y bancos desde tu aplicación Android.

---

## 🎯 5 Casos de Uso Principales

### 1. **Consultar Saldo** 💵
```kotlin
USSDController.callUSSDInvoke(this, "*152#", map,
    object : USSDController.CallbackInvoke {
        override fun over(msg: String) {
            // msg: "Tu saldo es: $50.00"
            mostrarSaldo(msg)
        }
    }
)
```
**Ideal para**: Apps que quieran mostrar saldo sin salir de la aplicación.

---

### 2. **Recargar Saldo Automáticamente** 💰
```kotlin
// Paso 1: Iniciar USSD
USSDController.callUSSDInvoke(this, "*123#", map,
    object : USSDController.CallbackInvoke {
        override fun responseInvoke(msg: String) {
            // Paso 2: Seleccionar monto
            if (msg.contains("1. Recargar")) {
                USSDController.send("1") { response ->
                    // Paso 3: Confirmar
                    USSDController.send("1") // $5
                }
            }
        }
        override fun over(msg: String) {
            // Paso 4: Resultado
            mostrarExito(msg)
        }
    }
)
```
**Ideal para**: Aplicaciones de recarga directa sin navegador.

---

### 3. **Activar Planes de Datos** 📊
```kotlin
// *156# → Seleccionar plan → Confirmar → Completado
```
**Ideal para**: Apps que venden datos/internet.

---

### 4. **Pagar Servicios** 💳
- Teléfono
- Internet
- Agua
- Luz
- Cable

**Ideal para**: Apps de finanzas y pagos.

---

### 5. **Transacciones Bancarias** 🏦
- Transferencias
- Pagos
- Retiros
- Consultas

**Ideal para**: Apps bancarias como backup a API.

---

## 🏢 Tipos de Aplicaciones

### Tipo 1: App de Operadora Móvil
```
┌──────────────────────┐
│  Mi Operadora        │
├──────────────────────┤
│ Saldo: $50.00        │
│ [Recargar]           │
│ [Ver planes]         │
│ [Datos]              │
│ [Transferencias]     │
└──────────────────────┘
```

### Tipo 2: App de Pagos
```
┌──────────────────────┐
│  Pagar Servicios     │
├──────────────────────┤
│ □ Teléfono: $15      │
│ □ Internet: $25      │
│ □ Agua: $10          │
│ □ Luz: $30           │
│                      │
│ [Pagar Todo]         │
└──────────────────────┘
```

### Tipo 3: App Bancaria
```
┌──────────────────────┐
│  Mi Banco            │
├──────────────────────┤
│ Saldo: $500.00       │
│ [Transferencias]     │
│ [Pagos]              │
│ [Retiros]            │
│ [Consultas]          │
└──────────────────────┘
```

### Tipo 4: App de Remesas
```
┌──────────────────────┐
│  Enviar Dinero       │
├──────────────────────┤
│ Destino: USSD        │
│ Monto: $100          │
│ Comisión: $2         │
│                      │
│ [Enviar]             │
└──────────────────────┘
```

### Tipo 5: App de IoT/Pagos
```
Máquina expendedora → Compra → Paga vía USSD
```

---

## ✅ Ventajas del Plugin

| Ventaja | Descripción |
|---------|-------------|
| **Sin API** | No depende de servicios de terceros |
| **Sin internet** | Funciona solo con conexión telefónica |
| **Bajo costo** | USSD es gratis o muy barato |
| **Universal** | Funciona con cualquier operadora |
| **Móvil first** | Optimizado para celulares |
| **Sin root** | Solo requiere AccessibilityService |
| **Multistep** | Soporta menús interactivos |
| **Confiable** | Usa sistema operativo del teléfono |

---

## 🌍 Mercados Donde Es Más Útil

### América Latina 🇲🇽 🇨🇴 🇵🇪 🇦🇷 🇨🇱
- USSD muy popular en recarga y consultas
- Muchos usuarios sin acceso a apps de operadoras

### África 🇳🇬 🇰🇪 🇬🇭
- USSD es estándar
- Datos limitados
- Muy usado en finanzas

### Asia 🇮🇳 🇵🇭 🇵🇰
- USSD ampliamente usado
- Mercado masivo
- Apps financieras

### Global 🌐
- Cualquier país con operadoras móviles USSD

---

## 💡 Casos Reales de Éxito

### 1. **Portel/Movistar/Claro**
- Integran USSD en app principal
- Consulta de saldo y recarga

### 2. **Apps de Remesas**
- Enviar dinero vía USSD
- Retirar sin tarjeta

### 3. **Apps Bancarias**
- Backup cuando API falla
- Consultas de saldo directas

### 4. **Apps Fintech**
- Microcrédit
- Seguros
- Inversiones

### 5. **Dispositivos IoT**
- Máquinas expendedoras
- Parquímetros
- Wifi público

---

## ✅ Cuándo Usar Este Plugin

### ✅ Ideal para:
- Consultar saldo de celular
- Recargar crédito
- Activar datos/planes
- Pagar servicios
- Transacciones sin API
- Mercados sin mucho 4G/5G
- Aplicaciones que necesiten funcionar offline
- Backup de APIs principales

### ❌ No es ideal para:
- Aplicaciones web (solo Android)
- Transacciones ultra-seguras (mejor OAuth)
- Aplicaciones que requieren APIs robustas
- Procesos automáticos sin interacción

---

## 📋 Ejemplo Completo: App de Recarga

```kotlin
class RecargaActivity : AppCompatActivity() {
    
    fun recargarSaldo() {
        // PASO 1: Iniciar USSD
        USSDController.callUSSDInvoke(
            this,
            "*123#",
            hashMapOf(
                "KEY_LOGIN" to listOf("Procesando..."),
                "KEY_ERROR" to listOf("Error")
            ),
            object : USSDController.CallbackInvoke {
                
                override fun responseInvoke(msg: String) {
                    // PASO 2: Mostrar opciones de monto
                    if (msg.contains("1. Recargar")) {
                        mostrarDialogoMontos()
                    }
                }
                
                override fun over(msg: String) {
                    // PASO 4: Resultado
                    if (msg.contains("exitoso")) {
                        Toast.makeText(this@RecargaActivity,
                            "¡Recarga exitosa!",
                            Toast.LENGTH_LONG).show()
                    }
                }
            }
        )
    }
    
    fun mostrarDialogoMontos() {
        AlertDialog.Builder(this)
            .setTitle("Seleccione monto:")
            .setItems(arrayOf("$5", "$10", "$20")) { _, which ->
                // PASO 3: Enviar opción elegida
                USSDController.send((which + 1).toString()) { response ->
                    Log.d("USSD", "Respuesta: $response")
                }
            }
            .show()
    }
}
```

---

## 🚀 Cómo Empezar

1. **Instalar el plugin**
   ```gradle
   implementation 'com.github.romellfudi.VoIpUSSD:ussd-library-kotlin:v1.2.g'
   ```

2. **Agregar permisos**
   ```xml
   <uses-permission android:name="android.permission.CALL_PHONE" />
   <uses-permission android:name="android.permission.READ_PHONE_STATE" />
   <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
   ```

3. **Habilitar AccessibilityService**
   - Configuración → Accesibilidad → Tu App

4. **Implementar callback**
   - Usar `callUSSDInvoke()` o `callUSSDOverlayInvoke()`
   - Procesar respuestas y menús

5. **Detectar MultiStep y enviar opciones**
   - Si respuesta contiene "1. Opción" → es menú
   - Usar `send()` para enviar opción

---

## 📞 Códigos USSD Comunes (Varía por operadora)

| Función | Código |
|---------|--------|
| Consultar saldo | *152# o *100# |
| Recargar | *123# o *121# |
| Datos/Internet | *156# o *155# |
| Transferencias | *157# |
| Ofertas | *159# |
| Soporte | *166# |

*Estos códigos varían según la operadora. Verifica con la tuya.*

---

**Resumen**: Con este plugin puedes crear aplicaciones que interactúen directamente con servicios USSD, permitiendo a tus usuarios recargar, consultar saldo, pagar servicios y más, **sin salir de tu aplicación y sin necesidad de acceso a internet**.

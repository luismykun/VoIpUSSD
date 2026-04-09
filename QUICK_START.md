# 🚀 GUÍA RÁPIDA - Usar VoIpUSSD v1.2.g (Actualizado)

**Status**: ✅ Listo para Google Play Store  
**Android**: 6.0+ hasta 15 (2024)  
**Publicación**: JitPack / Maven Central

---

## 📦 Instalación

### build.gradle
```gradle
dependencies {
    // Kotlin
    implementation 'com.github.romellfudi.VoIpUSSD:ussd-library-kotlin:v1.2.g'
    
    // O Java
    implementation 'com.github.romellfudi.VoIpUSSD:ussd-library:v1.2.g'
}

repositories {
    maven { url 'https://jitpack.io' }
}
```

---

## ⚙️ Configuración

### AndroidManifest.xml
```xml
<uses-permission android:name="android.permission.CALL_PHONE" />
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />

<service
    android:name="com.romellfudi.ussdlibrary.USSDServiceKT"
    android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
    <intent-filter>
        <action android:name="android.accessibilityservice.AccessibilityService" />
    </intent-filter>
    <meta-data
        android:name="android.accessibilityservice"
        android:resource="@xml/ussd_service" />
</service>
```

---

## 💻 Uso Básico

### USSD Simple (Single-Step)
```kotlin
USSDController.callUSSDInvoke(
    context = this,
    ussdPhoneNumber = "*152#",  // Consultar saldo
    map = hashMapOf(
        "KEY_LOGIN" to listOf("Cargando..."),
        "KEY_ERROR" to listOf("Error")
    ),
    callbackInvoke = object : USSDController.CallbackInvoke {
        override fun responseInvoke(message: String) {
            Log.d("USSD", message)
        }
        override fun over(message: String) {
            Toast.makeText(context, message, Toast.LENGTH_LONG).show()
        }
    }
)
```

### USSD MultiStep (Con Menú)
```kotlin
USSDController.callUSSDInvoke(
    context = this,
    ussdPhoneNumber = "*123#",
    map = hashMapOf(
        "KEY_LOGIN" to listOf("Cargando..."),
        "KEY_ERROR" to listOf("Error")
    ),
    callbackInvoke = object : USSDController.CallbackInvoke {
        
        override fun responseInvoke(message: String) {
            // El servidor responde con menú
            Log.d("USSD", "Menú: $message")
            
            // Detectar si es multiStep
            if (message.contains(Regex("1\\.|2\\.|3\\."))) {
                // Es un menú - Enviar opción
                USSDController.send("1") { response ->
                    // Respuesta del servidor
                    Log.d("USSD", "Respuesta: $response")
                    Toast.makeText(context, response, Toast.LENGTH_LONG).show()
                }
            }
        }
        
        override fun over(message: String) {
            Log.d("USSD", "Finalizado: $message")
        }
    }
)
```

---

## 🎯 Ejemplos Comunes

### Consultar Saldo
```kotlin
fun consultarSaldo() {
    USSDController.callUSSDInvoke(
        this, "*152#", hashMapOf(
            "KEY_LOGIN" to listOf("loading"),
            "KEY_ERROR" to listOf("error")
        ),
        object : USSDController.CallbackInvoke {
            override fun responseInvoke(message: String) {}
            override fun over(message: String) {
                if (message.contains("\$") || message.contains("saldo")) {
                    Toast.makeText(this@MainActivity, message, Toast.LENGTH_LONG).show()
                }
            }
        }
    )
}
```

### Recargar (4 pasos)
```kotlin
fun recargar() {
    USSDController.callUSSDInvoke(
        this, "*123#", hashMapOf(
            "KEY_LOGIN" to listOf("Conectando..."),
            "KEY_ERROR" to listOf("error")
        ),
        object : USSDController.CallbackInvoke {
            override fun responseInvoke(msg: String) {
                // Paso 1
                if (msg.contains("1. Recargar")) {
                    USSDController.send("1") {
                        // Paso 2
                        if (it.contains("1. \$5")) {
                            USSDController.send("1") {
                                // Paso 3
                                if (it.contains("Confirmar")) {
                                    USSDController.send("1") {
                                        // Paso 4 - Resultado
                                        Toast.makeText(this@MainActivity, it, Toast.LENGTH_LONG).show()
                                    }
                                }
                            }
                        }
                    }
                }
            }
            override fun over(msg: String) {}
        }
    )
}
```

---

## 🎨 Overlay (Mostrar Loading)
```kotlin
USSDController.callUSSDOverlayInvoke(
    this,
    "*152#",
    hashMapOf("KEY_LOGIN" to listOf("loading")),
    object : USSDController.CallbackInvoke {
        override fun responseInvoke(message: String) {}
        override fun over(message: String) {
            Toast.makeText(this@MainActivity, message, Toast.LENGTH_LONG).show()
        }
    }
)
```

---

## ⚡ Cancelar en Cualquier Momento
```kotlin
// Cancelar USSD actual
USSDController.cancel()

// O después de timeout
Handler(Looper.getMainLooper()).postDelayed({
    if (USSDController.isRunning == true) {
        USSDController.cancel()
    }
}, 30000) // 30 segundos
```

---

## ❓ Preguntas Frecuentes

**P: ¿Se puede hacer más de un USSD?**  
R: SÍ, puedes hacer múltiples `callUSSDInvoke()` en secuencia.

**P: ¿Cuándo es MultiStep?**  
R: Cuando el servidor responde con menú (opciones numeradas: 1. Opción...)

**P: ¿Funciona en Android 15?**  
R: SÍ, compilado y optimizado para Android 15 (API 35).

**P: ¿Necesito root?**  
R: NO, solo requiere AccesibilityService habilitado en Configuración.

**P: ¿Cuál es la duración máxima?**  
R: ~2 minutos de inactividad antes de timeout automático.

---

## 📚 Documentación Completa

- **[PUBLICATION_GUIDE.md](PUBLICATION_GUIDE.md)** - Publicar en Play Store
- **[USSD_MULTISTEP_GUIDE.md](USSD_MULTISTEP_GUIDE.md)** - Ejemplos detallados
- **[COMPATIBILITY_REPORT.md](COMPATIBILITY_REPORT.md)** - Compatibilidad
- **[UPDATE_SUMMARY.md](UPDATE_SUMMARY.md)** - Cambios v1.2.g

---

**¡Listo para usar!** ✅

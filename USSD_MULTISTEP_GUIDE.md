# 🔄 USSD MultiStep - Guía Completa

**Concepto**: El plugin soporta conversaciones USSD de múltiples pasos, permitiendo enviar solicitudes intermedias y recibir respuestas en cada etapa.

---

## 📊 Diagrama de Flujo USSD MultiStep

```
┌─────────────────────────────────────────┐
│  Inicio: Llamada USSD (*152#)           │
└──────────────────┬──────────────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │ Servidor USSD        │
        │ Envía: "Menú"        │
        │ 1. Consultar Saldo   │
        │ 2. Recargar          │
        │ 3. Ofertas           │
        └──────────────────────┘
                   │
                   ▼
     ┌───────────────────────────┐
     │ responseInvoke(message)   │
     │ Detecta: Es MultiStep     │
     │ showInputDialog()          │
     └──────────────┬─────────────┘
                    │
                    ▼ (Usuario selecciona 1)
        ┌──────────────────────┐
        │ send("1")            │
        │ callbackMessage()     │
        │ Servidor responde:    │
        │ "Tu saldo: $50"       │
        └──────────────────────┘
                   │
                   ▼
     ┌───────────────────────────┐
     │ callbackMessage("Tu...")  │
     │ Mostrar saldo al usuario  │
     │ cancel() - Cerrar sesión  │
     └───────────────────────────┘
```

---

## 💻 Ejemplo 1: Consulta de Saldo (2 pasos)

```kotlin
import com.romellfudi.ussdlibrary.USSDController

class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Paso 1: Iniciar USSD
        consultarSaldo()
    }
    
    private fun consultarSaldo() {
        USSDController.callUSSDInvoke(
            context = this,
            ussdPhoneNumber = "*152#",
            map = hashMapOf(
                "KEY_LOGIN" to listOf("Cargando...", "Por favor espere"),
                "KEY_ERROR" to listOf("Error", "Intente nuevamente")
            ),
            callbackInvoke = object : USSDController.CallbackInvoke {
                
                override fun responseInvoke(message: String) {
                    // Paso 2: Respuesta del menú USSD
                    Log.d("USSD", "Menú recibido: $message")
                    
                    if (message.contains("1. Consultar Saldo", ignoreCase = true)) {
                        // Es un menú multiStep
                        // Enviar opción 1 para consultar saldo
                        USSDController.send("1") { response ->
                            Log.d("USSD", "Saldo: $response")
                            Toast.makeText(this@MainActivity, response, Toast.LENGTH_LONG).show()
                        }
                    }
                }
                
                override fun over(message: String) {
                    // Sesión USSD finalizada
                    Log.d("USSD", "Finalizado: $message")
                }
            }
        )
    }
}
```

---

## 💻 Ejemplo 2: Recargar Saldo (4 pasos)

```kotlin
private fun recargarSaldo() {
    USSDController.callUSSDInvoke(
        context = this,
        ussdPhoneNumber = "*123#",
        map = hashMapOf(
            "KEY_LOGIN" to listOf("Conectando...", "Cargando"),
            "KEY_ERROR" to listOf("error", "fail")
        ),
        callbackInvoke = object : USSDController.CallbackInvoke {
            
            override fun responseInvoke(message: String) {
                // PASO 1: Menú principal
                if (message.contains("1. Recargar") && !isStep2Started) {
                    isStep2Started = true
                    Log.d("USSD", "Paso 1 - Menú principal: $message")
                    
                    // Enviar opción 1 - Recargar
                    USSDController.send("1") { response ->
                        // PASO 2: Seleccionar monto
                        Log.d("USSD", "Paso 2 - Opciones de monto: $response")
                        
                        if (response.contains("1. \$5") && !isStep3Started) {
                            isStep3Started = true
                            
                            // Enviar opción 1 - Monto $5
                            USSDController.send("1") { response2 ->
                                // PASO 3: Confirmar
                                Log.d("USSD", "Paso 3 - Confirmar: $response2")
                                
                                if (response2.contains("1. Confirmar") && !isStep4Started) {
                                    isStep4Started = true
                                    
                                    // Enviar opción 1 - Confirmar
                                    USSDController.send("1") { response3 ->
                                        // PASO 4: Resultado
                                        Log.d("USSD", "Paso 4 - Resultado: $response3")
                                        Toast.makeText(this@MainActivity, response3, Toast.LENGTH_LONG).show()
                                    }
                                }
                            }
                        }
                    }
                }
            }
            
            override fun over(message: String) {
                Log.d("USSD", "Transacción finalizada: $message")
            }
        }
    )
}

private var isStep2Started = false
private var isStep3Started = false
private var isStep4Started = false
```

---

## 💻 Ejemplo 3: Entrada de Usuario Interactiva

```kotlin
private fun menuInteractivo() {
    val commands = listOf(
        "*152#"  // Consultar saldo
    )
    
    for (command in commands) {
        USSDController.callUSSDInvoke(
            context = this,
            ussdPhoneNumber = command,
            map = hashMapOf(
                "KEY_LOGIN" to listOf("loading"),
                "KEY_ERROR" to listOf("error")
            ),
            callbackInvoke = object : USSDController.CallbackInvoke {
                
                override fun responseInvoke(message: String) {
                    if (isMultiStep(message)) {
                        // Mostrar diálogo para que usuario seleccione opción
                        showSelectionDialog(message) { selectedOption ->
                            USSDController.send(selectedOption) { response ->
                                handleResponse(response)
                            }
                        }
                    } else {
                        // Respuesta final
                        handleResponse(message)
                    }
                }
                
                override fun over(message: String) {
                    Log.d("USSD", "Done: $message")
                }
            }
        )
    }
}

private fun isMultiStep(message: String): Boolean {
    return message.contains(Regex("""(1\.|2\.|3\.|opción|seleccione|choose|menu)""", RegexOption.IGNORE_CASE))
}

private fun showSelectionDialog(message: String, onSelect: (String) -> Unit) {
    // Extraer opciones del mensaje
    val options = message.split("\n")
        .filter { it.matches(Regex("""\d+\..*""")) }
        .map { it.trim() }
    
    AlertDialog.Builder(this)
        .setTitle("Seleccione opción")
        .setItems(options.toTypedArray()) { _, which ->
            onSelect((which + 1).toString())
        }
        .show()
}

private fun handleResponse(message: String) {
    Log.d("USSD", "Response: $message")
    Toast.makeText(this, message, Toast.LENGTH_LONG).show()
}
```

---

## ✅ Criterios para Detectar MultiStep

```kotlin
fun isUSSDMultiStep(message: String): Boolean {
    val patterns = listOf(
        "\\d+\\.",              // "1.", "2.", "3.", etc
        "(?i)opción",           // "opción", "OPCIÓN"
        "(?i)seleccione",       // "seleccione"
        "(?i)choose",           // "choose"
        "(?i)select",           // "select"
        "(?i)menú|menu",        // "menú", "menu"
        "\\[\\d+\\]",           // "[1]", "[2]"
    )
    
    return patterns.any { pattern ->
        message.contains(Regex(pattern))
    }
}
```

---

## ⏱️ Timeouts y Duraciones

| Evento | Duración | Comportamiento |
|--------|----------|---|
| **Espera por respuesta USSD** | ~30 segundos | Timeout automático |
| **Inactividad en sesión** | ~2 minutos | `over()` se dispara automáticamente |
| **Entre pasos** | Inmediato | `send()` debe completarse antes de siguiente |
| **Retry en error** | Manual | Llamar nuevamente `callUSSDInvoke()` |

---

## 🚫 Manejo de Errores en MultiStep

```kotlin
private var retryCount = 0
private val maxRetries = 3

private fun consultarConReintentos() {
    if (retryCount >= maxRetries) {
        Toast.makeText(this, "Error después de 3 intentos", Toast.LENGTH_LONG).show()
        return
    }
    
    USSDController.callUSSDInvoke(
        context = this,
        ussdPhoneNumber = "*152#",
        map = hashMapOf(
            "KEY_LOGIN" to listOf("loading"),
            "KEY_ERROR" to listOf("error", "failed", "null")
        ),
        callbackInvoke = object : USSDController.CallbackInvoke {
            
            override fun responseInvoke(message: String) {
                if (message.contains("error", ignoreCase = true)) {
                    retryCount++
                    Log.w("USSD", "Error detectado, reintentando ($retryCount/$maxRetries)")
                    Handler(Looper.getMainLooper()).postDelayed({
                        consultarConReintentos()
                    }, 2000) // Esperar 2 segundos antes de reintentar
                } else {
                    retryCount = 0 // Reset counter
                    processValidResponse(message)
                }
            }
            
            override fun over(message: String) {
                Log.d("USSD", "Sesión finalizada")
            }
        }
    )
}

private fun processValidResponse(message: String) {
    Log.d("USSD", "Respuesta válida: $message")
}
```

---

## 🔄 Cancelación de Sesión

```kotlin
// Cancelar sesión USSD en cualquier momento
fun cancelarUSSD() {
    USSDController.cancel()
    Toast.makeText(this, "USSD cancelado", Toast.LENGTH_SHORT).show()
}

// Ejemplo: Cancelar después de 10 segundos si no hay respuesta
Handler(Looper.getMainLooper()).postDelayed({
    if (USSDController.isRunning == true) {
        USSDController.cancel()
        Log.w("USSD", "Timeout - USSD cancelado")
    }
}, 10000)
```

---

## 📊 Tabla Comparativa: Single vs MultiStep

| Aspecto | Single-Step | Multi-Step |
|--------|---|---|
| **Pasos** | 1 | 2+ |
| **Ejemplo** | `*100#` → Saldo | `*123#` → Seleccionar → Confirmar |
| **Callback** | `over()` | `responseInvoke()` + múltiples `send()` |
| **User Input** | No | Sí, en cada paso |
| **Duración** | ~5 seg | ~15-30 seg |
| **Complejidad** | Baja | Media-Alta |
| **Casos de Uso** | Consultas simples | Rechargos, pagos, transacciones |

---

## ✨ Mejores Prácticas

### ✅ DO
```kotlin
// ✅ Verificar siempre si es MultiStep
if (isMultiStep(message)) {
    // Permitir entrada del usuario
}

// ✅ Usar timeouts para cancelación
Handler().postDelayed({ USSDController.cancel() }, 30000)

// ✅ Mostrar feedback visual al usuario
showProgressDialog("Cargando menú USSD...")

// ✅ Parsear respuestas correctamente
val options = parseUSSDOptions(message)
```

### ❌ DON'T
```kotlin
// ❌ No asumir que siempre es MultiStep
USSDController.send("1")  // Podría fallar en single-step

// ❌ No hacer múltiples send() sin esperar respuesta
USSDController.send("1")
USSDController.send("2")  // Podría perderse

// ❌ No ignorar timeouts
// Sin manejo de timeout = aplicación colgada

// ❌ No confiar en estructura fija del mensaje
// Diferentes operadores = diferentes formatos
```

---

## 🎓 Resumen

El plugin VoIpUSSD **soporta completamente MultiStep USSD** mediante:

1. **`responseInvoke()`** - Recibe menú/respuestas
2. **`send()`** - Envía opciones del usuario
3. **`callbackMessage()`** - Respuestas intermedias
4. **`over()`** - Finalización de sesión

**Uso**: Encadenar múltiples `send()` en los callbacks para implementar conversaciones USSD complejas.

---


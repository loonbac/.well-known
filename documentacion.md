## 📝 Sistema de Logs

### Configuración Básica

```python
# Activar logs en consola
bot = TelegramBot(BOT_TOKEN, sheets_lib=sheets, ranges=RANGES, debug=True)

# Desactivar logs en consola (se guardan en memoria)
bot = TelegramBot(BOT_TOKEN, sheets_lib=sheets, ranges=RANGES, debug=False)
```

### Métodos de Logging

```python
# Obtener todos los logs
todos_los_logs = bot.get_logs()

# Obtener los últimos N logs
ultimos_10 = bot.get_logs(10)

# Limpiar logs
bot.clear_logs()

# Ver cantidad de logs actuales
cantidad = len(bot.get_logs())
```

### Implementación con Menú Interactivo

```python
import threading
import time

def ejecutar_bot():
    """Ejecutar bot en segundo plano"""
    bot.enviar_vencimientos_proximos(ALLOWED_GROUP_ID)
    bot.run(ALLOWED_GROUP_ID, DEVELOPER_ID)

def mostrar_logs_tiempo_real():
    """Mostrar logs en tiempo real"""
    print("📋 LOGS EN TIEMPO REAL (Presiona Enter para salir)")
    logs_mostrados = len(bot.get_logs())
    
    try:
        while True:
            logs_actuales = bot.get_logs()
            if len(logs_actuales) > logs_mostrados:
                nuevos_logs = logs_actuales[logs_mostrados:]
                for log in nuevos_logs:
                    print(log)
                logs_mostrados = len(logs_actuales)
            time.sleep(0.5)
            # Aquí puedes agregar lógica para salir del bucle
    except KeyboardInterrupt:
        pass

def main():
    # Bot en segundo plano
    bot_thread = threading.Thread(target=ejecutar_bot, daemon=True)
    bot_thread.start()
    
    while True:
        print("\n=== MENÚ ===")
        print("1. Ver logs")
        print("2. Ver últimos 10 logs")
        print("3. Limpiar logs")
        print("4. Salir")
        
        opcion = input("Opción: ")
        
        if opcion == "1":
            logs = bot.get_logs()
            for log in logs:
                print(log)
        elif opcion == "2":
            logs = bot.get_logs(10)
            for log in logs:
                print(log)
        elif opcion == "3":
            bot.clear_logs()
            print("Logs limpiados")
        elif opcion == "4":
            break

if __name__ == '__main__':
    main()
```

## 🎯 Comandos del Bot

### Para el Usuario (en Telegram)

- `/pendientes` - Ver todas las facturas pendientes de pago
- `/pagado` - Marcar como pagada (debe ser respuesta a un mensaje de factura del bot)

### Funciones Programáticas

```python
# Enviar facturas próximas a vencer (por defecto 5 días)
bot.enviar_vencimientos_proximos(ALLOWED_GROUP_ID)

# Enviar facturas próximas a vencer en X días
bot.enviar_vencimientos_proximos(ALLOWED_GROUP_ID, dias=3)

# Obtener estadísticas
total_logs = len(bot.get_logs())
facturas_en_memoria = len(getattr(bot, 'msgid_to_facturaid', {}))
```

## 📋 Tipos de Logs

El sistema incluye logs con emojis para fácil identificación:

- 🔄 **Procesos iniciados**
- 📊 **Datos estadísticos**
- ⏰ **Filtros de tiempo**
- 💳 **Estados de pago**
- 📤 **Envío de mensajes**
- ✅ **Operaciones exitosas**
- ⚠️ **Advertencias**
- ❌ **Errores**
- 💬 **Mensajes procesados**
- 🔍 **Búsquedas**
- 💰 **Comandos de pago**
- 💾 **Actualizaciones**
- 🚀 **Inicio del bot**
- 📨 **Mensajes recibidos**
- 💥 **Errores críticos**

### Modificar Límite de Logs

```python
bot.max_logs = 200  # Cambiar límite de logs en memoria
```

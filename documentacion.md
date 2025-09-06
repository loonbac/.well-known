# 📋 TelegramBot - Sistema de Logging

Este README documenta las funciones de logging disponibles en la librería `TelegramBot` para monitorear y visualizar la actividad del bot en tiempo real.

## 📚 Índice

- [Descripción del Sistema de Logging](#descripción-del-sistema-de-logging)
- [Funciones Disponibles](#funciones-disponibles)
- [Ejemplos de Uso](#ejemplos-de-uso)
- [Configuración del Logging](#configuración-del-logging)
- [Casos de Uso Comunes](#casos-de-uso-comunes)

## 📝 Descripción del Sistema de Logging

La librería `TelegramBot` incluye un sistema interno de logging que:

- ✅ **Almacena logs en memoria** para consulta posterior
- ✅ **Usa emojis** para identificar tipos de eventos fácilmente
- ✅ **Gestiona automáticamente** el límite de logs (máximo 100 entradas)
- ✅ **Permite consulta bajo demanda** sin spam en consola
- ✅ **Registra todas las operaciones** del bot con timestamps

## 🛠️ Funciones Disponibles

### 1. `get_logs(last_n=None)`

Obtiene una lista de logs del bot para procesamiento o visualización.

**Parámetros:**
- `last_n` (int, opcional): Número de logs más recientes a obtener. Si es `None`, retorna todos los logs.

**Retorna:**
- `list`: Lista de strings con los logs formateados

**Ejemplo:**
```python
from telegram_bot_lib import TelegramBot

bot = TelegramBot(token, sheets_lib=sheets, ranges=ranges)

# Obtener todos los logs
todos_los_logs = bot.get_logs()

# Obtener los últimos 5 logs
ultimos_logs = bot.get_logs(5)

# Procesar logs
for log in ultimos_logs:
    print(log)
```

### 2. `print_logs(last_n=10)`

Imprime los logs directamente en consola de forma formateada.

**Parámetros:**
- `last_n` (int, opcional): Número de logs más recientes a mostrar. Por defecto son 10.

**Comportamiento:**
- Muestra un encabezado informativo
- Imprime cada log en una línea separada
- Maneja el caso cuando no hay logs disponibles

**Ejemplo:**
```python
from telegram_bot_lib import TelegramBot

bot = TelegramBot(token, sheets_lib=sheets, ranges=ranges)

# Imprimir los últimos 10 logs (por defecto)
bot.print_logs()

# Imprimir los últimos 20 logs
bot.print_logs(20)

# Imprimir todos los logs
bot.print_logs(len(bot.logs))
```

### 3. `clear_logs()`

Limpia todos los logs almacenados en memoria.

**Parámetros:**
- Ninguno

**Comportamiento:**
- Elimina todos los logs de la memoria
- Registra un evento de limpieza

**Ejemplo:**
```python
from telegram_bot_lib import TelegramBot

bot = TelegramBot(token, sheets_lib=sheets, ranges=ranges)

# Limpiar todos los logs
bot.clear_logs()
print("Logs limpiados")
```

## 🚀 Ejemplos de Uso

### Ejemplo 1: Monitoreo Básico

```python
import asyncio
from google_sheets_lib import GoogleSheetsReader
from telegram_bot_lib import TelegramBot

async def main():
    # Configurar bot
    sheets = GoogleSheetsReader('key.json', 'tu_sheet_id')
    bot = TelegramBot('tu_token', sheets_lib=sheets, ranges=ranges)
    
    # Ejecutar bot en background
    task = asyncio.create_task(bot.run(group_id, developer_id))
    
    # Monitorear logs cada 30 segundos
    while True:
        await asyncio.sleep(30)
        print("\n" + "="*50)
        print("📊 ESTADO DEL BOT")
        print("="*50)
        bot.print_logs(5)  # Mostrar últimos 5 logs
        
if __name__ == '__main__':
    asyncio.run(main())
```

### Ejemplo 2: Logging Avanzado con Filtros

```python
import asyncio
from datetime import datetime
from telegram_bot_lib import TelegramBot

def filtrar_logs_por_tipo(bot, emoji_filtro):
    """Filtrar logs por tipo de emoji"""
    todos_logs = bot.get_logs()
    logs_filtrados = [log for log in todos_logs if emoji_filtro in log]
    return logs_filtrados

async def monitoreo_avanzado():
    bot = TelegramBot('tu_token', sheets_lib=sheets, ranges=ranges)
    
    # Ejecutar bot
    task = asyncio.create_task(bot.run(group_id, developer_id))
    
    while True:
        await asyncio.sleep(60)  # Cada minuto
        
        print(f"\n🕐 {datetime.now().strftime('%H:%M:%S')} - Reporte de Estado")
        print("-" * 60)
        
        # Mostrar errores
        errores = filtrar_logs_por_tipo(bot, "❌")
        if errores:
            print("🚨 ERRORES RECIENTES:")
            for error in errores[-3:]:  # Últimos 3 errores
                print(f"   {error}")
        
        # Mostrar éxitos
        exitos = filtrar_logs_por_tipo(bot, "✅")
        print(f"\n✅ Operaciones exitosas: {len(exitos)}")
        
        # Mostrar actividad de mensajes
        mensajes = filtrar_logs_por_tipo(bot, "📨")
        print(f"📨 Mensajes procesados: {len(mensajes)}")
        
        print("-" * 60)

if __name__ == '__main__':
    asyncio.run(monitoreo_avanzado())
```

### Ejemplo 3: Sistema de Debugging

```python
import asyncio
from telegram_bot_lib import TelegramBot

class BotDebugger:
    def __init__(self, bot):
        self.bot = bot
        self.logs_anteriores = 0
    
    def mostrar_logs_nuevos(self):
        """Mostrar solo logs nuevos desde la última verificación"""
        logs_actuales = self.bot.get_logs()
        nuevos_logs = logs_actuales[self.logs_anteriores:]
        
        if nuevos_logs:
            print(f"\n🆕 {len(nuevos_logs)} nuevos eventos:")
            for log in nuevos_logs:
                print(f"   {log}")
        
        self.logs_anteriores = len(logs_actuales)
    
    def reporte_completo(self):
        """Generar reporte completo de actividad"""
        logs = self.bot.get_logs()
        
        # Contar tipos de eventos
        tipos = {}
        for log in logs:
            for emoji in ["🚀", "✅", "❌", "📨", "💰", "🔍", "💾"]:
                if emoji in log:
                    tipos[emoji] = tipos.get(emoji, 0) + 1
        
        print("\n📊 REPORTE COMPLETO DE ACTIVIDAD")
        print("=" * 50)
        print(f"Total de eventos: {len(logs)}")
        print("\nEventos por tipo:")
        for emoji, count in tipos.items():
            nombre = {
                "🚀": "Inicializaciones",
                "✅": "Operaciones exitosas", 
                "❌": "Errores",
                "📨": "Mensajes",
                "💰": "Comandos /pagado",
                "🔍": "Búsquedas",
                "💾": "Guardado de datos"
            }.get(emoji, "Otros")
            print(f"   {emoji} {nombre}: {count}")

async def debugging_session():
    bot = TelegramBot('tu_token', sheets_lib=sheets, ranges=ranges)
    debugger = BotDebugger(bot)
    
    # Ejecutar bot
    task = asyncio.create_task(bot.run(group_id, developer_id))
    
    # Sesión de debugging interactiva
    while True:
        await asyncio.sleep(10)  # Verificar cada 10 segundos
        debugger.mostrar_logs_nuevos()
        
        # Cada 5 minutos, mostrar reporte completo
        if len(bot.get_logs()) % 30 == 0:  # Aproximadamente cada 5 min
            debugger.reporte_completo()

if __name__ == '__main__':
    asyncio.run(debugging_session())
```

## ⚙️ Configuración del Logging

### Límite de Logs

Por defecto, el bot mantiene un máximo de **100 logs** en memoria:

```python
bot = TelegramBot(token, sheets_lib=sheets, ranges=ranges)
print(f"Límite actual: {bot.max_logs} logs")  # Output: 100

# Los logs más antiguos se eliminan automáticamente
# cuando se supera este límite
```

### Emojis de Eventos

El sistema usa estos emojis para categorizar eventos:

| Emoji | Tipo de Evento | Descripción |
|-------|----------------|-------------|
| 🚀 | Inicialización | Bot iniciado, configuración |
| ✅ | Éxito | Operaciones completadas exitosamente |
| ❌ | Error | Errores y fallos |
| 📨 | Mensajes | Envío y recepción de mensajes |
| 💰 | Pagos | Comandos /pagado |
| 🔍 | Búsqueda | Operaciones de búsqueda |
| 💾 | Datos | Guardado y carga de datos |
| 🧹 | Limpieza | Operaciones de mantenimiento |
| 🔒 | Seguridad | Autorización y permisos |

## 💡 Casos de Uso Comunes

### 1. **Debugging en Desarrollo**
```python
# Durante desarrollo, mostrar logs frecuentemente
bot.print_logs(20)  # Ver últimos 20 eventos
```

### 2. **Monitoreo en Producción**
```python
# En producción, verificar solo errores
errores = [log for log in bot.get_logs() if "❌" in log]
if errores:
    print("🚨 Hay errores que revisar:")
    for error in errores:
        print(error)
```

### 3. **Análisis de Performance**
```python
# Analizar cuántos mensajes se han enviado
mensajes_enviados = [log for log in bot.get_logs() if "📨" in log and "enviado" in log]
print(f"Mensajes enviados: {len(mensajes_enviados)}")
```

### 4. **Limpieza Periódica**
```python
# Limpiar logs después de guardarlos en archivo
logs_para_archivo = bot.get_logs()
# ... guardar en archivo ...
bot.clear_logs()  # Limpiar memoria
```

## 🔧 Integración con main.py

Para usar estas funciones desde tu `main.py`:

```python
# main.py
import asyncio
from google_sheets_lib import GoogleSheetsReader  
from telegram_bot_lib import TelegramBot

async def main():
    bot = TelegramBot(BOT_TOKEN, sheets_lib=sheets, ranges=RANGES)
    
    # Ejecutar bot y mostrar logs al final
    try:
        await bot.run(ALLOWED_GROUP_ID, DEVELOPER_ID)
    except KeyboardInterrupt:
        print("\n📋 Últimos logs del bot:")
        bot.print_logs(10)
    except Exception as e:
        print(f"❌ Error: {e}")
        print("\n📋 Logs para debugging:")
        bot.print_logs(20)

if __name__ == '__main__':
    asyncio.run(main())
```

---

## 📞 Soporte

Este sistema de logging está diseñado para ser:
- **Silencioso por defecto** (no hace spam en consola)
- **Accesible bajo demanda** (cuando tú lo solicites)
- **Informativo y detallado** (con emojis y timestamps)
- **Eficiente en memoria** (límite automático de logs)

Para más información sobre otras funciones del bot, consulta el código fuente en `telegram_bot_lib.py`.

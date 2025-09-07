# 📚 TelegramBot Library - Sistema de Logging

Documentación completa del sistema de logging de la librería `telegram_bot_lib.py` para integración y monitoreo desde código externo.

## 🚀 Introducción

La librería `TelegramBot` incluye un sistema de logging interno completo que permite monitorear en tiempo real todas las operaciones del bot sin interferir con la ejecución silenciosa. Este sistema está diseñado para ser consultado desde archivos externos como `main.py`.

### ✨ Características Principales

- **📝 Logging en memoria** - No genera archivos ni salida de consola automática
- **🔕 Operación silenciosa** - El bot funciona sin imprimir nada por defecto
- **📊 Monitoreo externo** - Acceso completo a logs desde código externo
- **⏰ Timestamps automáticos** - Cada log incluye hora exacta
- **🎨 Emojis descriptivos** - Identificación visual rápida del tipo de evento
- **🔄 Rotación automática** - Límite de logs para evitar uso excesivo de memoria

## 📋 Sistema de Logging

### Estructura del Log

Cada entrada de log tiene el formato:
```
[HH:MM:SS] 🔹 Mensaje descriptivo
```

### Tipos de Eventos Logged

| Emoji | Tipo | Descripción |
|-------|------|-------------|
| 📄 | Sistema | Operaciones de archivos y datos |
| 🔄 | Proceso | Carga/recarga de datos |
| 💾 | Datos | Guardado de información |
| 💰 | Comando | Comando /pagado ejecutado |
| 📊 | Estadísticas | Comando /stats o consultas de datos |
| 🧹 | Limpieza | Eliminación de datos obsoletos |
| 🔗 | Mapeo | Asociación mensaje-factura |
| 📤 | Envío | Mensajes enviados al grupo |
| ✅ | Éxito | Operaciones completadas correctamente |
| ❌ | Error | Errores y problemas |
| 🔍 | Búsqueda | Operaciones de búsqueda |
| 🚀 | Inicio | Arranque del bot |
| 🛑 | Parada | Cierre del bot |
| 🟢 | Estado | Bot ejecutándose |
| ⏰ | Tiempo | Operaciones relacionadas con fechas |
| 💳 | Filtro | Filtrado de datos |
| 📨 | Mensaje | Mensajes recibidos/enviados |
| 🔒 | Seguridad | Configuración de permisos |

## 🛠️ Métodos de Logging Disponibles

### 1. `log(emoji, message)`

**Propósito**: Agregar una entrada al sistema de logging interno.

```python
def log(self, emoji, message):
    """Sistema de logging interno - solo guarda en memoria"""
```

**Parámetros**:
- `emoji` (str): Emoji descriptivo del evento
- `message` (str): Mensaje descriptivo del evento

**Uso interno**: Este método es llamado automáticamente por el bot.

### 2. `get_logs(last_n=None)`

**Propósito**: Obtener logs para consulta externa sin mostrarlos en consola.

```python
def get_logs(self, last_n=None):
    """Obtener logs para mostrar en consola desde main.py"""
```

**Parámetros**:
- `last_n` (int, opcional): Número de logs más recientes a obtener
- Si es `None`: Retorna todos los logs disponibles

**Retorna**: Lista de strings con los logs formateados

**Ejemplo**:
```python
# Obtener todos los logs
todos_los_logs = bot.get_logs()

# Obtener últimos 5 logs
ultimos_logs = bot.get_logs(5)
```

### 3. `print_logs(last_n=10)`

**Propósito**: Imprimir logs en consola cuando se llama explícitamente.

```python
def print_logs(self, last_n=10):
    """Imprimir logs en consola - solo cuando se llama explícitamente"""
```

**Parámetros**:
- `last_n` (int): Número de logs recientes a mostrar (default: 10)

**Comportamiento**:
- Solo imprime cuando se llama explícitamente
- Incluye encabezado descriptivo
- Maneja el caso de logs vacíos

**Ejemplo**:
```python
# Mostrar últimos 10 logs
bot.print_logs()

# Mostrar últimos 20 logs
bot.print_logs(20)
```

### 4. `clear_logs()`

**Propósito**: Limpiar todos los logs del sistema.

```python
def clear_logs(self):
    """Limpiar logs"""
```

**Comportamiento**:
- Elimina todos los logs de memoria
- Agrega un log de confirmación de limpieza
- Útil para reiniciar el monitoreo

## 🔌 Integración con main.py

### Ejemplo Básico de Monitoreo

```python
# main.py
import asyncio
from telegram_bot_lib import TelegramBot
from google_sheets_lib import GoogleSheetsReader

async def main():
    # Configuración
    sheets = GoogleSheetsReader('credentials.json', 'sheet_id')
    bot = TelegramBot(token='bot_token', sheets_lib=sheets, ranges=ranges)
    
    # Ejecutar bot en background
    task = asyncio.create_task(bot.run(GROUP_ID, DEVELOPER_ID))
    
    # Monitoreo en tiempo real
    try:
        await asyncio.sleep(5)  # Esperar que el bot se inicialice
        
        # Mostrar logs de inicio
        print("🚀 Bot iniciado - Logs de arranque:")
        bot.print_logs(15)
        
        # Esperar a que termine el bot
        await task
        
    except KeyboardInterrupt:
        print("\n🛑 Deteniendo bot...")
        bot.print_logs(10)  # Mostrar últimos logs antes de cerrar
        task.cancel()

if __name__ == "__main__":
    asyncio.run(main())
```

### Monitoreo Periódico

```python
# main.py con monitoreo cada 30 segundos
import asyncio
from datetime import datetime

async def monitorear_bot_periodicamente(bot, intervalo=30):
    """Monitorear bot cada X segundos"""
    while True:
        try:
            await asyncio.sleep(intervalo)
            
            logs_recientes = bot.get_logs(5)
            if logs_recientes:
                print(f"\n📊 Estado del bot - {datetime.now().strftime('%H:%M:%S')}")
                print("─" * 50)
                for log in logs_recientes:
                    print(log)
                print("─" * 50)
            
        except asyncio.CancelledError:
            break
        except Exception as e:
            print(f"❌ Error en monitoreo: {e}")

async def main():
    # ... configuración del bot ...
    
    # Crear tareas
    bot_task = asyncio.create_task(bot.run(GROUP_ID, DEVELOPER_ID))
    monitor_task = asyncio.create_task(monitorear_bot_periodicamente(bot))
    
    try:
        await asyncio.gather(bot_task, monitor_task)
    except KeyboardInterrupt:
        print("\n🛑 Cerrando aplicación...")
        bot.print_logs(10)
        bot_task.cancel()
        monitor_task.cancel()
```

### Logging con Archivo Externo

```python
# main.py con logging a archivo
import asyncio
from datetime import datetime

def guardar_logs_a_archivo(bot, archivo="bot_logs.txt"):
    """Guardar logs del bot a archivo externo"""
    try:
        logs = bot.get_logs()
        with open(archivo, 'w', encoding='utf-8') as f:
            f.write(f"Logs del Bot - {datetime.now().isoformat()}\n")
            f.write("=" * 60 + "\n")
            for log in logs:
                f.write(log + "\n")
        print(f"✅ Logs guardados en {archivo}")
    except Exception as e:
        print(f"❌ Error guardando logs: {e}")

async def main():
    # ... configuración del bot ...
    
    try:
        # Ejecutar bot
        await bot.run(GROUP_ID, DEVELOPER_ID)
    finally:
        # Guardar logs al finalizar
        guardar_logs_a_archivo(bot)
```

## 📊 Ejemplos Prácticos

### 1. Dashboard Simple en Consola

```python
import asyncio
import os
from datetime import datetime

async def dashboard_simple(bot):
    """Dashboard simple para monitorear el bot"""
    while True:
        try:
            # Limpiar pantalla (Windows/Linux)
            os.system('cls' if os.name == 'nt' else 'clear')
            
            print("🤖 TELEGRAM BOT DASHBOARD")
            print("=" * 50)
            print(f"⏰ Actualizado: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
            
            # Estadísticas rápidas
            stats = bot.get_estadisticas()
            print(f"📊 Facturas procesadas: {stats.get('facturas_procesadas', 0)}")
            print(f"💰 Facturas pagadas: {stats.get('facturas_pagadas', 0)}")
            
            print("\n📝 ÚLTIMOS LOGS:")
            print("-" * 30)
            
            logs_recientes = bot.get_logs(8)
            for log in logs_recientes:
                print(log)
            
            print("\n⏳ Actualizando en 10 segundos... (Ctrl+C para salir)")
            await asyncio.sleep(10)
            
        except KeyboardInterrupt:
            print("\n👋 Dashboard cerrado")
            break
        except Exception as e:
            print(f"❌ Error en dashboard: {e}")
            await asyncio.sleep(5)

# Uso en main.py
async def main():
    # ... configuración ...
    
    bot_task = asyncio.create_task(bot.run(GROUP_ID, DEVELOPER_ID))
    dashboard_task = asyncio.create_task(dashboard_simple(bot))
    
    await asyncio.gather(bot_task, dashboard_task)
```

### 2. Filtrado de Logs por Tipo

```python
def filtrar_logs_por_emoji(bot, emoji_objetivo):
    """Filtrar logs por tipo de emoji"""
    todos_los_logs = bot.get_logs()
    logs_filtrados = [log for log in todos_los_logs if emoji_objetivo in log]
    return logs_filtrados

# Ejemplos de uso
def mostrar_solo_errores(bot):
    """Mostrar solo logs de error"""
    errores = filtrar_logs_por_emoji(bot, "❌")
    if errores:
        print("🚨 ERRORES DETECTADOS:")
        for error in errores:
            print(error)
    else:
        print("✅ No hay errores registrados")

def mostrar_solo_exitos(bot):
    """Mostrar solo operaciones exitosas"""
    exitos = filtrar_logs_por_emoji(bot, "✅")
    print(f"🎉 OPERACIONES EXITOSAS ({len(exitos)}):")
    for exito in exitos[-5:]:  # Últimos 5
        print(exito)
```

### 3. Análisis de Rendimiento

```python
def analizar_rendimiento(bot):
    """Analizar rendimiento basado en logs"""
    logs = bot.get_logs()
    
    # Contar tipos de eventos
    contadores = {}
    for log in logs:
        # Extraer emoji (primer carácter después del timestamp)
        try:
            emoji = log.split('] ')[1].split(' ')[0]
            contadores[emoji] = contadores.get(emoji, 0) + 1
        except IndexError:
            continue
    
    print("📊 ANÁLISIS DE ACTIVIDAD:")
    print("-" * 30)
    for emoji, count in sorted(contadores.items(), key=lambda x: x[1], reverse=True):
        print(f"{emoji} {count} eventos")
    
    # Estadísticas adicionales
    print(f"\n📈 Total de eventos: {len(logs)}")
    print(f"🔥 Tipo más frecuente: {max(contadores, key=contadores.get) if contadores else 'N/A'}")

# Uso en main.py
async def main():
    # ... después de ejecutar el bot por un tiempo ...
    
    print("\n" + "="*50)
    analizar_rendimiento(bot)
    print("="*50)
```

## ⚙️ Configuración y Personalización

### Modificar Límite de Logs

El bot por defecto mantiene un máximo de 100 logs en memoria:

```python
# Al crear el bot
bot = TelegramBot(
    token='tu_token',
    sheets_lib=sheets,
    ranges=ranges,
    debug=False  # False = silencioso, True = verbose
)

# Modificar límite después de crear
bot.max_logs = 200  # Mantener 200 logs en lugar de 100
```

### Habilitar Debug Mode

```python
# Bot con más información de debug
bot = TelegramBot(
    token='tu_token',
    sheets_lib=sheets,
    ranges=ranges,
    debug=True  # Habilita información adicional
)
```

### Logs Personalizados

Puedes agregar tus propios logs desde main.py:

```python
# Agregar log personalizado
bot.log("🔧", "Proceso personalizado ejecutado desde main.py")

# Ver el resultado
bot.print_logs(5)
```

## 🚨 Troubleshooting

### Problema: No aparecen logs

**Síntomas**: `get_logs()` retorna lista vacía

**Soluciones**:
```python
# Verificar que el bot se haya inicializado
if hasattr(bot, 'logs'):
    print(f"Sistema de logs activo: {len(bot.logs)} entradas")
else:
    print("❌ Sistema de logs no inicializado")

# Agregar log de prueba
bot.log("🧪", "Test de sistema de logging")
bot.print_logs(1)
```

### Problema: Demasiados logs en memoria

**Síntomas**: El bot consume mucha memoria

**Soluciones**:
```python
# Reducir límite de logs
bot.max_logs = 50

# Limpiar logs periódicamente
def limpiar_logs_periodicamente(bot, cada_minutos=30):
    import asyncio
    while True:
        await asyncio.sleep(cada_minutos * 60)
        bot.clear_logs()
        bot.log("🧹", f"Logs limpiados automáticamente cada {cada_minutos} min")
```

### Problema: Logs no se muestran en orden

**Síntomas**: Los logs aparecen desordenados

**Solución**: Los logs se almacenan en orden cronológico automáticamente. Si aparecen desordenados, verificar:

```python
# Verificar orden de logs
logs = bot.get_logs(5)
for i, log in enumerate(logs):
    print(f"{i+1}. {log}")
```

### Problema: Caracteres raros en logs

**Síntomas**: Emojis no se muestran correctamente

**Solución**:
```python
# Asegurar encoding UTF-8
import sys
if sys.stdout.encoding != 'utf-8':
    print("⚠️  Configurar terminal para UTF-8")
    
# En Windows PowerShell:
# [Console]::OutputEncoding = [System.Text.Encoding]::UTF8
```

## 🎯 Mejores Prácticas

### 1. **Monitoreo No Intrusivo**
- Usar `get_logs()` en lugar de `print_logs()` para análisis automático
- Implementar monitoreo asíncrono para no bloquear el bot

### 2. **Gestión de Memoria**
- Ajustar `max_logs` según necesidades
- Limpiar logs periódicamente en procesos de larga duración

### 3. **Debugging Efectivo**
- Usar filtros por emoji para encontrar tipos específicos de eventos
- Implementar dashboard simple para monitoreo visual

### 4. **Persistencia**
- Guardar logs críticos en archivos para análisis posterior
- Implementar rotación de archivos de log si es necesario

---

## 📞 Soporte

Para problemas con el sistema de logging:

1. **Verificar** que el bot esté correctamente inicializado
2. **Revisar** que `get_logs()` retorne datos
3. **Probar** con `bot.log("🧪", "test")` para verificar funcionamiento
4. **Consultar** esta documentación para ejemplos específicos

**¡El sistema de logging está diseñado para ser tu ventana hacia el funcionamiento interno del bot!** 🚀

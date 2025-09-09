# 🤖 Telegram Bot Lib - Bot de Gestión de Facturas

## 🎯 ¿Qué hace este módulo?

Este módulo proporciona un bot completo de Telegram para gestionar facturas empresariales de forma automática. Incluye:

- ✅ **Envío automático** de facturas próximas a vencer
- ✅ **Comando /pagado** para marcar facturas como pagadas
- ✅ **Comando /stats** para ver estadísticas del sistema
- ✅ **Mapeo de mensajes** para vincular respuestas con facturas específicas
- ✅ **Sistema de logs interno** sin archivos externos
- ✅ **Integración completa** con Google Sheets
- ✅ **Seguridad por chat** (solo usuarios autorizados)

## 🚀 Uso Básico

```python
from modulos.telegram_bot_lib import TelegramBot
from modulos.google_sheets_final import GoogleSheetsReader

# Configurar Google Sheets
RANGES = {
    'C': 'C2:C', 'D': 'D2:D', 'H': 'H2:H', 
    'J': 'J2:J', 'MONTO': 'F2:F'
}

# Crear instancias
sheets = GoogleSheetsReader("key.json", SHEET_ID, RANGES)
bot = TelegramBot(
    token="TU_BOT_TOKEN",
    sheets_lib=sheets,
    ranges=RANGES,
    debug=False
)

# Ejecutar bot
await bot.run(GROUP_ID, DEVELOPER_ID)
```

## 📱 Comandos Disponibles

### /pagado
Marca una factura como pagada respondiendo al mensaje del bot:

```
Usuario: /pagado (respondiendo a mensaje de factura)
Bot: ✅ Factura F004-01098027 de S/ 481.44 actualizada como PAGADA.
```

### /stats
Muestra estadísticas completas del sistema:

```
📊 **Estadísticas de Facturas**

💰 **Estado de Facturas:**
• Total empresas: 45
• Activas (pendientes): 12
• Pagadas: 33

🔗 **Mapeo de Mensajes:**
• Mensajes mapeados: 156

🧹 **Sistema:**
• Cache activo: ✅
• Datos en cache: 45
• Última sincronización: 2025-09-09 14:30:25
```

## 🔍 Sistema de Logs

### Uso Básico de Logs

```python
# Crear bot
bot = TelegramBot(token, sheets, ranges)

# Obtener logs programáticamente
logs = bot.get_logs()  # Todos los logs
logs_recientes = bot.get_logs(10)  # Últimos 10

# Mostrar logs en consola
bot.print_logs()  # Últimos 10
bot.print_logs(20)  # Últimos 20

# Limpiar logs
bot.clear_logs()

# Los logs se generan automáticamente durante operaciones
```

### Ejemplo de Monitoreo en main.py

```python
def monitorear_bot(bot):
    \"\"\"Función para monitorear el bot y mostrar logs\"\"\"
    while True:
        time.sleep(60)  # Cada minuto
        
        print("\\n--- LOGS DEL BOT ---")
        bot.print_logs(5)  # Últimos 5 logs
        
        # Verificar errores
        logs = bot.get_logs()
        errores = [log for log in logs if "❌" in log]
        if errores:
            print(f"⚠️ {len(errores)} errores en el bot")
            for error in errores[-3:]:  # Últimos 3 errores
                print(error)

# En tu main.py
import threading
monitor_thread = threading.Thread(target=monitorear_bot, args=(bot,), daemon=True)
monitor_thread.start()
```

## 🛠️ Funciones de Conveniencia

El bot incluye funciones para acceder fácilmente a los datos desde main.py:

```python
# Obtener datos de empresas
todas_empresas = bot.obtener_todas_las_empresas()
pendientes = bot.obtener_empresas_pendientes() 
stats = bot.obtener_estadisticas_completas()

# Forzar actualización de datos
bot.forzar_actualizacion_datos()

# Mostrar resumen en consola
bot.mostrar_resumen_empresas(solo_pendientes=True)
bot.mostrar_logs_recientes(15)
```

### Ejemplo Completo en main.py

```python
async def main():
    # Crear bot
    bot = TelegramBot(BOT_TOKEN, sheets, RANGES)
    
    # Mostrar estadísticas iniciales
    print("📊 Estadísticas iniciales:")
    stats = bot.obtener_estadisticas_completas()
    print(f"📈 Total empresas: {stats.get('total_empresas', 0)}")
    print(f"📋 Pendientes: {stats.get('facturas_pendientes', 0)}")
    
    # Mostrar resumen de empresas pendientes
    bot.mostrar_resumen_empresas(solo_pendientes=True)
    
    # Función de monitoreo en thread separado
    def monitoreo_periodico():
        while True:
            time.sleep(300)  # Cada 5 minutos
            bot.forzar_actualizacion_datos()
            print("\\n🔄 Datos actualizados automáticamente")
            bot.print_logs(3)
    
    threading.Thread(target=monitoreo_periodico, daemon=True).start()
    
    # Ejecutar bot
    await bot.run(GROUP_ID, DEVELOPER_ID)
```

## 📨 Formato de Mensajes

El bot envía facturas con formato elegante:

```
🏢 **DIMERC PERU S.A.C**
📄 F004-01098027
💰 S/ 481.44
📅 Vence: 10/09/2025
🔴 PENDIENTE
```

## ⚙️ Configuración

### Variables de Entorno Requeridas

```bash
BOT_TOKEN=tu_token_de_telegram
ALLOWED_GROUP_ID=id_del_grupo_autorizado  
DEVELOPER_ID=tu_id_de_desarrollador
```

### Parámetros del Constructor

```python
TelegramBot(
    token="BOT_TOKEN",           # Token del bot
    sheets_lib=sheets_reader,    # Instancia de GoogleSheetsReader
    ranges=RANGES,               # Diccionario de rangos
    debug=False                  # Modo debug (opcional)
)
```

## 🔐 Seguridad

- ✅ **Filtros de chat**: Solo responde en chats autorizados
- ✅ **Control de acceso**: GROUP_ID y DEVELOPER_ID configurables
- ✅ **Logs silenciosos**: No genera archivos de log externos
- ✅ **Mapeo seguro**: Vincula mensajes con facturas específicas

## 🔄 Flujo de Trabajo

1. **Inicio**: Bot carga datos desde Google Sheets
2. **Envío**: Envía facturas próximas a vencer automáticamente
3. **Mapeo**: Guarda relación mensaje_id ↔ factura_id
4. **Escucha**: Queda escuchando comandos /pagado y /stats
5. **Actualización**: Cuando recibe /pagado, actualiza Google Sheets

## 📊 EmpresaDataManager Integrado

El bot usa automáticamente el sistema de gestión de datos:

```python
# El bot accede automáticamente a:
bot.data_manager.obtener_todas_las_empresas()
bot.data_manager.actualizar_estado_factura(id, pagado=True)
bot.data_manager.obtener_proximos_vencimientos(dias=5)
bot.data_manager.guardar_mapeo_mensaje(msg_id, factura_id)
```

## 🐛 Debugging y Logs

### Tipos de Logs Automáticos

- 🚀 **Inicio**: Arranque del bot
- 📤 **Envíos**: Mensajes enviados al grupo
- 💰 **Pagos**: Facturas marcadas como pagadas
- 🔍 **Búsquedas**: Operaciones de lectura de datos
- ❌ **Errores**: Problemas durante la ejecución
- 📊 **Estadísticas**: Solicitudes de /stats

### Ejemplo de Logs

```
[14:30:25] 🚀 Iniciando bot de forma autónoma...
[14:30:26] 🔄 Datos cargados desde Google Sheets
[14:30:27] ⏰ Facturas próximas a vencer: 3
[14:30:28] 📤 Enviando factura 1/3: F004-01098027
[14:30:30] ✅ Mensaje enviado, ID: 123 -> Factura ID: 50
[14:35:15] 💰 Comando /pagado ejecutado por chat -123456789
[14:35:16] ✅ Factura F004-01098027 actualizada correctamente
```

## ⚠️ Notas Importantes

1. **Dependencias**: Requiere GoogleSheetsReader funcionando
2. **Permisos**: El bot necesita permisos para enviar mensajes
3. **Mapeo**: Se guarda automáticamente en bot_data.json
4. **Cache**: Los datos se sincronizan automáticamente
5. **Hilos**: El bot corre en el hilo principal, usa threads para monitoreo

## 🔧 Funciones Principales

| Función | Descripción |
|---------|------------|
| `run()` | Ejecuta el bot principal |
| `enviar_vencimientos_proximos()` | Envía facturas próximas a vencer |
| `cmd_pagado()` | Maneja comando /pagado |
| `cmd_stats()` | Maneja comando /stats |
| `obtener_todas_las_empresas()` | Acceso rápido a datos |
| `mostrar_resumen_empresas()` | Muestra resumen en consola |

## 🎯 Ejemplo de Integración Completa

```python
# main.py completo
import asyncio
import threading
import time

async def main():
    # Setup
    sheets = GoogleSheetsReader("key.json", SHEET_ID, RANGES)
    bot = TelegramBot(BOT_TOKEN, sheets, RANGES)
    
    # Monitoreo en background
    def monitor():
        while True:
            time.sleep(180)  # Cada 3 minutos
            print("\\n=== ESTADO DEL BOT ===")
            bot.print_logs(5)
            stats = bot.obtener_estadisticas_completas()
            print(f"Pendientes: {stats.get('facturas_pendientes', 0)}")
    
    threading.Thread(target=monitor, daemon=True).start()
    
    # Ejecutar bot
    await bot.run(GROUP_ID, DEVELOPER_ID)

if __name__ == '__main__':
    asyncio.run(main())
```

# 📊 Google Sheets Final Library

Una librería unificada y optimizada para interactuar con Google Sheets, diseñada específicamente para trabajar con el sistema de gestión de facturas del Telegram Bot. Incluye un sistema de logging en tiempo real similar al de `telegram_bot_lib`.

## 🚀 Características Principales

- ✅ **Lectura de datos** vía CSV y API oficial de Google
- ✅ **Actualización de celdas** (marcar facturas como pagadas)
- ✅ **Filtrado inteligente** de vencimientos próximos
- ✅ **Sistema de logging en memoria** sin archivos
- ✅ **Cache inteligente** para optimizar rendimiento
- ✅ **Validación robusta** de datos
- ✅ **Compatibilidad total** con telegram_bot_lib

## 📦 Instalación de Dependencias

```bash
# Dependencias requeridas
pip install google-api-python-client google-auth-httplib2 google-auth-oauthlib

# Dependencias opcionales (para mejor rendimiento)
pip install urllib3
```

## 🔧 Uso Básico

### Inicialización

```python
from modulos.google_sheets_final import GoogleSheetsReader

# Crear instancia con credenciales
reader = GoogleSheetsReader("key.json", "tu_sheet_id")

# O usar el SHEET_ID por defecto
reader = GoogleSheetsReader("key.json")
```

### Operaciones Principales

```python
# Leer columnas específicas
ranges = {
    'C': 'Hoja 1!C2:C',
    'D': 'Hoja 1!D2:D', 
    'H': 'Hoja 1!H2:H',
    'J': 'Hoja 1!J2:J',
    'MONTO': 'Hoja 1!F2:F'
}

rows = reader.read_columns(
    ranges['C'], ranges['D'], ranges['H'], 
    ranges['J'], ranges['MONTO']
)

# Filtrar facturas próximas a vencer (5 días)
proximos = reader.filtrar_proximos_vencimientos(rows, dias=5)

# Formatear datos para mostrar
mensaje = reader.format_rows_bonito(proximos)

# Actualizar estado de factura
reader.actualizar_pagado(fila_id=5)
```

## 📝 Sistema de Logging en Tiempo Real

### Configuración Básica

La librería incluye un sistema de logging en memoria que **NO crea archivos**, similar al de `telegram_bot_lib`:

```python
from modulos.google_sheets_final import (
    GoogleSheetsReader,
    get_google_sheets_logs,
    print_google_sheets_logs,
    clear_google_sheets_logs,
    log_google_sheets_info
)
```

### 🔥 Uso del Sistema de Logs en main.py

#### 1. Obtener Logs en Tiempo Real

```python
# En tu main.py
import time
import threading
from modulos.google_sheets_final import get_google_sheets_logs, print_google_sheets_logs

def monitor_google_sheets_logs():
    """Mostrar logs de Google Sheets en tiempo real"""
    while True:
        try:
            # Obtener últimos 5 logs
            logs = get_google_sheets_logs(5)
            
            if logs:
                print("\n🔍 Google Sheets Activity:")
                for log in logs:
                    print(f"  {log}")
                print("-" * 50)
            
            time.sleep(10)  # Actualizar cada 10 segundos
            
        except Exception as e:
            print(f"Error monitoreando logs: {e}")
            time.sleep(5)

# Iniciar monitoreo en hilo separado
log_thread = threading.Thread(target=monitor_google_sheets_logs, daemon=True)
log_thread.start()
```

#### 2. Integración con Terminal Logs

```python
# En tu main.py
from modulos.terminal_libs import terminal
from modulos.google_sheets_final import get_google_sheets_logs

def show_combined_logs():
    """Mostrar logs combinados de Telegram Bot y Google Sheets"""
    
    # Logs del Bot de Telegram
    bot_logs = telegram_bot.get_logs(5)
    
    # Logs de Google Sheets
    sheets_logs = get_google_sheets_logs(5)
    
    print("\n📊 === ESTADO DEL SISTEMA ===")
    
    print("\n🤖 Telegram Bot:")
    for log in bot_logs:
        print(f"  {log}")
    
    print("\n📋 Google Sheets:")
    for log in sheets_logs:
        print(f"  {log}")
    
    print("=" * 50)

# Llamar cada 30 segundos
def periodic_status():
    while True:
        time.sleep(30)
        show_combined_logs()

status_thread = threading.Thread(target=periodic_status, daemon=True)
status_thread.start()
```

#### 3. Logging Manual desde main.py

```python
from modulos.google_sheets_final import (
    log_google_sheets_info,
    log_google_sheets_debug,
    log_google_sheets_warning,
    log_google_sheets_error
)

# Agregar logs manuales
log_google_sheets_info("Sistema iniciado desde main.py")
log_google_sheets_debug("Verificando conexión con Google Sheets")
log_google_sheets_warning("Reintentando conexión...")
log_google_sheets_error("Error crítico en lectura de datos")
```

### 🎯 Tipos de Logs Disponibles

| Emoji | Tipo | Descripción |
|-------|------|-------------|
| ℹ️ | Info | Información general del sistema |
| 🔍 | Debug | Información detallada (solo si GOOGLE_SHEETS_VERBOSE=true) |
| ⚠️ | Warning | Advertencias y problemas menores |
| ❌ | Error | Errores críticos |
| 📥 | Data Input | Inicio de lectura de datos |
| 📊 | Data Processing | Procesamiento de información |
| ✅ | Success | Operaciones exitosas |
| 💾 | Update | Actualizaciones de datos |
| 🔄 | Retry | Reintentos de operaciones |
| 🧹 | Cleanup | Limpieza de cache/logs |
| ⏰ | Time | Operaciones relacionadas con fechas |
| 🎯 | Filter | Filtrado de datos |
| 🎨 | Format | Formateo de salida |

### 📊 Dashboard de Logs en Tiempo Real

```python
# Ejemplo completo para main.py
import os
import time
import threading
from datetime import datetime
from modulos.google_sheets_final import (
    GoogleSheetsReader, 
    get_google_sheets_logs, 
    print_google_sheets_logs,
    log_google_sheets_info
)

def create_sheets_dashboard():
    """Crear dashboard de monitoreo en tiempo real"""
    
    def update_dashboard():
        while True:
            try:
                os.system('cls' if os.name == 'nt' else 'clear')  # Limpiar pantalla
                
                print("┌" + "─" * 60 + "┐")
                print("│" + " 📊 GOOGLE SHEETS DASHBOARD ".center(60) + "│")
                print("│" + f" {datetime.now().strftime('%Y-%m-%d %H:%M:%S')} ".center(60) + "│")
                print("├" + "─" * 60 + "┤")
                
                # Mostrar últimos logs
                logs = get_google_sheets_logs(10)
                if logs:
                    for log in logs[-8:]:  # Últimos 8 logs
                        print(f"│ {log:<58} │")
                else:
                    print("│" + " No hay logs disponibles ".center(60) + "│")
                
                print("└" + "─" * 60 + "┘")
                
                time.sleep(5)  # Actualizar cada 5 segundos
                
            except KeyboardInterrupt:
                break
            except Exception as e:
                print(f"Error en dashboard: {e}")
                time.sleep(5)
    
    dashboard_thread = threading.Thread(target=update_dashboard, daemon=True)
    dashboard_thread.start()
    return dashboard_thread

# En tu función main()
async def main():
    # ... tu código existente ...
    
    # Inicializar Google Sheets
    sheets_reader = GoogleSheetsReader("key.json", SHEET_ID)
    log_google_sheets_info("Sistema Google Sheets inicializado desde main.py")
    
    # Crear dashboard (opcional)
    # dashboard = create_sheets_dashboard()
    
    # ... resto de tu código ...
```

## 🛠️ Funciones del Sistema de Logs

### Funciones de Consulta

```python
# Obtener logs (sin mostrar en pantalla)
all_logs = get_google_sheets_logs()           # Todos los logs
recent_logs = get_google_sheets_logs(10)      # Últimos 10 logs

# Mostrar logs en consola
print_google_sheets_logs()                    # Últimos 10 logs
print_google_sheets_logs(20)                  # Últimos 20 logs

# Limpiar logs
clear_google_sheets_logs()
```

### Funciones de Logging Manual

```python
# Diferentes niveles de log
log_google_sheets_info("Operación completada")
log_google_sheets_debug("Detalles de debug")  
log_google_sheets_warning("Advertencia del sistema")
log_google_sheets_error("Error encontrado")
```

### Logging Avanzado con Variables de Entorno

```python
# Activar logging verbose
os.environ['GOOGLE_SHEETS_VERBOSE'] = 'true'

# Ahora se mostrarán logs de debug
reader = GoogleSheetsReader("key.json", sheet_id)
# Los logs de debug ahora aparecerán automáticamente
```

## 🔧 Configuración Avanzada

### Variables de Entorno

```bash
# Activar logging detallado
export GOOGLE_SHEETS_VERBOSE=true

# O en Windows
set GOOGLE_SHEETS_VERBOSE=true
```

### Personalización del Logger

```python
# Acceso directo al logger interno
from modulos.google_sheets_final import LOGGER

# Configurar tamaño máximo de logs
LOGGER.max_logs = 200

# Log personalizado
LOGGER.log("🚀", "Mi mensaje personalizado")
```

## 📋 Métodos de la Clase GoogleSheetsReader

### Métodos Principales

```python
reader = GoogleSheetsReader("key.json", "sheet_id")

# Lectura de datos
rows = reader.read_columns(range_c, range_d, range_h, range_j, range_monto)

# Filtrado
proximos = reader.filtrar_proximos_vencimientos(rows, dias=5)
pendientes = reader.filtrar_pendientes(rows)

# Formateo
mensaje = reader.format_rows_bonito(rows)

# Actualización
resultado = reader.actualizar_pagado(fila_id, sheet_name='Hoja 1', col_j='J')
```

### Métodos de Logging

```python
# Logging directo desde la instancia
reader.log("🔥", "Mi mensaje personalizado")

# Obtener logs de esta instancia
logs = reader.get_logs(10)

# Mostrar logs
reader.print_logs(15)

# Limpiar logs
reader.clear_logs()
```

## 🚨 Manejo de Errores

La librería incluye manejo robusto de errores:

```python
try:
    reader = GoogleSheetsReader("key.json", "sheet_id")
    rows = reader.read_columns(range_c, range_d, range_h)
    
except GoogleSheetsConnectionError as e:
    log_google_sheets_error(f"Error de conexión: {e}")
    
except GoogleSheetsAuthError as e:
    log_google_sheets_error(f"Error de autenticación: {e}")
    
except GoogleSheetsDataError as e:
    log_google_sheets_warning(f"Error en datos: {e}")
    
except Exception as e:
    log_google_sheets_error(f"Error general: {e}")
```

## 💡 Mejores Prácticas

### 1. Monitoreo Continuo

```python
def setup_continuous_monitoring():
    """Configurar monitoreo continuo de Google Sheets"""
    
    def log_monitor():
        last_log_count = 0
        
        while True:
            current_logs = get_google_sheets_logs()
            
            if len(current_logs) > last_log_count:
                # Nuevos logs disponibles
                new_logs = current_logs[last_log_count:]
                
                for log in new_logs:
                    # Procesar nuevos logs
                    if "❌" in log:
                        terminal.log_error(f"Google Sheets Error: {log}")
                    elif "⚠️" in log:
                        terminal.log_warning(f"Google Sheets Warning: {log}")
                    else:
                        terminal.log_info(f"Google Sheets: {log}")
                
                last_log_count = len(current_logs)
            
            time.sleep(2)  # Verificar cada 2 segundos
    
    monitor_thread = threading.Thread(target=log_monitor, daemon=True)
    monitor_thread.start()
    return monitor_thread
```

### 2. Integración con Sistema de Alertas

```python
def setup_alert_system():
    """Sistema de alertas basado en logs"""
    
    def check_alerts():
        while True:
            logs = get_google_sheets_logs(5)
            
            for log in logs:
                if "❌" in log and "Error crítico" in log:
                    # Enviar alerta crítica
                    terminal.log_error("🚨 ALERTA CRÍTICA EN GOOGLE SHEETS")
                    
                elif "⚠️" in log and "no pudieron ser parseadas" in log:
                    # Alerta de datos
                    terminal.log_warning("📊 Problemas con datos de Google Sheets")
            
            time.sleep(30)
    
    alert_thread = threading.Thread(target=check_alerts, daemon=True)
    alert_thread.start()
```

### 3. Logging Estructurado

```python
def structured_logging_example():
    """Ejemplo de logging estructurado"""
    
    # Log de inicio de operación
    log_google_sheets_info("🚀 Iniciando lectura de facturas")
    
    try:
        reader = GoogleSheetsReader("key.json", sheet_id)
        
        # Log de progreso
        log_google_sheets_info("📊 Conectado a Google Sheets")
        
        rows = reader.read_columns(range_c, range_d, range_h, range_j, range_monto)
        
        # Log de resultados
        log_google_sheets_info(f"✅ Leídas {len(rows)} facturas exitosamente")
        
        proximos = reader.filtrar_proximos_vencimientos(rows, dias=5)
        
        # Log final
        log_google_sheets_info(f"🎯 Encontradas {len(proximos)} facturas próximas a vencer")
        
    except Exception as e:
        log_google_sheets_error(f"❌ Error en operación: {e}")
```

## 🔗 Integración con Telegram Bot

El sistema está diseñado para trabajar perfectamente con `telegram_bot_lib`:

```python
# En telegram_bot_lib.py ya no necesitas importar Terminal
# Solo usa la instancia de GoogleSheetsReader que le pases

# En main.py
sheets_reader = GoogleSheetsReader("key.json", SHEET_ID)
telegram_bot = TelegramBot(BOT_TOKEN, sheets_reader, RANGES)

# Los logs de ambos sistemas estarán disponibles por separado
bot_logs = telegram_bot.get_logs(10)
sheets_logs = get_google_sheets_logs(10)
```

## 📈 Rendimiento y Optimización

- **Cache automático**: Los datos se cachean por 5 minutos
- **Reintentos inteligentes**: Sistema de reintentos exponencial
- **Logs limitados**: Máximo 100 logs en memoria (configurable)
- **Sin archivos**: Todo en memoria para máximo rendimiento

---

## 📞 Soporte

Para problemas o preguntas sobre el sistema de logging:

1. Verifica los logs con `print_google_sheets_logs()`
2. Activa modo verbose: `os.environ['GOOGLE_SHEETS_VERBOSE'] = 'true'`
3. Revisa los errores en tiempo real con el dashboard
4. Usa `clear_google_sheets_logs()` si los logs se llenan

**La librería está optimizada para trabajar sin interrupciones con tu sistema de gestión de facturas.**

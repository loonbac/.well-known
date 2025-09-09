# 📊 Google Sheets Final - Sistema de Gestión de Facturas

## 🎯 ¿Qué hace este módulo?

Este módulo proporciona una interfaz completa para leer y actualizar datos de Google Sheets, específicamente diseñado para gestionar facturas de empresas. Incluye:

- ✅ **Lectura de datos** desde Google Sheets (vía API oficial y CSV como respaldo)
- ✅ **Actualización de estados** de facturas (marcar como PAGADO)
- ✅ **Gestión de datos persistentes** en archivo JSON local
- ✅ **Sistema de logs interno** sin archivos externos
- ✅ **Validación y filtrado** de datos automático
- ✅ **Cache inteligente** para optimizar rendimiento

## 🚀 Uso Básico

```python
from modulos.google_sheets_final import GoogleSheetsReader

# Configurar RANGES específicos para tu hoja
RANGES = {
    'C': 'C2:C',     # Nombres de empresas
    'D': 'D2:D',     # Números de factura
    'H': 'H2:H',     # Fechas de vencimiento
    'J': 'J2:J',     # Estados de pago
    'MONTO': 'F2:F'  # Montos
}

# Crear instancia
sheets = GoogleSheetsReader("key.json", "TU_SHEET_ID", RANGES)

# Leer datos
datos = sheets.read_columns(
    RANGES['C'], RANGES['D'], RANGES['H'], 
    RANGES['J'], RANGES['MONTO']
)

# Formatear para mostrar
mensaje_bonito = sheets.format_rows_bonito(datos)
print(mensaje_bonito)
```

## 📋 EmpresaDataManager - Gestión Avanzada

El módulo incluye un gestor de datos integrado que sincroniza automáticamente con Google Sheets:

```python
# Acceder al data manager
data_manager = sheets.data_manager

# Sincronizar con Google Sheets
data_manager.sincronizar_con_sheets(forzar=True)

# Obtener todas las empresas
empresas = data_manager.obtener_todas_las_empresas()

# Obtener solo facturas pendientes
pendientes = data_manager.obtener_empresas_pendientes()

# Obtener facturas próximas a vencer (5 días por defecto)
proximas = data_manager.obtener_proximos_vencimientos(dias=7)

# Actualizar estado de factura
data_manager.actualizar_estado_factura("50", pagado=True)

# Obtener estadísticas completas
stats = data_manager.obtener_estadisticas()
```

## 💾 Funciones de Actualización Automática de bot_data.json

El sistema actualiza automáticamente el archivo `bot_data.json` que contiene todos los datos persistentes:

### Funciones Principales de Actualización

```python
# 1. Sincronización Completa
data_manager.sincronizar_con_sheets(forzar=True)
# - Lee todos los datos desde Google Sheets
# - Actualiza bot_data.json con empresas completas
# - Recalcula estadísticas automáticamente
# - Mantiene mapeos de mensajes existentes

# 2. Forzar Actualización Total
data_manager.forzar_actualizacion_completa()
# - Limpia cache interno
# - Fuerza nueva lectura desde Google Sheets
# - Actualiza bot_data.json completamente
# - Ideal para resolver inconsistencias

# 3. Actualizar Estado de Factura
data_manager.actualizar_estado_factura("50", pagado=True)
# - Actualiza Google Sheets (si tiene permisos)
# - Actualiza cache local inmediatamente
# - Actualiza bot_data.json automáticamente
# - Recalcula estadísticas de pendientes/pagadas

# 4. Guardar Mapeo de Mensajes
data_manager.guardar_mapeo_mensaje(message_id, factura_id)
# - Mapea message_id del bot con factura_id
# - Actualiza bot_data.json inmediatamente
# - Esencial para comando /pagado del bot
```

### Estructura del bot_data.json

```json
{
  "metadata": {
    "created_at": "2025-09-09T14:30:25",
    "last_updated": "2025-09-09T15:45:12", 
    "version": "3.0",
    "description": "Datos completos de empresas sincronizados desde Google Sheets"
  },
  "empresas": {
    "50": {
      "id": 50,
      "cliente": "DIMERC PERU S.A.C",
      "factura": "F004-01098027",
      "monto": "S/ 481.44",
      "fecha_vencimiento": "10/09/2025",
      "estado": "PAGADO",
      "pagado": true,
      "fila_excel": 50,
      "ultima_sincronizacion": "2025-09-09T15:45:12"
    }
  },
  "msgid_to_facturaid": {
    "123": "50",
    "124": "51"
  },
  "estadisticas": {
    "empresas_totales": 45,
    "empresas_pendientes": 12,
    "empresas_pagadas": 33,
    "mensajes_mapeados": 156,
    "ultima_sincronizacion": "2025-09-09T15:45:12"
  }
}
```

### Funciones de Gestión de Datos

```python
# Cargar datos completos al inicializar
data_manager.cargar_datos_completos()
# - Carga empresas y mapeos desde bot_data.json
# - Se ejecuta automáticamente al crear la instancia

# Cargar solo mapeos de mensajes
data_manager.cargar_mapeo_mensajes()
# - Carga solo msgid_to_facturaid desde JSON
# - Útil para el bot de Telegram

# Obtener factura por mensaje
factura_id = data_manager.obtener_factura_por_mensaje(message_id)
# - Busca en mapeos cargados en memoria
# - Retorna None si no encuentra el mapeo

# Limpiar cache y forzar actualización
data_manager.limpiar_cache()
data_manager.forzar_actualizacion_completa()
# - Limpia datos en memoria
# - Fuerza nueva lectura desde Google Sheets
# - Actualiza bot_data.json completamente
```

### Ejemplo de Uso en main.py

```python
def actualizar_sistema_completo():
    """Función para actualizar todo el sistema de datos"""
    
    # 1. Forzar actualización desde Google Sheets
    print("🔄 Actualizando desde Google Sheets...")
    exito = data_manager.forzar_actualizacion_completa()
    
    if exito:
        print("✅ Datos actualizados correctamente")
        
        # 2. Verificar estadísticas actualizadas
        stats = data_manager.obtener_estadisticas()
        print(f"📊 Total empresas: {stats['total_empresas']}")
        print(f"📋 Pendientes: {stats['facturas_pendientes']}")
        print(f"✅ Pagadas: {stats['facturas_pagadas']}")
        
        # 3. Verificar que bot_data.json se actualizó
        with open('bot_data.json', 'r') as f:
            data = json.load(f)
            ultima_sync = data['metadata']['last_updated']
            print(f"📅 Última sincronización: {ultima_sync}")
            
    else:
        print("❌ Error actualizando datos")

# Automatizar actualización cada 10 minutos
import threading
import time

def actualizacion_periodica():
    while True:
        time.sleep(600)  # 10 minutos
        actualizar_sistema_completo()

# Iniciar en thread separado
threading.Thread(target=actualizacion_periodica, daemon=True).start()
```

### Funciones de Monitoreo de Datos

```python
# Verificar integridad de datos
def verificar_integridad_datos():
    # Verificar que bot_data.json existe y es válido
    if os.path.exists('bot_data.json'):
        data = data_manager.load_data()
        print(f"✅ bot_data.json válido: {len(data.get('empresas', {}))} empresas")
        return True
    else:
        print("❌ bot_data.json no encontrado")
        return False

# Comparar datos locales vs Google Sheets
def comparar_datos():
    # Datos locales
    empresas_locales = data_manager.obtener_todas_las_empresas()
    
    # Forzar lectura desde Google Sheets
    data_manager.forzar_actualizacion_completa()
    empresas_sheets = data_manager.obtener_todas_las_empresas()
    
    diferencias = len(empresas_sheets) - len(empresas_locales)
    print(f"📊 Diferencia de registros: {diferencias}")
    
    return diferencias == 0

# Usar en main.py para monitoreo
if __name__ == '__main__':
    if verificar_integridad_datos():
        if comparar_datos():
            print("✅ Datos locales y remotos sincronizados")
        else:
            print("⚠️ Datos desincronizados, actualizando...")
            actualizar_sistema_completo()
```

## 🔍 Sistema de Logs

### Uso Básico de Logs

```python
from modulos.google_sheets_final import GoogleSheetsReader
from modulos.google_sheets_final import (
    get_google_sheets_logs,
    print_google_sheets_logs, 
    clear_google_sheets_logs,
    log_google_sheets_info,
    log_google_sheets_debug,
    log_google_sheets_warning,
    log_google_sheets_error
)

# Crear instancia
sheets = GoogleSheetsReader("key.json", "SHEET_ID", RANGES)

# Obtener logs programáticamente
logs = get_google_sheets_logs()  # Todos los logs
logs_recientes = get_google_sheets_logs(10)  # Últimos 10

# Mostrar logs en consola
print_google_sheets_logs()  # Últimos 10
print_google_sheets_logs(20)  # Últimos 20

# Limpiar logs
clear_google_sheets_logs()

# Agregar logs personalizados
log_google_sheets_info("Mi mensaje informativo")
log_google_sheets_debug("Mensaje de debug")
log_google_sheets_warning("Mensaje de advertencia")
log_google_sheets_error("Mensaje de error")
```

### Logs desde la Instancia

```python
# También puedes usar logs directamente desde la instancia
sheets.log("🔍", "Mi mensaje personalizado")
sheets.print_logs(15)  # Mostrar últimos 15 logs
sheets.clear_logs()    # Limpiar logs
logs = sheets.get_logs(5)  # Obtener últimos 5 logs
```

### Ejemplo Completo en main.py

```python
# En tu main.py
from modulos.google_sheets_final import (
    GoogleSheetsReader,
    print_google_sheets_logs,
    log_google_sheets_info
)

def main():
    # Crear instancia
    sheets = GoogleSheetsReader("key.json", SHEET_ID, RANGES)
    
    # Log personalizado
    log_google_sheets_info("Iniciando procesamiento de facturas")
    
    # Hacer operaciones
    datos = sheets.read_columns(RANGES['C'], RANGES['D'], RANGES['H'])
    
    # Mostrar logs en cualquier momento
    print("\\n--- LOGS DE GOOGLE SHEETS ---")
    print_google_sheets_logs(10)
    
    # Verificar si hay errores en los logs
    logs = sheets.get_logs()
    errores = [log for log in logs if "❌" in log]
    if errores:
        print(f"⚠️ Se encontraron {len(errores)} errores en los logs")
```

## 🎨 Formato de Mensajes

El método `format_rows_bonito()` genera mensajes con emojis y formato limpio:

```
🏢 **DIMERC PERU S.A.C**
📄 F004-01098027
💰 S/ 481.44
📅 Vence: 10/09/2025
🔴 PENDIENTE
```

## ⚙️ Configuración Avanzada

### Variables de Entorno

```bash
# Habilitar logs detallados (opcional)
GOOGLE_SHEETS_VERBOSE=true
```

### Configuración de Cache

```python
# El cache se maneja automáticamente
# Duración por defecto: 5 minutos
# Para forzar actualización:
data_manager.forzar_actualizacion_completa()
```

## 🔧 Funciones Principales

| Función | Descripción |
|---------|------------|
| `read_columns()` | Lee columnas específicas desde Google Sheets |
| `format_rows_bonito()` | Formatea datos para mostrar con emojis |
| `actualizar_pagado()` | Actualiza estado de factura a PAGADO |
| `filtrar_proximos_vencimientos()` | Filtra facturas próximas a vencer |
| `filtrar_pendientes()` | Filtra facturas no vencidas |

## 📁 Archivos Generados

- `bot_data.json` - Datos persistentes y cache local
- `key.json` - Credenciales de Google Sheets (no incluir en git)

## ⚠️ Notas Importantes

1. **Credenciales**: Asegúrate de que `key.json` tenga permisos de lectura Y escritura
2. **SHEET_ID**: Debe ser el ID correcto de tu Google Sheet
3. **RANGES**: Configura los rangos según tu estructura de datos
4. **Cache**: Los datos se cachean automáticamente por 5 minutos
5. **Logs**: Se mantienen en memoria (máximo 100 entradas)

## 🐛 Debugging

```python
# Para debuggear problemas:
import os
os.environ['GOOGLE_SHEETS_VERBOSE'] = 'true'

# Crear instancia y revisar logs
sheets = GoogleSheetsReader("key.json", SHEET_ID, RANGES)
sheets.print_logs()  # Ver todos los logs de debug
```

# 📋 Especificación Técnica: GoogleSheetsReader

## 📖 Descripción General

Debes crear una clase `GoogleSheetsReader` que maneje la lectura y escritura de datos de Google Sheets para un sistema de gestión de facturas. La clase debe ser **100% compatible** con el sistema existente.

## 🏗️ Arquitectura Requerida

### Clase Principal
```python
class GoogleSheetsReader:
    def __init__(self, creds_path, sheet_id):
        # Tu implementación aquí
        pass
```

### Dependencias Recomendadas
- `google-api-python-client>=2.0.0`
- `google-auth>=2.0.0` 
- `google-auth-oauthlib>=0.5.0`
- `google-auth-httplib2>=0.1.0`

## 🔧 Métodos Obligatorios

### 1. `__init__(self, creds_path, sheet_id)`

**Propósito:** Inicializar la conexión con Google Sheets

**Parámetros:**
- `creds_path` (str): Ruta al archivo JSON de credenciales de Google Service Account
- `sheet_id` (str): ID de la hoja de Google Sheets

**Requisitos:**
- Establecer conexión con Google Sheets API
- Configurar autenticación usando Service Account
- Implementar sistema de logging interno silencioso
- NO debe imprimir warnings ni logs de Google APIs

### 2. `read_columns(self, range_c, range_d, range_h, range_j=None, range_monto=None)`

**Propósito:** Leer datos de múltiples columnas y retornar estructura unificada

**Parámetros:**
- `range_c` (str): Rango de la columna C (nombres de empresas)
- `range_d` (str): Rango de la columna D (números de factura)
- `range_h` (str): Rango de la columna H (fechas de vencimiento)
- `range_j` (str, opcional): Rango de la columna J (estados de pago)
- `range_monto` (str, opcional): Rango de cualquier columna con montos

**Retorno Obligatorio:**
```python
[
    {
        'id': 2,                              # int (número de fila en Google Sheets)
        'nombre': 'EMPRESA S.A.C.',           # str (columna C)
        'factura': 'F012-00165807',           # str (columna D)
        'vencimiento': '6/09/2025',           # str (columna H)
        'estado': 'PAGADO',                   # str (columna J o '')
        'monto': '$155.96'                    # str (columna monto o '')
    },
    # ... más filas
]
```

**Comportamiento Crítico:**
- `id` DEBE ser `i+2` donde `i` es el índice del array (fila 2 = índice 0)
- Si una celda está vacía, usar string vacío `''`
- Si `range_j` o `range_monto` son `None`, esos campos deben ser `''`
- Retornar lista vacía `[]` en caso de error
- NO debe imprimir errores

### 3. `format_rows_bonito(self, rows)`

**Propósito:** Formatear lista de filas en texto legible para Telegram

**Parámetros:**
- `rows` (list): Lista de diccionarios con estructura de `read_columns`

**Retorno Obligatorio:**
```python
# Para UNA fila:
"ID: 9\nEmpresa: FIORELLA REPRESENTACIONES S.A.C.\nFactura: F012-00165807\nMonto: $155.96\nVencimiento: 6/09/2025\nEstado: PAGADO\n-------------------------"

# Para MÚLTIPLES filas (separadas por \n):
"ID: 9\nEmpresa: FIORELLA...\n-------------------------\nID: 22\nEmpresa: OTRA..."

# Si no hay datos:
"No hay datos."
```

**Formato Exacto:**
- Línea 1: `ID: {id}`
- Línea 2: `Empresa: {nombre}`
- Línea 3: `Factura: {factura}`
- Línea 4: `Monto: {monto}`
- Línea 5: `Vencimiento: {vencimiento}`
- Línea 6: `Estado: {estado}`
- Línea 7: `-------------------------` (25 guiones)

**Comportamiento:**
- Solo incluir filas que tengan al menos `nombre`, `factura` O `vencimiento`
- Usar `row.get('monto','')` y `row.get('estado','')` para campos opcionales

### 4. `actualizar_pagado(self, fila_id, sheet_name='Hoja 1', col_j='J')`

**Propósito:** Actualizar una celda específica con el valor "PAGADO"

**Parámetros:**
- `fila_id` (str/int): Número de fila en Google Sheets
- `sheet_name` (str): Nombre de la hoja (por defecto 'Hoja 1')
- `col_j` (str): Columna a actualizar (por defecto 'J')

**Comportamiento:**
- Actualizar celda en `{sheet_name}!{col_j}{fila_id}` con valor "PAGADO"
- Usar `valueInputOption='RAW'`
- Retornar resultado de la operación o `None` si falla
- NO debe imprimir errores

**Ejemplo:** `actualizar_pagado("9")` debe actualizar `Hoja 1!J9` con "PAGADO"

### 5. `filtrar_proximos_vencimientos(self, rows, dias=5)`

**Propósito:** Filtrar facturas que vencen en los próximos N días

**Parámetros:**
- `rows` (list): Lista de diccionarios de `read_columns`
- `dias` (int): Número de días hacia adelante (por defecto 5)

**Retorno:** Lista filtrada con mismo formato que `read_columns`

**Lógica Requerida:**
- Parsear `row['vencimiento']` usando estos formatos en orden:
  1. `"%d/%m/%Y"` (6/09/2025)
  2. `"%Y-%m-%d"` (2025-09-06)
  3. `"%d-%m-%Y"` (06-09-2025)  
  4. `"%d/%m/%y"` (6/09/25)
- Calcular diferencia: `(fecha_vencimiento - fecha_hoy).days`
- Incluir solo si: `0 <= diferencia_dias <= dias`
- Ignorar filas con fechas inválidas (no retornar error)

### 6. `filtrar_pendientes(self, rows)`

**Propósito:** Filtrar facturas que aún no han vencido (fecha >= hoy)

**Parámetros:**
- `rows` (list): Lista de diccionarios de `read_columns`

**Retorno:** Lista filtrada con mismo formato que `read_columns`

**Lógica Requerida:**
- Usar mismo parsing de fechas que `filtrar_proximos_vencimientos`
- Incluir solo si: `(fecha_vencimiento - fecha_hoy).days >= 0`
- Ignorar filas con fechas inválidas

## 🔒 Requisitos de Silenciamiento

**CRÍTICO:** La librería NO debe imprimir NADA en consola:

- ❌ Sin warnings de `file_cache is only supported with oauth2client<4.0.0`
- ❌ Sin logs de autenticación de Google
- ❌ Sin mensajes de error de HTTP
- ❌ Sin prints de debugging

**Métodos recomendados:**
- Configurar `warnings.filterwarnings('ignore')`
- Desactivar loggers de Google APIs
- Usar try/catch silenciosos

## 📊 Sistema de Logging Interno

**Opcional pero Recomendado:**

Implementar sistema interno de logging similar a:

```python
def log(self, emoji, message):
    # Guardar en memoria, NO imprimir
    pass

def get_logs(self, last_n=None):
    # Retornar logs para debugging
    pass

def print_logs(self, last_n=10):
    # Imprimir solo cuando se solicite explícitamente
    pass
```

## 🎯 Criterios de Éxito

Tu implementación será exitosa si:

1. ✅ Todos los métodos retornan datos en formato exacto especificado
2. ✅ `read_columns` genera IDs secuenciales empezando en 2
3. ✅ `format_rows_bonito` produce texto idéntico al esperado
4. ✅ `actualizar_pagado` modifica correctamente Google Sheets
5. ✅ Filtros de fechas funcionan con múltiples formatos
6. ✅ No imprime absolutamente nada en consola
7. ✅ Maneja errores graciosamente (retorna [] o None)
8. ✅ Pasa todos los tests de compatibilidad

## 🔧 Herramientas de Desarrollo

En esta carpeta encontrarás:
- `test_compatibility.py` - Tests automáticos de compatibilidad
- `sample_data.py` - Datos de ejemplo para testing
- `interface_contract.py` - Interfaz que debes implementar
- `validation_tool.py` - Herramienta de validación automática

## 🚀 Próximos Pasos

1. Implementa la clase `GoogleSheetsReader` con todos los métodos
2. Ejecuta `python test_compatibility.py` para verificar compatibilidad
3. Usa `python validation_tool.py` para validación continua
4. Refina hasta que pases todos los tests

¡Buena suerte! 🎉

# 📦 Inventario · Beta

App web (móvil) para **toma de inventario físico** a partir del archivo de stock del sistema.
Es un único archivo `index.html` (sin instalación, sin servidor, funciona offline). Todos los
datos se procesan **solo en el teléfono**.

> Versión **beta**. Pensada para probar el flujo completo antes de una versión definitiva.

## Probarla (GitHub Pages)

En **Settings → Pages** de este repo: *Deploy from a branch* → Branch **`main`** → Folder **`/ (root)`** → **Save**.
En ~1 minuto queda en:

```
https://migvelang.github.io/inventario/
```

Al servirse por **HTTPS**, la cámara funciona en el teléfono. Ábrela en el celular y guárdala en la
pantalla de inicio para usarla como app.

## Cómo se usa

1. **Cargar** el archivo `.txt`/`.csv` que exporta el sistema (separado por `;`, con las columnas
   `NUMBER`, `UPC`, `STOCK DISPONIBLE`, `CLASE`, `SUBCLASE`, etc.). En `ejemplo/` hay un archivo real de muestra.
2. **Elegir el alcance**: todo el archivo, o solo una **clase** o **subclase** en particular.
3. **Escanear / ingresar** códigos UPC/EAN o el SKU (`NUMBER`). Cada lectura suma al stock físico.
4. Revisar **diferencias en línea** y los **códigos no encontrados**.

## Qué hace

| Requisito | Estado |
|---|---|
| Cargar `.txt` y exportarlo a `.csv` (Excel) | ✅ Carga el `.txt`; exporta base, diferencias y no-encontrados a CSV |
| Ingresar UPC o SKU y sumarlos como stock físico | ✅ Campo de captura + cantidad |
| Mostrar los últimos 3 productos tomados y lo que falta | ✅ Tarjetas con físico / disponible / faltante |
| Alerta si hay excedente | ✅ Aviso rojo + sonido cuando físico > disponible |
| Escanear EAN/UPC con la cámara | ✅ `BarcodeDetector` (Android/Chrome) |
| Conectar un lector Bluetooth | ✅ Cualquier lector en modo **HID/teclado** escribe en el campo y suma |
| SKU con 2 EAN/UPC → sumar al que tenga stock | ✅ El conteo se agrega por `NUMBER`; muestra el UPC con stock |
| Código no reconocido → no sumar, alertar y revisar luego | ✅ Va a *Pendientes*; se confirma para pasar a *No encontrados* |
| Revisar diferencias en línea | ✅ Pestaña Diferencias con filtros |
| Elegir qué tomar (todo / clase / subclase) | ✅ Selectores de alcance |
| Historial de inventarios (pausar uno, continuar otro) | ✅ Cada archivo abre un inventario con nombre; se guardan todos |
| No perder el progreso al actualizar la página | ✅ Se restaura automáticamente el inventario activo |
| CSV "de todo" con el formato original + `STOCK FISICO` | ✅ Mismas columnas, `;`, CRLF, ISO-8859-1 y columna STOCK FISICO por fila |

## Historial de inventarios

Cada archivo que cargas abre un **inventario con nombre** (por defecto, nombre del archivo + fecha).
Todos quedan guardados en el dispositivo y aparecen en **Cargar → Historial de inventarios**, donde
puedes **Continuar** cualquiera, exportar su CSV o borrarlo. Si **actualizas o cierras la página**, al
volver se restaura solo el inventario activo con todo su avance (no hace falta volver a subir el archivo).

## Exportación CSV (formato del sistema)

El botón **Exportar CSV de todo** genera un archivo con **exactamente las mismas columnas** del archivo
original, separado por `;`, con saltos CRLF y codificación ISO-8859-1, agregando al final la columna
**`STOCK FISICO`** con el conteo por fila. En productos con dos UPC, el físico se asigna a la fila del
UPC que tiene stock disponible (las demás quedan en 0), de modo que la suma por `NUMBER` cuadra.

## Escaneo: cámara vs. Bluetooth

- **Lector Bluetooth (recomendado, universal):** la mayoría funcionan como **teclado HID**: emparejas,
  dejas el campo de código enfocado y el lector "escribe" el número y da Enter. Funciona en **cualquier
  teléfono** (Android e iPhone) sin código extra.
- **Cámara del teléfono (embebida, no pantalla completa):** el escáner se abre en un recuadro dentro de
  la pestaña Escanear, para seguir viendo lo tomado y **deshacer** sin cerrarlo. Cada lectura **suena**.
  - ✅ **Android / Chrome:** usa la API nativa `BarcodeDetector`.
  - ✅ **iPhone / Safari:** usa la librería **ZXing** (cargada desde CDN la primera vez); requiere Safari
    y aceptar el permiso de cámara.

## Pegar lista de códigos

Botón **📋 Pegar lista** en Escanear: pega muchos UPC/EAN o SKU (uno por línea o separados por espacio/
coma) y, **tras confirmar**, suma **1 a cada código**. Muestra cuántos se reconocieron y cuántos no.

## Compartir a otro dispositivo (Finalizar)

localStorage es por navegador/dispositivo, así que un inventario **no aparece solo** en otro teléfono.
Para pasarlo: **🏁 Finalizar inventario** y luego **🔗 compartir** (usa "Compartir" del teléfono —
AirDrop, WhatsApp, correo— o descarga un archivo `.json`). En el otro dispositivo, **⬆ Importar** ese
archivo y queda en su historial con las diferencias listas para revisar.

## Filtros de diferencias

La pestaña **Diferencias** combina los estados (Todos / Con diferencia / Faltantes / Excedentes / Sin
contar) con filtros por **marca**, **clase** y **subclase**.

## Modelo de datos (multi-EAN)

Un mismo `NUMBER` (SKU) puede tener **varios UPC/EAN** (p. ej. el del fabricante y uno interno que
empieza en `2000…`). El conteo se **agrega por `NUMBER`**: escanear cualquiera de sus códigos suma al
mismo producto, asociándolo al UPC que tiene stock disponible. El disponible comparado es la **suma** de
las variantes del SKU.

## Persistencia

El conteo, los pendientes y los no-encontrados se guardan automáticamente en el navegador
(`localStorage`); puedes cerrar la página o quedarte sin señal sin perder el avance. El botón
*Borrar conteo* reinicia todo.

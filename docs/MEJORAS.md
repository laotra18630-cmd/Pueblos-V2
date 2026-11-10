# Análisis del `index.html` original y mejoras aplicadas

Este documento resume el trabajo hecho sobre la app **¿Qué pueblo es?** (repositorio
`Pueblos-V2`). Se parte del `index.html` que estaba en la carpeta `Pueblos/` y se compara
con la versión modular de `Quepuebloes/`.

---

## 1. Estado del archivo analizado

`Pueblos/index.html` era la **versión autocontenida** de la app:

| Aspecto | Situación |
|---|---|
| Tamaño | 707 líneas en un solo archivo |
| CSS | ~235 líneas dentro de `<style>` |
| JavaScript | Todo el motor dentro de `<script>` |
| Datos | Bloque `#datosPueblos` embebido en el HTML (texto plano) |
| PWA | Referenciaba `manifest.json` **que no existía** |
| Service worker | **No había** (`sw.js` inexistente) |
| Iconos | Solo `icon.png` |
| Proveedores de relieve | Open-Meteo **y** OpenTopoData (sin CORS) |

### Problemas detectados

1. **Referencias rotas**: el manifiesto PWA apuntaba a `manifest.json`, que no existía en
   esa carpeta, así que la app no se podía instalar.
2. **Sin funcionamiento offline**: no había service worker.
3. **Un solo archivo gigante**: cualquier cambio obligaba a editar 707 líneas mezclando
   estilos, datos y lógica. Sin caché selectiva: el navegador revalidaba todo.
4. **Datos y código acoplados**: los pueblos iban dentro del HTML, imposible de regenerar
   o de actualizar sin tocar la lógica.
5. **Respaldo de relieve inválido**: se intentaba usar OpenTopoData, cuya API no envía
   cabeceras CORS; el navegador bloqueaba la petición y solo quedaba Open-Meteo.
6. **CSS duplicado y con erratas** en la versión previa de la refactorización.
7. **Arranque sin feedback**: la portada no explicaba el estado de carga ni los permisos.

---

## 2. Mejoras aplicadas

### 2.1 Arquitectura modular

| Antes | Después |
|---|---|
| 1 archivo (707 líneas) | `index.html` + 4 hojas CSS + `app.js` + 2 archivos de datos |
| CSS y JS en línea | Archivos separados, cacheables por el navegador |
| Datos embebidos en el HTML | `data/pueblos.js` y `data/picos.js` |

Ventajas reales: el navegador cachea `pueblos.js` (4,2 MB) y `picos.js` (1,8 MB) por
separado del HTML; al cambiar solo la interfaz no se vuelven a descargar los datos.

### 2.2 PWA completa

- `manifest.json` con iconos 180/192/512 y versión *maskable*.
- `sw.js` que precarga la app y sirve **caché primero** para los recursos propios,
  **red primero** para la navegación y deja pasar las APIs externas a la red.
- `.nojekyll` para que GitHub Pages no procese los archivos con Jekyll.

Con esto la app es **instalable** y abre **sin conexión** tras la primera visita.

### 2.3 Carga diferida de picos

`data/picos.js` (1,8 MB) **no se descarga** hasta que se pulsa **Picos**. La portada carga
solo lo imprescindible.

### 2.4 Relieve más robusto

- OpenTopoData eliminado; **Open-Meteo es el único proveedor remoto**.
- Presupuesto local de **520 puntos/minuto** con ventana deslizante de 60 s.
- Panorama **adaptativo** (2,5° → 5° → 7,5°) según el cupo disponible.
- Reserva de puntos para las altitudes de las etiquetas.
- Caché de relieve en **IndexedDB** por celdas, para no repetir peticiones.
- Reintentos automáticos y aviso claro cuando se agota el cupo.

### 2.5 Accesibilidad y arranque

- Etiquetas `aria-*`, `role` y `aria-live` en estados y diálogos.
- Botón de inicio con `aria-busy` mientras se cargan los datos.
- Avisos legibles antes de pedir los permisos de cámara, GPS y brújula.

---

## 3. Qué se ha modificado en este repositorio

- `index.html`: versión limpia y modular (sin CSS ni JS en línea).
- `styles/`: cuatro hojas coherentes (`base`, `portada`, `layout`, `camera`).
- `scripts/app.js`: motor completo como script clásico (compatible con `file://`).
- `data/`: localidades y picos extraídos del bloque embebido.
- `manifest.json`, `sw.js`, `icons/`, `icon.png`: PWA e instalación.
- `README.md` y este documento.

El `index.html` original de 707 líneas se conserva **fuera del repositorio**, en la
carpeta de copia de seguridad `Pueblos_backup/`, por si hay que comparar o revertir.

---

## 4. Pendiente / posibles mejoras futuras

- Minificar `pueblos.js` / `picos.js` (reduciría bastante el primer arranque).
- Empaquetar los datos en binario + `DecompressionStream` (el motor ya soporta
  `#datosRelieve`).
- Tests automáticos del cálculo de rumbo/visibilidad (hoy se han probado a mano).
- Sustituir el `alert`/textos de error por mensajes en línea (parcialmente hecho).

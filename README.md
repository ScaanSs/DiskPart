# DiskPart+

**Registro forense de la actividad de discos en Windows.**

DiskPart+ registra todo lo que les pasa a los discos de un equipo: discos y USB que entran y salen, discos virtuales (VHD/VHDX/ISO) que se montan y expulsan, letras que se asignan o se quitan, particiones creadas o borradas, discos formateados, y programas o archivos usados desde otros discos.

Sirve para responder preguntas como *«¿alguien conectó un USB, ejecutó algo y se lo llevó?»*. Además de lo que registra a partir de su instalación, reconstruye lo que pasó **antes** a partir de las huellas que Windows guarda por su cuenta.

---

## Requisitos

- Windows 10 u 11 (64 bits).
- Permisos de administrador (el programa los pide al abrirse).
- No necesita instalar nada más: es un único `DiskPart+.exe`.

---

## Primer uso

1. Abre `DiskPart+.exe` y acepta el aviso de administrador.
   - Como el programa no está firmado digitalmente, Windows SmartScreen puede avisar la primera vez: **Más información → Ejecutar de todas formas**.
2. DiskPart+ te ofrecerá **instalar el servicio de registro**. Acepta: es lo que permite registrar la actividad aunque la ventana esté cerrada.
3. La primera vez tarda un poco más, porque analiza el historial de Windows y busca discos virtuales en todas las unidades.

Al cerrar la ventana, el programa se cierra del todo; no se queda en la bandeja del sistema. El registro lo sigue llevando el servicio, que no tiene icono ni ventana.

---

## Cómo funciona

DiskPart+ tiene dos partes que están en el mismo `.exe`:

| Parte | Qué hace |
|---|---|
| **Servicio de registro** | Funciona en segundo plano desde que se enciende el equipo, incluso antes de iniciar sesión. Comprueba cada segundo el estado de discos, volúmenes, particiones, letras y unidades, y anota cada cambio con su hora exacta. |
| **Ventana** | Muestra el estado actual y la cronología completa, y reúne lo registrado por el servicio con las huellas que deja Windows. |

### Dónde se guardan los datos

- **Visor de eventos de Windows** → Registros de Windows → **Aplicación**, origen **«DiskPart+»**. No se crea ningún archivo de texto.
- **Copia protegida** en el registro de Windows (`HKLM\SOFTWARE\DiskPart+\Copia`), con los últimos ~30.000 eventos. Solo un administrador puede modificarla.
- El servicio se instala en `C:\Archivos de programa\DiskPart+\DiskPart+.exe`, así que sigue funcionando aunque borres o muevas el `.exe` de Documentos.

---

## La ventana

### Parte superior

- **Estado actual:** número de discos, particiones, unidades con letra y VHD montados.
- **Servicio de registro:** muestra si está activo, detenido o sin instalar, con botones para instalarlo, actualizarlo o desinstalarlo.
- **Mostrar:** periodo de la cronología: desde el inicio de sesión, últimas 24 horas, últimos 7 días, últimos 30 días o todo el historial disponible.
- **Analizar de nuevo** y **Exportar CSV…**

### Pestañas

| Pestaña | Contenido |
|---|---|
| **Discos conectados ahora** | Número, nombre, tipo de conexión (USB, SATA, NVMe, VHD…), tamaño, letras, estado, nº de serie y ruta del archivo si es un VHD. |
| **Particiones** | Todas las particiones, tengan letra o no (EFI, MSR, recuperación, ocultas…), con su tipo, tamaño, letra, sistema de archivos y etiqueta. |
| **Dispositivos que se han conectado alguna vez** | Cada disco, USB, VHD o lector que Windows recuerda, con su nº de serie, **primera conexión**, **última conexión**, **última desconexión** y si está conectado ahora. |
| **Letras usadas** | A qué dispositivo asignó Windows por última vez cada letra, incluidas las letras que ya no están en uso. |
| **Archivos VHD / VHDX** | Todos los discos virtuales del equipo (ver más abajo). |
| **Unidades con letra** | Cada letra con su tipo y si **hay un disco puesto o no** (lectores de CD/DVD y de tarjetas). |

### Cronología de actividad

La lista inferior es el centro del análisis. Cada línea tiene:

- **Hora:** fecha y hora real del suceso.
- **Tipo:** `ENTRA`, `SALE`, `CAMBIO`, `USO` (programa o archivo usado), `ALERTA` o `INFO`.
- **Origen:** de dónde sale la información (ver la tabla siguiente).
- **Tipo de archivo:** qué es el archivo (programa, script, documento…). Los programas y scripts llevan ⚠.
- **Evento** y **Detalle**.

Colores: verde = entra · rojo = sale/borrado · amarillo = cambio · morado = programa o archivo usado · rosa = alerta.

**Filtros:** *Ver* (discos que entran o salen, programas y archivos usados, letras/particiones/formateos/borrados, alertas, solo programas y scripts) y **Buscar** (por nombre de archivo, letra, nº de serie, modelo, tipo…).

**Exportar CSV** guarda lo que tienes filtrado, en orden cronológico, en un archivo que se abre directamente en Excel.

---

## Qué detecta y de dónde lo saca

| Origen | Qué aporta | ¿Necesita el servicio? |
|---|---|---|
| **Servicio** | Cada cambio en directo: discos y VHD conectados o expulsados, letras asignadas, quitadas o cambiadas, particiones creadas, eliminadas o redimensionadas, medios insertados o expulsados, unidades de red, discos en línea o fuera de línea. También lo que cambió **con el equipo apagado** (indica entre qué horas). | Sí |
| **Windows** | Historial de particiones de Windows: llegada y salida de cada disco, **discos borrados** (sin particiones), **particiones eliminadas o creadas**, **volúmenes formateados**, ISO montadas, encendidos, apagados y apagados inesperados. | No |
| **Dispositivos** | Primera y última conexión y última desconexión de cada dispositivo de almacenamiento. | No |
| **Prefetch** | **Programas ejecutados desde otros discos** (con sus últimas 8 ejecuciones) y archivos de otros discos usados por programas. Identifica el disco por el nº de serie del volumen aunque ya no esté conectado. | No |
| **BAM** | Última ejecución de cada programa, por usuario, con el volumen de origen. | No |
| **UserAssist** | Programas abiertos con doble clic desde otras unidades, con número de veces. | No |
| **Recientes** | Archivos abiertos desde discos extraíbles, CD/DVD u otros discos, con etiqueta y nº de serie del volumen. | No |
| **Registro** | Hora del último cambio en la asignación de letras. | No |
| **Copia protegida** | Eventos recuperados que alguien borró del Visor de eventos. | Sí |

### Ejemplo

Alguien conecta un USB, ejecuta un programa y se lo lleva. En la cronología aparecería algo así:

```
ENTRA   Dispositivos  Última conexión del dispositivo — Kingston DataTraveler — USB — S/N 60A44C…
USO     Prefetch      Programa EJECUTADO desde otro disco: SETUP.EXE
                      Ruta: [1A2B-3C4D]\setup.exe · ese disco NO está conectado ahora
                      · identificado como: Kingston DataTraveler USB (S/N 60A44C…)
SALE    Dispositivos  Última desconexión / expulsión del dispositivo — Kingston DataTraveler
```

### Tipo de archivo y archivos engañosos

DiskPart+ reconoce unos 80 tipos de archivo. Si el disco sigue conectado, abre el archivo y comprueba qué es en realidad. Si un archivo se hace pasar por otra cosa (por ejemplo, `factura.pdf` que en realidad es un programa), lo marca así:

> ⚠ ¡ENGAÑOSO! se llama pdf pero es: Programa de Windows (ejecutable)

---

## Discos virtuales (VHD / VHDX)

La pestaña **Archivos VHD / VHDX** busca en todas las unidades locales y extraíbles los archivos `.vhd`, `.vhdx`, `.avhd` y `.avhdx`, y les añade los VHD que Windows recuerda haber montado.

Cada disco virtual aparece con uno de estos estados:

- **MONTADO con letra X:**
- **Montado SIN letra**
- **No montado**
- **Ya NO existe (borrado o movido):** Windows lo montó en algún momento, pero el archivo ya no está.

También muestra el tipo, el tamaño, el nº de disco, **la última vez que Windows lo usó** y las fechas de modificación y creación.

**Clic derecho** sobre un archivo:

- **Montar**
- **Montar solo lectura:** recomendado para revisar un disco sospechoso sin modificar nada.
- **Expulsar**
- **Abrir ubicación** (también con doble clic)
- **Copiar ruta**

Funciona con VHD, VHDX e ISO, sin necesitar Hyper-V.

---

## Protección del registro

DiskPart+ avisa con una **ALERTA** cuando alguien intenta tapar lo que ha hecho:

- **Se borra un registro de eventos** de Windows o el registro de Seguridad (indica qué usuario lo hizo).
- **Faltan eventos de DiskPart+ en el Visor de eventos:** los recupera de la copia protegida y los muestra con el origen «Copia protegida».
- **Se detiene el servicio con el equipo encendido.** Las paradas normales al apagar o reiniciar no generan alerta, y las actualizaciones del propio DiskPart+ tampoco.
- **El servicio termina a la fuerza** o **le cambian el tipo de inicio**.
- **El servicio ha sido desinstalado.**
- **El registro se interrumpió de forma inesperada** (corte de luz, cuelgue…).

Además, si el servicio se cae, Windows lo vuelve a arrancar automáticamente.

---

## Identificadores de evento (Visor de eventos → Aplicación → origen DiskPart+)

| ID | Significado |
|---|---|
| 1 | Servicio de registro iniciado |
| 2 | Servicio de registro detenido |
| 3 | Estado de los discos al iniciar el registro |
| 4 | El registro se interrumpió de forma inesperada |
| 5 | **ALERTA** — servicio detenido manualmente con el equipo encendido |
| 6 | **ALERTA** — se borró un registro de eventos de Windows |
| 7 | **ALERTA** — cambios en el servicio (desinstalación, tipo de inicio…) |
| 100 | Algo ENTRA (disco, VHD, letra, partición, medio) |
| 150 | Algo ENTRÓ con el equipo apagado o el servicio detenido |
| 200 | Algo SALE |
| 250 | Algo SALIÓ con el equipo apagado o el servicio detenido |
| 300 | CAMBIO (letra cambiada, redimensión, en línea/fuera de línea) |
| 350 | CAMBIO ocurrido con el equipo apagado o el servicio detenido |
| 400 | Información |
| 500 | Error leyendo el estado de los discos |

---

## Limitaciones

- **Cambios de letra antes de instalar el servicio:** Windows no guarda un historial de letras, solo la última asignación de cada una y la hora del último cambio. Desde que el servicio está instalado, todos los cambios de letra quedan registrados.
- **Dispositivos:** Windows guarda la primera y la última conexión de cada dispositivo, no todas las intermedias.
- **Prefetch** guarda las últimas 8 ejecuciones de cada programa. En algunos equipos está desactivado.
- **Historial de Windows:** los registros de eventos tienen un tamaño limitado y Windows descarta lo más antiguo cuando se llenan.
- **UserAssist** solo se lee para el usuario que abre DiskPart+.
- **Copia protegida:** un administrador que sepa que existe podría borrarla. Aun así, borrar el Visor de eventos o detener el servicio deja rastro en los registros propios de Windows.

---

## Actualizar

Sustituye `DiskPart+.exe` por la versión nueva y ábrelo. Al abrirse, la ventana actualiza el servicio a esa versión de forma automática.

## Desinstalar

1. Abre DiskPart+ y pulsa **Desinstalar servicio**. La desinstalación queda registrada.
2. Borra `DiskPart+.exe` y la carpeta `C:\Archivos de programa\DiskPart+`.
3. Opcional: borra la clave `HKLM\SOFTWARE\DiskPart+` del registro (contiene la copia protegida y el último estado conocido).

Los eventos ya escritos en el Visor de eventos se conservan.

---

## Solución de problemas

| Problema | Solución |
|---|---|
| SmartScreen bloquea el programa | **Más información → Ejecutar de todas formas.** |
| El servicio aparece como «NO INSTALADO» | Pulsa **Instalar servicio**. |
| El servicio aparece como «DETENIDO» | Pulsa **Reinstalar / arrancar servicio**. |
| «Prefetch: la carpeta está vacía…» | Prefetch está desactivado en ese equipo. El resto del análisis funciona igual. |
| No se puede expulsar un VHD | Cierra los archivos y ventanas que estén usando ese disco y vuelve a intentarlo. |
| La búsqueda de VHD tarda | Depende del tamaño de los discos. Puedes seguir usando el resto de la ventana mientras busca. |

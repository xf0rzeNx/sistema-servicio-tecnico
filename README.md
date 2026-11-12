# Sistema para servicio técnico

Sistema web para talleres y servicios técnicos: garantías (RMAs), presupuestos,
seguimiento online para el cliente, chat en vivo, avisos por WhatsApp, compras,
envíos y facturas de crédito. Pensado para Uruguay (pesos y dólares, IVA, C.I. y RUT),
funciona en cualquier navegador y en el celular.

Viene en **dos versiones** con las mismas funciones: **web** (en PythonAnywhere u otro servidor, se usa desde
cualquier lado) y **de escritorio** (un programa de Windows instalado en una PC del local).

> **Software con licencia.** Leé [LICENCIA.md](LICENCIA.md) antes de instalarlo.

---

## Qué incluye

| Módulo | Para qué sirve |
|---|---|
| **Panel de RMAs** | Cargar equipos en garantía, seguir su estado, historial, fotos, notas, remito de ingreso y de retiro con QR. |
| **Presupuestos** | Reparaciones o ventas con precios con IVA incluido, archivos para el cliente, impresión y remito. |
| **Portal de seguimiento** | Sitio aparte donde el cliente, con el código de su remito, ve el estado de su equipo, acepta o rechaza el presupuesto y chatea con el taller. |
| **Chat en vivo** | Estilo Messenger, con fotos, archivos, videos de YouTube y respuesta automática. |
| **Avisos por WhatsApp** | Al cambiar el estado, manual (gratis), con tu propio servidor WAHA (gratis) o con la API oficial de Meta (paga). |
| **Servicios y tarifario** | La lista de precios de los trabajos del taller, para mostrársela al cliente. |
| **Clientes y empresas** | Agenda por cédula o RUT: los datos se completan solos. |
| **Compras, catálogo, envíos, facturas de crédito, proveedores, estadísticas** | Administración del local. |
| **Usuarios y permisos** | Cada usuario ve solo los apartados que le das (ver, editar o todo). |

---

## Qué necesitás

* Dos sitios web en **[PythonAnywhere](https://www.pythonanywhere.com)** (o cualquier servidor con Python 3.10 o más nuevo):
  1. **el sistema** (carpeta `sistema/`), que usa el personal;
  2. **el portal de seguimiento** (carpeta `portal/`), que usan los clientes.

  Con la cuenta gratis de PythonAnywhere se puede tener un solo sitio por cuenta: usá **dos cuentas gratis**
  (por ejemplo `miempresa` y `miempresaseguimiento`) o una cuenta paga.
* Unos 20 minutos.

---

## Instalación en PythonAnywhere

### 1. Subir el sistema

1. Entrá a la cuenta del **sistema** → pestaña **Web** → **Add a new web app** → *Manual configuration* → **Python 3.10** (o la más nueva).
2. Pestaña **Files**: creá la carpeta `/home/TU_USUARIO/mysite` y subí ahí **todo lo que hay dentro de `sistema/`**
   (`app.py`, `enviar_alertas.py`, `requirements.txt` y las carpetas `templates/` y `static/` completas).
3. Pestaña **Consoles** → **Bash**, y escribí:
   ```bash
   cd ~/mysite
   pip3.10 install --user -r requirements.txt
   ```

### 2. Generar las claves

En la misma consola, corré **tres veces** este comando y guardá cada resultado (son tus claves, no las compartas):
```bash
python3.10 -c "import secrets; print(secrets.token_hex(32))"
```
* La primera es la **SECRET_KEY**.
* La segunda es el **TRACKING_API_TOKEN** (va igual en el sistema y en el portal).
* La tercera es la **CLAVE_INSTALACION** (la pide el asistente la primera vez).

### 3. Configurar el archivo WSGI del sistema

Pestaña **Web** → link del **WSGI configuration file**. Borrá todo y pegá esto, con tus datos:
```python
import os, sys

os.environ['SECRET_KEY'] = 'la-primera-clave'
os.environ['TRACKING_API_TOKEN'] = 'la-segunda-clave'
os.environ['CLAVE_INSTALACION'] = 'la-tercera-clave'
os.environ['URL_PORTAL_RASTREO'] = 'https://TU-PORTAL.pythonanywhere.com'

carpeta = '/home/TU_USUARIO/mysite'
if carpeta not in sys.path:
    sys.path.insert(0, carpeta)
os.chdir(carpeta)

from app import app as application
```
Guardá y, en la pestaña **Web**:
* **Static files**: URL `/static/` → carpeta `/home/TU_USUARIO/mysite/static`
* **Force HTTPS**: activado
* Tocá **Reload**.

### 4. Primer inicio

Abrí `https://TU_USUARIO.pythonanywhere.com`. Aparece el **asistente de instalación**:
1. el nombre, teléfono, WhatsApp y dirección de tu empresa;
2. el usuario administrador y su contraseña;
3. la **clave de instalación** (la tercera del paso 2).

Listo: entrás con ese usuario. Después, en **⚙️ (tu usuario) → Datos de la empresa**, subí el **logo**,
revisá las **condiciones del remito** y completá el resto.

### 5. Cargar la licencia

Recién instalado, el sistema anda **15 días de prueba** (arriba aparece un cartel con los días que quedan).
Para seguir usándolo, cargá la **clave de licencia** que te mandaron al comprarlo:
**⚙️ (tu usuario) → 🔑 Licencia** → pegala entera (empieza con `LIC1-`) → **Activar**.

* Cada licencia sirve en **hasta 2 sitios**: la dirección o la IP de tu sistema (por ejemplo
  `miempresa.pythonanywhere.com`, o `192.168.1.10` si lo usás en una PC del local). En otro sitio no anda.
  La pantalla de la licencia te muestra la dirección exacta para pasarle al vendedor. Si te mudás, pedí una clave nueva.
* Si la prueba termina o la licencia vence, el sistema solo deja entrar a la pantalla de la licencia: **no se
  pierde ningún dato**, y vuelve a andar apenas cargás la clave. El portal de los clientes sigue andando.
* La licencia se verifica en el mismo sistema, sin internet.

### 6. Subir el portal de seguimiento

En la **otra** cuenta (o en otro sitio de la cuenta paga):
1. Creá la web app igual que en el paso 1, y subí a `/home/TU_USUARIO/mysite` **todo lo de `portal/`**.
2. En la consola: `cd ~/mysite && pip3.10 install --user -r requirements.txt`
3. Archivo WSGI del portal:
   ```python
   import os, sys

   os.environ['API_URL'] = 'https://TU-SISTEMA.pythonanywhere.com/api/v1/tracking_seguro/'
   os.environ['TRACKING_API_TOKEN'] = 'la-segunda-clave'

   carpeta = '/home/TU_USUARIO/mysite'
   if carpeta not in sys.path:
       sys.path.insert(0, carpeta)
   os.chdir(carpeta)

   from app import app as application
   ```
4. **Static files**: `/static/` → `/home/TU_USUARIO/mysite/static`, **Force HTTPS** activado, **Reload**.
5. Probá `https://TU-PORTAL.pythonanywhere.com/estado-interno`: tiene que decir `"token_configurado": true` y `"chat": "ok"`.

> Si usás la cuenta gratis de PythonAnywhere para el portal, en la cuenta del **sistema** no hace falta
> nada más. El portal solo se conecta al sistema; los clientes nunca entran al sistema.

---

## Versión de escritorio (Windows)

En vez de usarlo por internet, el sistema se puede instalar como un **programa en una PC del local**.

1. Ejecutá `Instalar-Servicio-Tecnico-X.exe`. No pide permisos de administrador.
   * Si aparece **"Windows protegió su PC"**: **Más información → Ejecutar de todas formas**.
   * Si querés que el sistema arranque solo al prender la PC, marcá esa opción.
2. Abrilo desde el ícono **Servicio Técnico**. La primera vez aparece el asistente: datos de la empresa y usuario
   administrador. En esta PC no pide clave de instalación.
3. Si Windows pregunta por el **firewall**, tocá **Permitir acceso**.
4. Al lado del reloj queda un ícono 🛠️ con todo a mano: abrir el sistema, ver la dirección para las otras PCs,
   hacer una copia de seguridad, o apagarlo.

**Otras PCs del local:** entran desde el navegador a la dirección que muestra el ícono, por ejemplo
`http://192.168.1.10:5000`. La PC donde está instalado tiene que estar prendida.

**Licencia:** en **⚙️ → 🔑 Licencia** está el **código de esta PC** (`PC-XXXX-…`). Pasáselo al vendedor y
pegá la clave que te mande.

**Datos y copias de seguridad:**
* Los datos están en `C:\Users\TU_USUARIO\AppData\Local\ServicioTecnico`. No se borran al actualizar ni al
  desinstalar.
* Cada día, y cada vez que lo apagás, se hace una copia en **Documentos\Servicio Técnico - Copias**. Quedan las
  últimas 10. Si tus Documentos están en OneDrive, las copias quedan también en la nube.
* Para volver a una copia, seguí el `LEEME.txt` que viene adentro de cada una.

**Correo y WhatsApp:** los ajustes (`SMTP_USER`, `WAHA_API_KEY`, `ZONA_HORARIA`…) van en el archivo
`ajustes.env` de la carpeta de datos (desde el ícono: **Abrir la carpeta de datos**). Después apagá y abrí el
sistema de nuevo.

**Actualizar:** ejecutá el instalador de la versión nueva encima. Los datos quedan.

**Portal de seguimiento:** en la versión de escritorio, el portal para los clientes (estado del equipo y chat)
todavía no se conecta, porque el sistema está en la red del local y no en internet. Se suma en una próxima versión.

---

## Opcionales

### Correo de alertas (RMAs demorados)
Agregá en el WSGI del sistema `SMTP_USER`, `SMTP_PASSWORD` (en Gmail, una *contraseña de aplicación*) y
`ALERTAS_DESTINO`. Para que salga solo, en la pestaña **Tasks** creá una tarea diaria con:
```bash
python3.10 /home/TU_USUARIO/mysite/enviar_alertas.py
```

### WhatsApp automático
En el sistema: **⚙️ → Avisos por WhatsApp**. Está explicado en la misma pantalla:
* **Manual** (gratis): aparece un botón que abre WhatsApp con el mensaje escrito.
* **WAHA** (gratis): tu propio servidor en una PC del local. La clave va en el WSGI como `WAHA_API_KEY`.
* **Meta** (pago por mensaje): `WHATSAPP_TOKEN` y `WHATSAPP_PHONE_ID` en el WSGI.

### Zona horaria
Por defecto, Uruguay. Para otro país: `os.environ['ZONA_HORARIA'] = 'America/Argentina/Buenos_Aires'` (por ejemplo).

Todas las variables posibles están en `sistema/.env.ejemplo` y `portal/.env.ejemplo`.

---

## Copias de seguridad

Los datos están en los archivos `rma.db`, `facturas.db` y `catalogo.db` y en la carpeta `uploads/` de
`/home/TU_USUARIO/mysite`. Descargalos cada tanto desde la pestaña **Files** (o con una tarea programada).
**Nunca** los subas a GitHub.

## Actualizar a una versión nueva

1. Hacé una copia de seguridad (arriba).
2. Reemplazá `app.py` y las carpetas `templates/` y `static/` por las nuevas. **No borres** los `.db` ni `uploads/`.
3. Tocá **Reload**. El sistema actualiza solo la base de datos al arrancar.

## Seguridad

* Las claves van **solo** en el archivo WSGI o en un `.env` que no se sube a ningún lado.
* El portal nunca muestra nombre, teléfono, cédula ni dirección del cliente: solo el estado del equipo.
* Cada usuario entra con su contraseña; los permisos se dan apartado por apartado en **Usuarios**.
* Si alguna clave se filtró, generá otra (paso 2), cambiala en los dos WSGI y tocá **Reload**.

## Soporte

Ver [LICENCIA.md](LICENCIA.md) para las condiciones de soporte y actualizaciones de tu licencia.

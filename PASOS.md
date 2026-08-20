# TikTok — de borrador a publicación directa

Guía de los pasos que faltan. Lo que ya está escrito y listo va marcado; lo que
requiere que Alfredo toque algo va numerado.

## Lo que encontré al revisar el portal

El formulario de **Producción estaba vacío**: todo lo que habíamos configurado vivía
solo en el Sandbox. Además, en el Sandbox el **Direct Post estaba apagado**, y por eso
el permiso `video.publish` no aparecía por ningún lado — ese permiso no se agrega desde
"Add scopes", viene incluido al encender Direct Post.

El bloqueo de fondo es otro: TikTok exige que el video demostrativo muestre **una
interfaz y las interacciones de un usuario**. Un cron de GitHub Actions no tiene
interfaz, así que grabar la corrida se habría rechazado. Por eso ahora existe
`publicar.html`.

## Lo que ya está listo (no requiere nada de tu parte)

- `publicar.html` — la interfaz completa del flujo: conectar, ver los datos reales de
  la cuenta, elegir el video, escribir la descripción, elegir privacidad, declarar
  contenido comercial, revisar el resumen y publicar.
- `worker.js` — el backend mínimo que guarda el `client_secret` fuera del navegador.
- `icono.png` — 1024x1024, el App icon que faltaba.
- `videos/006.mp4` + `videos/index.json` — el video de prueba, servido desde el dominio
  que ya está verificado en TikTok.
- El texto en inglés para el campo de revisión (abajo, 996 caracteres de 1000).

La interfaz respeta las tres reglas por las que TikTok rechaza más seguido:

1. Solo ofrece los niveles de privacidad que `creator_info` devuelve para esa cuenta.
2. Deshabilita (no solo desmarca) las casillas de comentar / dueto / stitch cuando la
   cuenta las tiene apagadas, y lo dice en pantalla.
3. El botón Publicar queda bloqueado hasta que se elige un nivel de privacidad, y antes
   de publicar muestra un resumen con descripción, privacidad y ajustes de interacción.

---

## Paso 1 — Subir los archivos nuevos al repositorio `tiktok-callback`

En `github.com/jovninis1-byte/tiktok-callback` → **Add file → Upload files**, y sube:

- `publicar.html`
- `videos/index.json`
- `videos/006.mp4`

(`worker.js` e `icono.png` no van al repo: uno se pega en Cloudflare y el otro se sube
al portal de TikTok.)

## Paso 2 — Crear la cuenta de Cloudflare y desplegar el Worker

El plan gratuito alcanza de sobra: 100 000 peticiones por día y nosotros haremos unas
pocas decenas.

1. Entra a `dash.cloudflare.com/sign-up` y crea la cuenta. **La contraseña la pones tú;
   yo no la manejo.**
2. En el panel: **Compute (Workers) → Create → Start from Hello World → Deploy**.
3. Abre **Edit code**, borra lo que traiga y pega el contenido de `worker.js`. Deploy.
4. En **Settings → Variables and Secrets**, agrega:

   | Nombre | Tipo | Valor |
   |---|---|---|
   | `TIKTOK_CLIENT_KEY` | Secret | El Client key del **Sandbox** |
   | `TIKTOK_CLIENT_SECRET` | Secret | El Client secret del **Sandbox** |
   | `REDIRECT_URI` | Text | `https://jovninis1-byte.github.io/tiktok-callback/publicar.html` |
   | `ORIGEN_PERMITIDO` | Text | `https://jovninis1-byte.github.io` |

   Se usa el Sandbox porque TikTok lo exige: *"If your app has not been approved before,
   you are required to use a sandbox environment to demonstrate the integration."*

5. Copia la URL del Worker (queda como `https://algo.tu-nombre.workers.dev`).
6. En `publicar.html`, cambia la línea

   ```js
   const WORKER = "https://REEMPLAZAR.workers.dev";
   ```

   por tu URL, y vuelve a subir el archivo.

## Paso 3 — Configurar el Sandbox

En el portal de TikTok, pestaña **Sandbox → agente-videos-pruebas**:

1. **Products → Content Posting API → activar "Direct Post"**. Esto agrega
   `video.publish` automáticamente.
2. **Login Kit → Redirect URI**: agregar
   `https://jovninis1-byte.github.io/tiktok-callback/publicar.html`
   (la que ya está, la de la raíz, puede quedarse.)
3. **Apply changes**.

## Paso 4 — Probar el flujo completo

Abre `https://jovninis1-byte.github.io/tiktok-callback/publicar.html` y recorre todo:
conectar, ver tu nombre y avatar, elegir el video, escribir la descripción, elegir
privacidad, revisar el resumen, publicar.

**Hazlo una vez sin grabar**, para descubrir cualquier error antes de la toma buena.
Si algo falla aquí, dímelo y lo corrijo.

## Paso 5 — Grabar el video demostrativo

En Windows: **Win + Alt + R** abre la grabadora de la Xbox Game Bar y guarda un mp4 en
`Vídeos\Capturas`. Máximo 50 MB, formato mp4 o mov.

Guion de la grabación, en este orden y sin cortes:

1. Empieza con la barra de direcciones visible mostrando
   `jovninis1-byte.github.io/tiktok-callback/publicar.html`. Esto importa: TikTok
   compara el dominio del video con el que declaraste.
2. Clic en **Conectar con TikTok**.
3. Se ve la pantalla de consentimiento de TikTok con los permisos. Acepta.
4. Regresa a la página: se ve tu apodo, tu avatar y la duración máxima permitida.
5. Elige el video en la lista. Se ve la vista previa.
6. Escribe la descripción despacio, que se vea el texto apareciendo.
7. Abre el desplegable de privacidad: que se vean las opciones. Elige una.
8. Toca las casillas de comentar / dueto / stitch. Si alguna sale deshabilitada,
   **detente un segundo ahí**: es justo lo que el revisor quiere ver.
9. Marca la casilla de contenido comercial, deja ver que aparece el texto legal, y
   desmárcala si el video no es comercial.
10. Baja al resumen y **léelo en pantalla un par de segundos**: descripción, privacidad,
    interacciones.
11. Clic en **Publicar en TikTok**. Espera a que salga el identificador de publicación.
12. Abre la app de TikTok y muestra el video ya en el perfil.

No cortes ni aceleres el video. Un recorrido sin la parte de autorización se rechaza.

## Paso 6 — Llenar Producción y enviar

En la pestaña **Production**:

1. **Import → Import from Sandbox → agente-videos-pruebas**. Copia toda la
   configuración de golpe.
2. Sube `icono.png` en **App icon**.
3. Verifica que quedaron: categoría Education, descripción, las dos URLs legales,
   plataforma Web, el Redirect URI, Direct Post encendido y los permisos
   `user.info.basic` + `video.publish`.
4. En **App review**, pega el texto de abajo en el campo de explicación.
5. Sube el mp4 de la grabación.
6. **Submit for review**.

### Texto para el campo de explicación (996 caracteres)

```
Personal, single-user tool. I publish only my own original short educational videos (practical AI and tech, in Spanish) to my own account, @creatividai.

Login Kit + user.info.basic: the user clicks "Conectar con TikTok", authorizes, and the app shows the nickname and avatar TikTok returns, to confirm the correct account. Nothing else is stored.

Content Posting API + video.publish: the app calls creator_info first and builds the composer strictly from it - only the privacy levels the account allows are listed, and the comment, duet and stitch checkboxes are disabled when the account has them off. The user writes the caption, picks a privacy level (Publish stays disabled until one is chosen), declares commercial content when applicable, and reviews a summary of caption, privacy and interaction settings before publishing. The video is sent with PULL_FROM_URL from jovninis1-byte.github.io, verified in this app.

Narration is synthetic TTS. No third-party copyrighted material is used.
```

## Paso 7 — Cuando aprueben

1. Correr `obtener_token_tiktok.py` otra vez, pero ahora con las credenciales de
   **Producción** (son distintas a las del Sandbox) y con `SCOPES` cambiado a
   `user.info.basic,video.publish`.
2. Actualizar los tres secretos en GitHub.
3. Cambiar la variable `TIKTOK_MODO` de `borrador` a `directo`.

No hay que tocar el código del agente: ya soporta los dos modos.

---

## Dos avisos honestos

**Hay dos puertas, no una.** La revisión de la app desbloquea `video.publish`. Aparte
existe la *auditoría de contenido*: mientras el cliente no esté auditado, TikTok fuerza
todo lo publicado por API a modo privado. Puede que se resuelvan con el mismo envío o
puede que haya que pedir la segunda por soporte; se sabrá al recibir la respuesta.

**El tiempo de respuesta es variable.** El acceso al producto suele tardar días; la
auditoría de contenido, bastante más. Mientras tanto el modo borrador sigue funcionando
y no hay que apagar nada.

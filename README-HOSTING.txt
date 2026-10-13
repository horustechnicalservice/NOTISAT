CENTINELA UFO ARGENTINA — MAPA PROPIO (imagen del usuario)

El mapa de fondo ahora es EXCLUSIVAMENTE tu imagen: assets/mapa-sudamerica.jpg
- Se eliminaron las teselas satelitales/cartograficas, las fronteras dibujadas por codigo,
  los nombres de paises, el contorno amarillo de Argentina y las Islas Malvinas dibujadas.
- La imagen se muestra completa, sin filtros ni vineta, con su proporcion original (941x1672).
- Los satelites, trayectorias y calculos orbitales (SGP4) se colocan sobre la imagen mediante una
  proyeccion calibrada con puntos reales (Titicaca, Uyuni, Peninsula Valdes, Malvinas, Uruguay,
  Tierra del Fuego). Error medio aprox. 8 px sobre 941 px.
- El panel de la app se reacomodo para no tapar los marcos y el radar que ya trae tu imagen.

Para cambiar la imagen: reemplaza assets/mapa-sudamerica.jpg por otra de la MISMA composicion.
Si usas otra distinta, hay que recalibrar el objeto GEO dentro de index.html.

El service worker se versiono como centinela-v12-mapa-usuario: cierra la app por completo y vuelve
a abrirla para que se vea el cambio. Leaflet, satellite.js y topojson siguen cargandose desde CDN.

----------------------------------------------------------------------
ALERTA MISIONES (notificaciones)
----------------------------------------------------------------------
- Campana (a la izquierda de SOBRE): activa/desactiva el aviso. Verde = activa. Se recuerda entre sesiones.
- Avisa 6 minutos antes de que un satelite pase por Misiones o por su zona de influencia
  (limite aproximado de la provincia + 100 km). Ajustable en index.html, objeto MIS_ALERT
  (leadMin (6), bufferKm, cooldownMin).
- SOBRE > "PROBAR NOTIFICACION" envia un aviso de prueba para verificar permisos.
- Siempre se muestra un aviso dentro de la app; ademas se intenta la notificacion del sistema por
  este orden: (1) puente Android window.Android.showNotification(titulo, texto),
  (2) Service Worker, (3) Notification API del navegador.

Puente Android (WebView) — ejemplo en Kotlin:

  class NotifBridge(private val act: Activity) {
    @JavascriptInterface fun requestNotificationPermission() {
      if (Build.VERSION.SDK_INT >= 33) ActivityCompat.requestPermissions(
        act, arrayOf(Manifest.permission.POST_NOTIFICATIONS), 1)
    }
    @JavascriptInterface fun showNotification(title: String, body: String) {
      // crear NotificationChannel "centinela" (IMPORTANCE_HIGH) y lanzar con NotificationCompat.Builder
    }
  }
  webView.addJavascriptInterface(NotifBridge(this), "Android")

Y en AndroidManifest.xml:  <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

IMPORTANTE: el calculo corre dentro de la pagina, asi que solo avisa mientras la app/WebView
este abierta (o viva en segundo plano). Con la app cerrada no puede avisar sin un servidor
o un servicio nativo en primer plano.

Notas v16 (alerta):
- La alerta vigila TODO el catalogo (antes solo los ~55 satelites dibujados en el mapa).
- La alerta usa HORA REAL. El reloj del mapa corre a velocidad x24 (variable speed), por eso un
  satelite que "pasa" por Misiones en el mapa NO coincide con un paso real.
- Al activar la campana o tocar "PROBAR NOTIFICACION" se muestra el estado y los proximos pasos
  reales de las estaciones (ISS/CSS) por Misiones, para verificar.
- Si la app estuvo en segundo plano y vuelve, hace un barrido completo inmediato.

NOTA v25 (alertas en segundo plano):
- La web calcula los pasos por Misiones de las proximas 8 h y los entrega a Android con
  window.Android.scheduleAlerts(json) (AlarmManager). Pasos con menos de 3 min de diferencia se
  agrupan en una sola notificacion. Se reprograma cada 10 min con la app abierta y al volver a ella.
- Configuracion en index.html, objeto MIS_ALERT: leadMin, bufferKm, horizonH, groupSec, maxAlarms y
  excludeTypes (ej. ['comms'] para ignorar Starlink).
- Requiere la version nueva de la app Android (NotifBridge + AlertReceiver).

NOTA v26 (continuidad del mapa):
- El mapa ahora usa HORA REAL (speed = 1). Antes el reloj del mapa iba x24 y reiniciaba al abrir/actualizar,
  por eso los satelites "volvian atras". Ahora, al salir y volver, estan en el mismo punto o mas adelante.
- Si se quisiera la animacion acelerada, cambiar "let speed = 1" en index.html (no es recomendable: reintroduce el salto atras).

NOTA v28 (prioridad Misiones, sin margen):
- bufferKm = 0: la alerta y la prioridad usan SOLO el territorio de Misiones, sin margen. La zona NO se dibuja en el mapa.
- Todo satelite que vaya a cruzar Misiones en los proximos 90 min (MIS_ALERT.priorityMin) se dibuja SIEMPRE,
  antes que Starlink y sin el tope de 55, con su orbita extendida hasta que salga de la provincia (resaltado en amarillo).
- Los demas satelites se muestran como siempre. Se reevalua cada 45 s y en cada actualizacion de TLE.
- Resolucion de la prediccion: 4-5 s (≈ 37 km), para no saltarse cruces cortos al no haber margen.

NOTA v29 (margen y contorno):
- bufferKm = 150 (zonas limitrofes: Corrientes, Paraguay, Brasil). La zona sigue sin dibujarse en el mapa.
- Se corrigio MIS_POLY: el contorno anterior dejaba afuera Posadas, Iguazu, El Soberbio y San Javier,
  por eso con margen 0 se podian perder pasos. Aplica tanto a la alerta como a la prioridad de dibujo.

NOTA v30 (satelites "trabados"):
- Causa: si UN satelite fallaba al dibujarse, createSatellites abortaba a mitad y los marcadores ya creados
  quedaban en el mapa fuera de la lista de actualizacion (huerfanos): se veian congelados. Ademas, un error
  dentro del bucle de animacion lo detenia para siempre.
- Ahora: registro de todo lo dibujado (se limpia siempre completo), cada satelite se dibuja/actualiza en su propio
  try/catch, el bucle de animacion nunca muere (try/finally + vigilante que lo reinicia), un satelite sin posicion
  valida se oculta en vez de quedar congelado, y no se dibujan duplicados.
- Prioridad Misiones acotada: MIS_ALERT.maxPriority = 24 (los que cruzan antes), mas los ~55 normales. Los que
  terminaron su paso se quitan solos. Se quito la sombra amarilla (pesada); queda el nombre en amarillo.

NOTA v31 (segundo plano):
- Con la app cerrada solo funciona si la app ANDROID tiene el modulo nativo (ver android-nativo.zip / LEEME.txt).
  En navegador o PWA no es posible: Android congela la pagina en segundo plano.
- Horizonte de alarmas nativas: 24 h (MIS_ALERT.horizonH), hasta 300 avisos.
- SOBRE > "DIAGNOSTICO + PRUEBA EN SEGUNDO PLANO" muestra por que (no) llegan: si hay modulo nativo, si la campana
  esta activa, cuantos avisos se programaron y (si la app lo informa) permisos de notificaciones, alarmas exactas
  y bateria. La pagina llama a window.Android.ensureBackgroundPermissions() y getStatus() si existen.

NOTA v32 (pestaña EN TRÁNSITO):
- El recuadro flotante "Próximo en tránsito" se quitó de la imagen y ahora es una pestaña desplegable en el borde
  izquierdo, igual que Tipos e Identificados, ubicada debajo de Identificados. Se abre/cierra al tocarla y solo hay
  un panel abierto a la vez. Su contenido (#nextPassBody) se sigue actualizando igual que antes.

NOTA v33: el panel EN TRÁNSITO se abre a la altura de su propia pestaña (debajo de Identificados) en vez de arriba,
para no taparse con el cartel ni con las otras ventanas. Se recalcula al abrir y al girar/redimensionar la pantalla.

NOTA v34: los tres paneles izquierdos (Tipos, Identificados, En tránsito) se abren cada uno a la altura de su propia
pestaña (alignLeftPanels), por lo que ya no se superponen ni se mezclan. Si el contenido no entra hacia abajo, hace
scroll dentro del propio panel.

NOTA v35: el bloque superior (CENTINELA UFO, tráfico orbital, scroll de satélites, en línea TLE vivo) sube completo,
con el mismo orden y separación. Ajuste fino: variable CSS --hdr-top (ahora 4.6%, antes 7.2%).

NOTA v36: se quitó el botón "ENTENDIDO · NO CONFUNDIR CON OVNIS" de la ventana SOBRE (se cierra con la ✕ o tocando fuera);
esa ventana ahora muestra barra de desplazamiento gruesa y un aviso "DESLIZÁ PARA VER MÁS" al pie. Nuevo botón RECARGAR
(arriba del radar, sobre SOBRE): busca actualización del service worker, borra cachés y recarga la app.

NOTA v37: la franja dice "● TRÁFICO ORBITAL · SGP4 · ARGENTINO". El botón RECARGAR queda justo encima de SOBRE
(bottom: calc(5.8% + 40px) = alto de SOBRE 34 px + 6 px de margen).

NOTA v38 (identificación instantánea al cruzar la línea amarilla):
- Antes la lista "sobre Argentina" se actualizaba cada 5 s (≈ 39 km de retraso a 7,8 km/s). Ahora el cruce se evalúa en
  CADA cuadro con el mismo contorno que dibuja la línea amarilla (overAR): retraso ≈ 10 ms. Al entrar se actualiza la
  lista, el marcador destella en amarillo y se despliega Identificados con todos sus datos (nombre, NORAD, tipo, hora de
  ingreso, posición, órbita, provincia, ruta).
- Vigilancia previa (WATCH_BOX ≈ 400 km alrededor de Argentina): todo satélite del catálogo que se acerca se dibuja ANTES
  de entrar, aunque no entre en el tope de 55; se quita al alejarse.
- No pisa lo que estás mirando: si ya tenés un satélite seleccionado que sigue sobre Argentina, solo se actualiza la lista.

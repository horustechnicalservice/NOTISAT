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

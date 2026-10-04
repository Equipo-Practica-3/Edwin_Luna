# Correcciones del proyecto "insect"

Este paquete contiene SOLO los archivos modificados, con la misma estructura de carpetas.
Cópialos encima de tu proyecto (`insect/insect/`), reemplazando los existentes.

## Cómo levantarlo

```bash
# 1) Copia los archivos de este zip sobre tu proyecto
# 2) Borra lo viejo para que no choque con el package.json nuevo
rm -rf node_modules package-lock.json
# 3) Crea tu .env a partir del ejemplo y pon claves NUEVAS
cp .env.example .env
# 4) Instala y alinea versiones con Expo SDK 52
npm install
npx expo install --fix
# 5) Arranca limpiando caché
npx expo start -c
```

Si ya habías corrido la app antes, desinstálala (o borra sus datos) del teléfono para empezar limpio.
El código nuevo también limpia por sí solo los insectos duplicados de la base de datos.

## Qué se cambió y por qué

| Archivo | Cambio |
|---|---|
| `src/components/menu.jsx` | `NavigationContainer independent` (evita el error de contenedor anidado con expo-router). La barra de pestañas se oculta en subpantallas con `getFocusedRouteNameFromRoute` (antes `tabBarStyle` dentro del Stack no hacía nada). Pantallas del stack definidas en una lista. "Coordenadas" ahora tiene encabezado con botón atrás. |
| `app/(tabs)/index.tsx` | Importa `react-native-gesture-handler` primero (lo exige el stack) y captura errores al iniciar la BD. |
| `src/databases/index.js` | Una sola conexión y una sola inicialización compartida (`getAllInsex` espera a que la BD esté lista). Corrige `> 60` (reinsertaba los 30 insectos en cada arranque). Limpia duplicados existentes + índice único + `INSERT OR IGNORE`. Siempre devuelve un arreglo. Los 30 insectos se conservan idénticos. |
| `src/PhotoContext.js` | Cada foto guarda su propia ubicación y título (adiós a los arreglos paralelos desincronizados). Copia la foto a almacenamiento permanente. Guarda automáticamente (también los títulos). Migra el formato anterior. `deletePhoto` ahora recibe el `id`. |
| `src/screens/cameraScreen.jsx` | Espera los permisos antes de abrir la cámara; `try/catch` para GPS y galería (si se niegan, la foto se guarda igual); sin recursión; `mediaTypes: ['images']`; botón "Volver al inicio". |
| `src/screens/subScreens/maps.jsx` | **Mapa sin Google**: ahora usa MapLibre + OpenFreeMap (gratis, sin API key ni cuenta), el mismo estilo de tu proyecto Kotlin, dentro de un `WebView` (funciona en Expo Go y en el APK). Los títulos editados se guardan; si no hay GPS centra en Durango. Requiere internet para los mosaicos. |
| `src/components/cordenadas.jsx` | CSV con comillas escapadas, usa las fotos con ubicación, quita el import innecesario del mapa. |
| `src/screens/userscreen.jsx` | Usa `id` en vez de índice, borde de color fijo (antes aleatorio en cada render), navega a `PredictionScreen` a través de la pestaña Home. |
| `src/screens/PredictionScreen.jsx` | Hooks antes de cualquier `return`. Clave de Clarifai desde `.env`. La información del insecto ahora viene de **Wikipedia** (API gratuita, sin clave; reemplaza a Google Custom Search, que está cerrada a clientes nuevos y se descontinúa el 1/ene/2027). Lee la imagen con `expo-file-system`. Valida la respuesta de Clarifai y muestra el error real. Cancela peticiones al salir. |
| `src/config.js` (nuevo) | Lee las claves de variables de entorno. |
| `src/components/InsectFound.jsx` | Una sola carga de datos (antes había dos), `keyExtractor` estable. |
| `src/screens/subScreens/Insectounded.jsx` | Hooks antes del `return` de error; quita `insect.categoria` (no existe, mostraba "undefined"). |
| `src/screens/Homescreen.jsx` | Quita `getAllInsex()` que se ejecutaba en cada render. |
| `app.json` | Plugins con textos de permisos (location, image-picker, media-library, sqlite). Quita permisos de almacenamiento obsoletos y el `projectId` de EAS (el proyecto ya no está atado a ninguna cuenta de Expo). |
| `package.json` | Quita `gesture-handler`, `vector-icons`, `react-native-vector-icons` y `react-native-maps` (ya no se usa). |
| `eas.json` | Deja solo los perfiles `preview` (APK) y `production`. |
| `.gitignore`, `.env.example` (nuevo) | Ignora `.env`. |
| `src/databases/enciclopedia.js` (nuevo) | Datos de la Insectopedia de Durango en JS (6 familias, 30 especies legibles) + funciones `getFamilias`, `getFamilia`, `getEspecie`, `buscarEspecies`. Ver aviso abajo. |

## Pendiente de tu parte

1. **Regenera la clave de Clarifai** (y la de Google Custom Search, ya sin uso, bórrala en Google Cloud): estuvieron escritas en el código y deben considerarse comprometidas. Aun con `.env`, las variables `EXPO_PUBLIC_*` viajan dentro de la app; para producción conviene un backend que haga esas llamadas.
2. **Borra `app.config.js`** si ya lo habías copiado de la versión anterior del zip (ya no hace falta, el mapa no usa clave de Google).
3. **Limpieza opcional** (no la hice porque son archivos, no código): borrar `countries/countries+states+cities.json` (37 MB, sin uso), `src/data/Insectos.DB/InsectosDB.db` (sin uso), la carpeta duplicada `image/imagedb` o `assets/imagedb`, y no comprimir `node_modules` ni `.git` al compartir el proyecto.
4. Los nombres de imagen con tilde/ñ (`Sofía-…`, `Abeja-melífera`, `maíz`, `caña`) funcionan, pero pueden dar problemas entre sistemas; si algún día fallan, renómbralos sin acentos y actualiza `imageMap.js` y la BD.

## Qué se probó

- El proyecto completo se compiló con Metro para Android (`expo export`) sin errores de imports ni sintaxis.
- La lógica SQL (limpieza de duplicados e inserción sin repetir) se probó aparte: 90 filas duplicadas → 30, y al reabrir se mantiene en 30.
- NO se probó en un teléfono o emulador (cámara, GPS, mapa y las APIs externas dependen de dispositivo y de tus claves).

## Sin cuenta de Expo / EAS

El mensaje "Log in to EAS" aparece al correr `eas build` (y a veces con `expo start --tunnel`). EAS Build **siempre** exige una cuenta de Expo; no se puede quitar con configuración. Alternativas sin cuenta:

- **Desarrollo:** `npx expo start` (sin `--tunnel`), con el teléfono y la PC en la misma red Wi-Fi, y abrir con Expo Go.
- **APK sin EAS (build local):** necesita JDK 17 y Android SDK (el que ya usas con Android Studio).
  ```bash
  npx expo prebuild --platform android
  cd android
  ./gradlew assembleRelease      # en Windows: gradlew.bat assembleRelease
  # APK en android/app/build/outputs/apk/release/app-release.apk
  ```
  O directo en un teléfono conectado: `npx expo run:android --variant release`.

## Enciclopedia: datos incompletos en el PDF

El PDF que me pasaste tiene las tablas de especies recortadas por el margen derecho. Lo que se pudo rescatar:

| Familia | Estado |
|---|---|
| Scarabaeidae | 17/17 completas (nombre científico, común y hábitat). En el PDF están impresas bajo el encabezado "Curculionidae"; son escarabajos de la familia Scarabaeidae. |
| Nymphalidae | 13/17 (solo nombre científico). *Junonia coenia* aparece cortada en el PDF ("Junonia coe") y se completó; verificar. |
| Curculionidae, Formicidae, Acrididae, Apidae | Sin especies legibles (`especies: []`). |

La información de las 6 familias (peligro, estatus de plaga, combate, estrategias, biomas) sí está completa.

# Panel del negocio — cómo activarlo (guía paso a paso)

Esta tarjeta ya trae un **Panel del negocio** (botón "⚙️ Panel del negocio" al final de la tarjeta) desde donde el dueño del restaurante puede editar precios, agregar/quitar productos, cambiar el horario, la galería, los métodos de pago y los datos de contacto — **sin tocar código**.

Tiene dos modos:

| Modo | Qué pasa cuando el dueño guarda cambios | Setup necesario |
|---|---|---|
| **Demo (por defecto)** | Se guarda solo en el navegador/dispositivo donde editó. Si un cliente abre el link en su celular, no ve los cambios. Sirve para mostrarle la tarjeta a un cliente antes de vender. | Ninguno, ya funciona tal cual subiste el archivo. |
| **Nube (recomendado para vender)** | Se guarda en una base de datos gratuita (Firebase). Todos los que abran el link ven el cambio al instante, desde cualquier dispositivo. | 10-15 minutos, una sola vez para todo tu negocio de tarjetas (no por cada cliente). |

---

## Parte 1 — Activar el modo nube (una sola vez para todos tus clientes)

### Paso 1. Crea un proyecto de Firebase (gratis)
1. Entra a [console.firebase.google.com](https://console.firebase.google.com) con la misma cuenta de Google que usas para GitHub.
2. Clic en **"Crear un proyecto"**. Ponle el nombre de tu agencia (ej. "Tarjetas Digitales"). No necesitas Google Analytics, puedes desactivarlo.
3. Espera a que se cree (unos segundos).

Este proyecto lo usarás para **todos** tus clientes futuros — no crees uno nuevo por cada restaurante.

### Paso 2. Activa Authentication (para que solo el dueño pueda editar)
1. En el menú izquierdo: **Build → Authentication → Get started**.
2. En la pestaña "Sign-in method", activa el proveedor **"Correo electrónico/contraseña"**.

### Paso 3. Activa Firestore Database (donde se guardan los menús)
1. Menú izquierdo: **Build → Firestore Database → Crear base de datos**.
2. Elige **"Modo producción"** y la región más cercana (ej. `southamerica-east1`).
3. Ve a la pestaña **"Reglas"** y reemplaza el contenido por esto:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /restaurants/{slug} {
      allow read: if true;
      allow create: if request.auth != null
                    && request.resource.data.config.ownerEmail == request.auth.token.email;
      allow update: if request.auth != null
                    && resource.data.config.ownerEmail == request.auth.token.email;
      allow delete: if false;
    }
  }
}
```

Esto significa: **cualquiera puede ver el menú** (para que los clientes lo consulten sin iniciar sesión), pero **solo el correo registrado como dueño de ese restaurante puede editarlo**. Clic en "Publicar".

### Paso 4. Copia tu configuración de Firebase
1. Clic en el ícono ⚙️ (arriba a la izquierda) → **"Configuración del proyecto"**.
2. Baja hasta "Tus apps" → clic en el ícono `</>` (web) → ponle un nombre (ej. "tarjetas") → **"Registrar app"**.
3. Firebase te muestra un bloque `firebaseConfig = {...}`. Copia esos valores.
4. Ábrelos en `index.html` y reemplaza el bloque `FIREBASE_CONFIG` (cerca de la línea 480) con tus valores reales:

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "tu-proyecto.firebaseapp.com",
  projectId: "tu-proyecto",
  storageBucket: "tu-proyecto.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

Este bloque es el mismo para **todas** las tarjetas que vendas (mismo proyecto Firebase, distintos clientes). Guarda este `index.html` como tu **plantilla base** y parte de ahí cada vez que crees una tarjeta nueva.

---

## Parte 2 — Por cada cliente nuevo (repite esto cuando vendas una tarjeta)

1. Copia tu plantilla base (la del Paso 4) y personalízala como ya haces hoy (nombre, menú, colores, fotos — puedes seguir usando Gemini para esto).
2. Cambia el identificador único del negocio, cerca de `RESTAURANT_ID`:
   ```js
   const RESTAURANT_ID = "nombre-del-restaurante"; // minúsculas, sin espacios ni tildes
   ```
3. Decide qué correo va a usar el dueño para entrar a su panel, y ponlo en el archivo en `DEFAULT_CONFIG.ownerEmail`:
   ```js
   ownerEmail: "correo-del-dueno@ejemplo.com",
   ```
4. En Firebase Console → **Authentication → Users → Add user**, crea ese mismo correo con una contraseña (esta se la entregas al cliente, junto con el link de su tarjeta).
5. Sube el `index.html` a un repositorio nuevo en GitHub y publícalo con GitHub Pages, tal como ya lo vienes haciendo.
6. La primera vez que el dueño entra al Panel del negocio, verá los datos con los que tú dejaste la tarjeta (su "semilla"). En cuanto le dé **Guardar cambios** una vez, esos datos quedan en la nube y desde ahí todo lo que edite se actualiza para todos sus clientes en vivo.

Ese es todo el trabajo extra por cliente: cambiar `RESTAURANT_ID`, poner su correo, y crear su usuario en Authentication. El resto (base de datos, reglas de seguridad, panel de edición) ya está resuelto en la plantilla.

---

## Cómo usa el panel el dueño del restaurante

1. Abre el link de su tarjeta (el mismo que ya usan sus clientes).
2. Toca **"⚙️ Panel del negocio"** al final de la tarjeta.
3. Ingresa el correo y contraseña que tú le diste.
4. Puede editar: nombre, frase, color de marca, foto de portada/avatar, teléfono, WhatsApp, dirección, Instagram, costo de domicilio, horarios, métodos de pago, galería de fotos, y **el menú completo**: agregar categorías, agregar/editar/eliminar productos y precios.
5. Toca **"💾 Guardar cambios"**. Los cambios se ven al instante para cualquiera que abra el link.

No necesita saber nada de código ni volver a pedirte que le edites algo — para cambios de precios, agregar un producto nuevo, o poner "agotado" quitándolo temporalmente, ya no dependen de ti.

---

## Nota sobre seguridad

Este sistema usa autenticación real de Firebase (no solo un PIN visible en el código), así que es razonablemente seguro para un negocio pequeño/mediano: cada dueño solo puede editar su propio restaurante, con su propio correo y contraseña. No es un sistema bancario de alta seguridad, pero es adecuado para el caso de uso (evitar que cualquier visitante edite el menú).

El **modo demo** (sin Firebase configurado) usa solo un PIN comparado en el navegador — útil para pruebas y demostraciones, pero no debe considerarse seguro ni usarse como el modo definitivo de un cliente que ya está pagando por el servicio.

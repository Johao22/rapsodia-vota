# 🎤 Rapsodia Vota — Sistema de Votación en Vivo

Sistema de votación del público en tiempo real para eventos de Rapsodia Perú.
El público escanea un QR, vota desde su celular, y los resultados se muestran
en vivo en la pantalla LED del evento.

## Arquitectura

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────┐
│  📱 Público  │────▶│  Firebase RTDB   │◀────│  🖥️ Pantalla │
│  vota.html   │     │  (gratis)        │     │ pantalla.html│
└─────────────┘     └──────────────────┘     └─────────────┘
                            ▲
                            │
                     ┌──────┴──────┐
                     │ 🎛️ Operador │
                     │  index.html  │
                     └─────────────┘
```

**3 páginas, 1 base de datos:**
- `index.html` → Panel de control (operador del evento)
- `vota.html` → Página de votación (público, móvil)
- `pantalla.html` → Display para LED 3x2m (pantalla completa)

## Despliegue Paso a Paso

### Paso 1: Crear proyecto en Firebase (5 min)

1. Ve a [console.firebase.google.com](https://console.firebase.google.com)
2. Click **"Agregar proyecto"** → nombre: `rapsodia-vota` → crear
3. En el dashboard, click **"Realtime Database"** en el menú izquierdo
4. Click **"Crear base de datos"**
5. Selecciona ubicación → **"Iniciar en modo de prueba"** → Habilitar
6. Ve a **Reglas** de la base de datos y pega esto:

```json
{
  "rules": {
    "rapsodia_event": {
      ".read": true,
      ".write": true,
      "voters": {
        "$roundId": {
          "$voterId": {
            ".write": "!data.exists()"
          }
        }
      }
    }
  }
}
```

> Las reglas permiten lectura pública (para la pantalla y votantes)
> y escritura libre. La regla de voters asegura que cada persona
> solo pueda votar una vez por batalla.

7. Ve a **Configuración del proyecto** (engranaje arriba a la izquierda)
8. En la sección **"Tus apps"**, click el ícono **"</>"** (Web)
9. Nombre: `rapsodia-vota` → Registrar app
10. **Copia los datos de `firebaseConfig`** — los necesitas en el paso 3

### Paso 2: Fork del repositorio + GitHub Pages (3 min)

**Opción A: Si ya tienes el repo clonado**
1. Sube los archivos a tu repo de GitHub
2. Ve a Settings → Pages → Source: `main` branch → `/ (root)` → Save
3. Espera 1-2 minutos → tu sitio estará en `https://tuusuario.github.io/rapsodia-vota`

**Opción B: Crear repo desde cero**
```bash
cd rapsodia-vota
git init
git add .
git commit -m "Rapsodia Vota v1"
git remote add origin https://github.com/TUUSUARIO/rapsodia-vota.git
git branch -M main
git push -u origin main
```
Luego activa Pages: Settings → Pages → Source: `main` → Save

### Paso 3: Configurar credenciales (2 min)

Edita `js/config.js` con los datos de Firebase del paso 1:

```javascript
const FIREBASE_CONFIG = {
  apiKey: "AIzaSyD...",            // Tu API key
  authDomain: "rapsodia-vota.firebaseapp.com",
  databaseURL: "https://rapsodia-vota-default-rtdb.firebaseio.com",
  projectId: "rapsodia-vota",
  storageBucket: "rapsodia-vota.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc..."
};

// Cambia esto por tu URL de GitHub Pages
const BASE_URL = "https://tuusuario.github.io/rapsodia-vota";
```

Haz commit y push:
```bash
git add js/config.js
git commit -m "Firebase config"
git push
```

### Paso 4: Verificar (1 min)

1. Abre `https://tuusuario.github.io/rapsodia-vota/` → Panel de control
2. Abre `https://tuusuario.github.io/rapsodia-vota/pantalla.html` → Pantalla LED
3. Abre `https://tuusuario.github.io/rapsodia-vota/vota.html` → Vista móvil

Las 3 páginas deben mostrar "Conectado" a Firebase.

---

## Uso el Día del Evento

### Antes del evento
1. Abre el **panel de control** (`index.html`) en tu celular o laptop
2. Agrega todas las batallas del día (MC1 vs MC2, fase)
3. Abre **pantalla.html** en la laptop conectada a la pantalla LED
4. Pon esa pestaña en **pantalla completa** (F11)

### Durante cada batalla

| Momento | Acción del operador | Lo que pasa |
|---------|-------------------|-------------|
| Host presenta la batalla | Click en la batalla en el panel | Pantalla muestra MC1 vs MC2 |
| "¡Saquen sus celulares!" | Click **"Mostrar QR"** | QR grande aparece en pantalla LED |
| Público escanea (~10 seg) | Esperar | La gente abre vota.html |
| "¡Voten ahora!" | Click **"Abrir votación"** | Countdown inicia, QR se achica, votos suben |
| Countdown llega a 0 | Automático | Votación se cierra |
| "Veamos qué dijo el público" | Click **"Mostrar resultados"** | Animación de ganador en pantalla |
| Siguiente batalla | Click en la siguiente | Se reinicia todo |

### Tips para el operador
- Practica la secuencia completa al menos 2 veces antes del evento
- Ten el panel en el celular, NO en la laptop de la pantalla
- Si algo falla, click "Reiniciar batalla actual" y vuelve a empezar
- El WiFi del venue es crítico — lleva un router portátil de respaldo
- Configura la duración de votación: 30s para octavos, 45s para final

---

## Estructura de archivos

```
rapsodia-vota/
├── index.html        ← Panel de control (operador)
├── vota.html         ← Votación móvil (público)
├── pantalla.html     ← Display LED (pantalla completa)
├── js/
│   └── config.js     ← Credenciales Firebase + URL
└── README.md
```

## Requisitos técnicos el día del evento

- **Internet**: WiFi estable en el venue (crítico)
- **Laptop**: Conectada a la pantalla LED vía HDMI, con Chrome abierto
- **Celular del operador**: Con el panel de control abierto
- **Los asistentes**: Solo necesitan celular con internet (datos móviles funciona)

## Personalización

### Cambiar colores
En cada archivo HTML, busca las variables CSS al inicio:
```css
--red: #E24B4A;    /* Color del MC 1 (izquierda) */
--blue: #378ADD;   /* Color del MC 2 (derecha) */
--amber: #EF9F27;  /* Timer / alertas */
```

### Cambiar duración de votación
En el panel de control hay un campo "Duración" antes de abrir votación.
El default es 30 segundos.

### Agregar logo
Reemplaza el texto "RAPSODIA" en pantalla.html con una imagen:
```html
<img src="img/logo.png" height="60" alt="Rapsodia">
```

---

## Seguridad

- Cada votante tiene un ID único generado en su navegador
- Un votante solo puede votar 1 vez por batalla (validado en Firebase)
- Las reglas de Firebase impiden sobreescribir votos existentes
- El panel de control no requiere login (proteger la URL)

### Para producción (después del 13 de junio)
- Agregar autenticación al panel de control
- Cambiar reglas de Firebase a modo restringido
- Considerar un dominio propio (rapsodia.peru/vota)

---

Hecho con 🎤 para Rapsodia Perú

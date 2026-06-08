# Setup: Firebase + GitHub (deploy automático)

Flujo final: hacés `git push` a `main` → GitHub Action → deploy solo a Firebase Hosting.
Hacés esto **una sola vez**. Después, cada cambio se publica automáticamente.

---

## 1. Crear el proyecto en Firebase

1. Entrá a https://console.firebase.google.com y tocá **"Crear un proyecto"**.
2. Nombre: por ejemplo `arma-valija`. (Google te asigna un **Project ID**, ej. `arma-valija-1a2b3` — anotalo, lo vas a usar.)
3. Podés desactivar Google Analytics, no hace falta.

## 2. Registrar la app web y copiar el config

1. En el proyecto, tocá el ícono **`</>`** (Agregar app web).
2. Apodo: `valija-web`. **No** marques "Firebase Hosting" todavía (lo hacemos por GitHub).
3. Te muestra un objeto `firebaseConfig`. Copialo.
4. Abrí `public/index.html`, buscá la línea `const FIREBASE_CONFIG = null;` y reemplazala por tu config:

```js
const FIREBASE_CONFIG = {
  apiKey: "...",
  authDomain: "arma-valija-1a2b3.firebaseapp.com",
  projectId: "arma-valija-1a2b3",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

## 3. Activar login con Google

1. En el menú izquierdo: **Compilación → Authentication → Comenzar**.
2. Pestaña **Sign-in method** → habilitá **Google** → Guardar.

## 4. Crear la base de datos (Firestore)

1. **Compilación → Firestore Database → Crear base de datos** (modo producción, región más cercana, ej. `southamerica-east1`).
2. Pestaña **Reglas** → pegá esto y publicá (cada usuario solo accede a sus datos):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /usuarios/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

## 5. Poner tu Project ID en los archivos

Reemplazá `TU_PROJECT_ID` por tu Project ID real en **dos** archivos:

- `.firebaserc`
- `.github/workflows/deploy.yml`

## 6. Crear el repo de GitHub y subir todo

```bash
cd "Armador de equipaje"
git init
git add .
git commit -m "App armar valija + deploy Firebase"
git branch -M main
# Creá el repo en https://github.com/new (ej. arma-valija) y luego:
git remote add origin https://github.com/TU_USUARIO/arma-valija.git
git push -u origin main
```

## 7. Conectar GitHub con Firebase (el secret de deploy)

La forma más fácil — necesitás Node y la CLI de Firebase:

```bash
npm install -g firebase-tools
firebase login
cd "Armador de equipaje"
firebase init hosting:github
```

Cuando pregunte:
- Repo: `TU_USUARIO/arma-valija`
- "Set up the workflow to run a build script before deploy?" → **No**
- "Set up automatic deployment on merge to main?" → **No** (ya tenemos el workflow)
- Sobrescribir el workflow existente → **No**

Esto crea automáticamente el secret **`FIREBASE_SERVICE_ACCOUNT`** en tu repo de GitHub, que es lo único que le falta al Action para funcionar.

> Alternativa manual: en GitHub → Settings → Secrets and variables → Actions → New secret, nombre `FIREBASE_SERVICE_ACCOUNT`, valor = el JSON de una cuenta de servicio de Firebase (Console → ⚙️ → Cuentas de servicio → Generar nueva clave privada).

## 8. Autorizar el dominio para el login

En **Authentication → Settings → Authorized domains** agregá tu dominio de hosting:
`TU_PROJECT_ID.web.app` (y `TU_PROJECT_ID.firebaseapp.com`).

## 9. Probar el deploy

```bash
git add .
git commit -m "config firebase"
git push
```

Andá a la pestaña **Actions** del repo en GitHub: vas a ver el deploy corriendo. Cuando termine, tu app está en:

**https://TU_PROJECT_ID.web.app**

---

## De acá en adelante

Cada vez que yo (o vos) cambie algo y hagamos `git push` a `main`, se redeploya solo. No tenés que tocar nada más.

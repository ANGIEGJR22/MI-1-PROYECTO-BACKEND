# Simulación y resolución de un conflicto de merge (Git)

Este documento muestra **paso a paso** cómo provocar un conflicto entre dos ramas, cómo se ve el archivo con **marcadores de conflicto**, y cómo **resolverlo manualmente**. Está pensado para tu proyecto `mi-proyecto-backend`.

---

## 1) Preparación (en `main`)
```bash
# Estar en el repo del proyecto
cd mi-proyecto-backend

# Asegúrate de estar en main y actualizado
git checkout main
git pull origin main  # si tienes remoto
```

Crea un archivo base que ambas ramas modificarán al mismo tiempo (misma sección):
```bash
mkdir -p src/controllers
cat > src/controllers/user.controller.js << 'EOF'
// user.controller.js (versión base en main)

function getAllUsers(req, res) {
  // TODO: implementar acceso a BD
  res.send("Lista de usuarios (base)");
}

module.exports = { getAllUsers };
EOF

git add src/controllers/user.controller.js
git commit -m "base: user.controller.js en main"
git push origin main
```

---

## 2) Rama A: `feature/user-pagination`
```bash
git checkout -b feature/user-pagination
```

Edita **la misma función** en la **misma zona** para forzar el conflicto:
```bash
cat > src/controllers/user.controller.js << 'EOF'
// user.controller.js (cambio rama A: paginación)

function getAllUsers(req, res) {
  // Implementa paginación con query params page/limit
  const page = Number(req.query.page || 1);
  const limit = Number(req.query.limit || 10);
  // TODO: leer desde BD con skip/limit
  res.send(`Usuarios paginados — page=${page}, limit=${limit}`);
}

module.exports = { getAllUsers };
EOF

git add src/controllers/user.controller.js
git commit -m "feat(pagination): getAllUsers con paginación"
git push -u origin feature/user-pagination
```

Vuelve a `main` y **fusiona** la rama A:
```bash
git checkout main
git merge feature/user-pagination
git push origin main
```

---

## 3) Rama B: `feature/user-filtering` (desde `main` antes del merge de A)
Para simular el conflicto, imagina que B se creó **antes** de fusionar A (o créala desde el commit base):

```bash
# Volver al commit base (antes del merge A) opcional, o simplemente crear nueva rama y editar igual zona
git checkout -b feature/user-filtering HEAD~1  # si el historial lo permite
# Si no es posible, puedes crear la rama y forzar el conflicto editando la misma zona igualmente:
# git checkout -b feature/user-filtering
```

Edita **la misma función** y zona, pero con otra lógica:
```bash
cat > src/controllers/user.controller.js << 'EOF'
// user.controller.js (cambio rama B: filtrado)

function getAllUsers(req, res) {
  // Implementa filtrado por rol a través de ?role=admin|user
  const role = String(req.query.role || "all");
  // TODO: filtrar en la BD por role
  res.send(`Usuarios filtrados — role=${role}`);
}

module.exports = { getAllUsers };
EOF

git add src/controllers/user.controller.js
git commit -m "feat(filtering): getAllUsers con filtrado por rol"
git push -u origin feature/user-filtering
```

---

## 4) Generar el **conflicto** al fusionar B en `main`
```bash
git checkout main
git merge feature/user-filtering
```

Verás algo como:
```
Auto-merging src/controllers/user.controller.js
CONFLICT (content): Merge conflict in src/controllers/user.controller.js
Automatic merge failed; fix conflicts and then commit the result.
```

Abre el archivo para ver los **marcadores de conflicto**:

```txt
// user.controller.js
<<<<<<< HEAD
// user.controller.js (cambio rama A: paginación)

function getAllUsers(req, res) {
  // Implementa paginación con query params page/limit
  const page = Number(req.query.page || 1);
  const limit = Number(req.query.limit || 10);
  // TODO: leer desde BD con skip/limit
  res.send(`Usuarios paginados — page=${page}, limit=${limit}`);
}

module.exports = { getAllUsers };
=======
// user.controller.js (cambio rama B: filtrado)

function getAllUsers(req, res) {
  // Implementa filtrado por rol a través de ?role=admin|user
  const role = String(req.query.role || "all");
  // TODO: filtrar en la BD por role
  res.send(`Usuarios filtrados — role=${role}`);
}

module.exports = { getAllUsers };
>>>>>>> feature/user-filtering
```

Los marcadores significan:
- `<<<<<<< HEAD` → la versión que hay en `main` (con paginación).
- `=======` → separador entre ambas versiones.
- `>>>>>>> feature/user-filtering` → la versión entrante (con filtrado).

---

## 5) **Resolución manual**

### Objetivo: combinar **paginación + filtrado** en una sola versión.
Edita el archivo y deja una versión final **sin marcadores**:

```javascript
// user.controller.js (resuelto: paginación + filtrado)

function getAllUsers(req, res) {
  // Filtrado por rol
  const role = String(req.query.role || "all");

  // Paginación
  const page = Number(req.query.page || 1);
  const limit = Number(req.query.limit || 10);

  // TODO: consultar a la BD aplicando filtro por role y paginación (skip/limit)
  res.send(`Usuarios — role=${role}, page=${page}, limit=${limit}`);
}

module.exports = { getAllUsers };
```

Guarda el archivo y marca el conflicto como resuelto:
```bash
git add src/controllers/user.controller.js
git commit -m "merge: resuelto conflicto en getAllUsers (paginación + filtrado)"
git push origin main
```

---

## 6) Consejos (opcional)
- Ver conflictos pendientes: `git status`
- Ver contenido con marcadores: `git diff`
- Usar una herramienta visual: `git mergetool` (requiere configurar p. ej. VSCode, Meld, etc.)
- Si te equivocaste, puedes abortar el merge y empezar de nuevo: `git merge --abort`

---

## 7) Resumen rápido de comandos
```bash
# Rama A
git checkout -b feature/user-pagination
# (editar) ...
git add -A && git commit -m "feat: paginación"
git checkout main && git merge feature/user-pagination && git push

# Rama B
git checkout -b feature/user-filtering HEAD~1  # o desde base equivalente
# (editar) ...
git add -A && git commit -m "feat: filtrado por rol"
git checkout main && git merge feature/user-filtering  # => conflicto

# Resolver a mano, luego:
git add src/controllers/user.controller.js
git commit -m "merge: resuelto conflicto (paginación + filtrado)"
git push origin main
```

---

**Resultado:** `main` contiene una versión combinada de `getAllUsers` con **paginación y filtrado**, y el merge conflict fue resuelto manualmente.

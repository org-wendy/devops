# 📌 Documentación Técnica – GitHub Actions Pipelines

Este repositorio contiene pipelines en **GitHub Actions** para la **gestión de usuarios** dentro de una organización de GitHub.  

Los pipelines permiten **alta**, **baja** y **auditoría** de usuarios con flujos estandarizados y automáticos.  

---

## 🚀 Requisitos Técnicos

- Se utilizó un **Classic Token de GitHub** con permisos de **Organization Admin** para poder agregar y quitar usuarios.  
- Los archivos YAML están diseñados para ser ejecutados directamente desde **GitHub Actions Workflows** en el repositorio.  
- Es necesario que el **token** esté configurado como **Secret** dentro del repositorio, por ejemplo:  
  - `ORG_ADMIN_TOKEN`

---

## 📂 Pipelines

### 1. **Alta de Usuario** (`altaUsuario.yml`)

**Objetivo:**  
Dar de alta a un nuevo usuario en la organización, asignándole un rol específico.  

**Parámetros requeridos:**  
- `email`: Correo electrónico del usuario a invitar.  
- `role`: Rol dentro de la organización (`admin` o `member`).  

**Pipeline completo:**  
```yaml
name: Alta Usuario

on:
  workflow_dispatch:
    inputs:
      email:
        description: "Correo del usuario a invitar"
        required: true
      role:
        description: "Rol del usuario (admin o member)"
        required: true

jobs:
  add-user:
    runs-on: ubuntu-latest
    steps:
      - name: Invitar usuario a la organización
        run: |
          gh api \
            --method POST \
            -H "Authorization: token ${{ secrets.ORG_ADMIN_TOKEN }}" \
            -H "Accept: application/vnd.github+json" \
            /orgs/${{ github.repository_owner }}/invitations \
            -f email='${{ github.event.inputs.email }}' \
            -f role='${{ github.event.inputs.role }}'
````

#### 🔧 Ejecución desde GitHub Actions

1. Ir a la pestaña **Actions** en el repositorio.
2. Seleccionar el workflow **Alta Usuario**.
3. Click en **Run workflow**.
4. Ingresar los parámetros:

   * **email:** `nuevo.usuario@ejemplo.com`
   * **role:** `member`
5. Click en **Run workflow**.
6. El sistema enviará la invitación automáticamente al correo indicado.

---

### 2. **Baja de Usuario** (`bajaUsuario.yml`)

**Objetivo:**
Dar de baja (remover) a un usuario existente de la organización.

**Parámetros requeridos:**

* `username`: Nombre de usuario en GitHub.

**Pipeline completo:**

```yaml
name: Baja Usuario

on:
  workflow_dispatch:
    inputs:
      username:
        description: "Nombre de usuario de GitHub a eliminar"
        required: true

jobs:
  remove-user:
    runs-on: ubuntu-latest
    steps:
      - name: Remover usuario de la organización
        run: |
          gh api \
            --method DELETE \
            -H "Authorization: token ${{ secrets.ORG_ADMIN_TOKEN }}" \
            -H "Accept: application/vnd.github+json" \
            /orgs/${{ github.repository_owner }}/members/${{ github.event.inputs.username }}
```

#### 🔧 Ejecución desde GitHub Actions

1. Ir a la pestaña **Actions**.
2. Seleccionar el workflow **Baja Usuario**.
3. Click en **Run workflow**.
4. Ingresar el parámetro:

   * **username:** `usuarioEjemplo`
5. Click en **Run workflow**.
6. El usuario será removido de la organización si existe.

---

### 3. **Auditoría de Permisos** (`auditoriaPermisos.yml`)

**Objetivo:**
Revisar y listar los permisos de los usuarios dentro de la organización.

**Parámetros requeridos:**
Ninguno.

**Pipeline completo:**

```yaml
name: Auditoría de Permisos

on:
  workflow_dispatch:

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - name: Generar CSV con miembros y roles
        run: |
          echo "login,role" > reporte_miembros.csv
          gh api \
            -H "Authorization: token ${{ secrets.ORG_ADMIN_TOKEN }}" \
            -H "Accept: application/vnd.github+json" \
            /orgs/${{ github.repository_owner }}/members --paginate \
            | jq -r '.[] | [.login, .role] | @csv' >> reporte_miembros.csv

          echo "📂 Archivo CSV generado: reporte_miembros.csv"
```

#### 🔧 Ejecución desde GitHub Actions

1. Ir a la pestaña **Actions**.
2. Seleccionar el workflow **Auditoría de Permisos**.
3. Click en **Run workflow** (no requiere parámetros).
4. Esperar la ejecución.
5. Descargar el artefacto `reporte_miembros.csv` desde la salida del job.

📄 **Ejemplo de CSV generado:**

```csv
login,role
usuario1,admin
usuario2,member
usuario3,member
```

---

## ✅ Buenas Prácticas

* Mantener el **token** con privilegios de organización en un secret seguro.
* Usar `workflow_dispatch` para ejecutar manualmente cada pipeline cuando sea necesario.
* Descargar y conservar el archivo CSV de auditoría como respaldo.

```


# Guía de Configuración de GitHub Classroom - S12

Esta guía te ayudará a configurar GitHub Classroom para distribuir el repositorio de estudiantes S12.

## 📋 Prerrequisitos

- Cuenta de GitHub con acceso a GitHub Classroom
- Permisos para crear repositorios en la organización
- Repositorio local `s12-estudiantes` con commits iniciales

## 🚀 Pasos de Configuración

### Paso 1: Crear Repositorio Template en GitHub

1. **Ir a GitHub** y crear un nuevo repositorio:
   - URL: https://github.com/new
   - **Nombre**: `s12-estudiantes-template` (o el nombre que prefieras)
   - **Descripción**: "Template repository para S12: Docker en CI/CD y Cloud Run"
   - **Visibilidad**: ⚠️ **Público** (necesario para GitHub Classroom)
   - ✅ Marcar como **Template repository**
   - ❌ NO inicializar con README, .gitignore o licencia

2. **Crear el repositorio** (sin inicializar contenido)

### Paso 2: Conectar Repositorio Local con GitHub

```bash
# Desde el directorio s12-estudiantes
cd "/Users/ogarcia/workspace/ASO-GIT/Curso 2025-2026/S11/S12/s12-estudiantes"

# Añadir remote (reemplaza TU_USUARIO y TU_ORG con tus valores)
git remote add origin https://github.com/TU_ORG/s12-estudiantes-template.git

# O si prefieres SSH:
# git remote add origin git@github.com:TU_ORG/s12-estudiantes-template.git

# Verificar remote
git remote -v
```

### Paso 3: Hacer Push del Código

```bash
# Push del código al repositorio remoto
git push -u origin main

# Si GitHub usa 'master' como rama por defecto:
# git branch -M main
# git push -u origin main
```

### Paso 4: Configurar como Template Repository

1. **Ir al repositorio en GitHub**
2. **Settings** → **Template repository**
3. ✅ Marcar **Template repository**
4. Guardar cambios

### Paso 5: Crear Asignación en GitHub Classroom

1. **Ir a GitHub Classroom**: https://classroom.github.com
2. **Seleccionar organización** donde está el repositorio
3. **Crear nueva asignación**:
   - Click en "New assignment" o "Crear nueva tarea"

4. **Configurar asignación**:
   - **Assignment title**: `S12 - Docker en CI/CD y Cloud Run`
   - **Repository visibility**: 
     - ✅ **Private** (recomendado para trabajo del estudiante)
   - **Add template repository**:
     - Seleccionar `TU_ORG/s12-estudiantes-template`
   - **Access permissions**:
     - ✅ **Allow repository admin access** (para que puedas ver el trabajo)
   - **Enable assignment invitation URL**:
     - ✅ Activar para generar URL de invitación

5. **Configurar opciones adicionales**:
   - **Deadline** (opcional): Fecha límite de entrega
   - **Enable feedback pull requests** (opcional): Para revisión de código
   - **Enable auto-grading** (opcional): Si tienes tests automatizados

6. **Crear asignación**

### Paso 6: Obtener URL de Invitación

Después de crear la asignación:

1. **Copiar la URL de invitación** que GitHub Classroom genera
2. **Compartir con estudiantes** mediante:
   - Aula virtual
   - Email
   - Mensajería del curso

### Paso 7: Verificar Configuración

1. **Probar la URL de invitación** con una cuenta de prueba
2. **Verificar que se crea el repositorio** correctamente
3. **Comprobar que el contenido** del template está presente

## 📝 Configuración Recomendada del Template

### Añadir Archivo `.github/classroom/autograding.json` (Opcional)

Si quieres configurar autograding, crea este archivo en el template:

```json
{
  "tests": [
    {
      "name": "Verificar estructura del repositorio",
      "setup": "",
      "run": "test -f guia-estudiante.md && test -d materiales",
      "input": "",
      "output": "",
      "comparison": "included",
      "timeout": 10,
      "points": 10
    }
  ]
}
```

### Añadir GitHub Actions para CI (Opcional)

Crear `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Verificar estructura
        run: |
          test -f guia-estudiante.md || exit 1
          test -d materiales || exit 1
          echo "✅ Estructura verificada"
```

## ✅ Checklist de Verificación

Antes de compartir con estudiantes, verifica:

- [ ] Repositorio template creado en GitHub
- [ ] Código pusheado correctamente
- [ ] Template repository marcado en GitHub
- [ ] Asignación creada en GitHub Classroom
- [ ] URL de invitación generada
- [ ] Repositorio template es público (o la organización permite acceso)
- [ ] Contenido del template está completo
- [ ] README.md es claro y útil para estudiantes

## 🔧 Troubleshooting

### Error: "Template repository not found"
- Verificar que el repositorio está marcado como template
- Verificar permisos de acceso
- Verificar que el repositorio es público o la organización tiene acceso

### Error: "Repository already exists"
- El estudiante ya aceptó la invitación anteriormente
- Eliminar el repositorio anterior o usar otro nombre

### Los estudiantes no ven el contenido del template
- Verificar que el template tiene commits
- Verificar que el template está marcado correctamente
- Verificar que GitHub Classroom tiene acceso al template

## 📚 Recursos Adicionales

- [GitHub Classroom Documentation](https://docs.github.com/en/education/manage-coursework-with-github-classroom)
- [Creating Template Repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository)
- [GitHub Classroom Guides](https://classroom.github.com/help)

## 🎯 Próximos Pasos Después de Configurar

1. **Compartir URL de invitación** con estudiantes
2. **Monitorear repositorios** creados en GitHub Classroom
3. **Revisar trabajo** de estudiantes en sus repositorios
4. **Proporcionar feedback** mediante pull requests o issues

---

**Nota**: Esta guía asume que ya tienes acceso a GitHub Classroom. Si no lo tienes, contacta con el administrador de tu organización educativa.


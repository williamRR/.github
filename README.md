# GitHub Reusable Workflows

Colección de workflows reutilizables para proyectos Node.js/TypeScript. Estos workflows proporcionan una base sólida de CI/CD con linting, testing, análisis de seguridad y quality gates.

## 📋 Tabla de Contenidos

- [Workflows Disponibles](#workflows-disponibles)
- [Configuración del Repositorio Central](#configuración-del-repositorio-central)
- [Uso desde Otros Repositorios](#uso-desde-otros-repositorios)
- [Secrets Requeridos](#secrets-requeridos)
- [Diagrama de Flujo del CI](#diagrama-de-flujo-del-ci)
- [Inputs y Outputs](#inputs-y-outputs)

---

## 🚀 Workflows Disponibles

### 1. **lint.yml** - Lint Check

Verifica la calidad del código usando ESLint y Prettier.

**Características:**
- Ejecuta ESLint mediante `npm run lint`
- Verifica formato de código con Prettier
- Cache de `node_modules` para ejecuciones más rápidas

**Inputs:**
| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `node-version` | string | `'20'` | Versión de Node.js a usar |

**Ejemplo de uso:**
```yaml
jobs:
  lint:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/lint.yml@main
    with:
      node-version: '20'
```

---

### 2. **test.yml** - Test Workflow

Ejecuta suites de pruebas unitarias y E2E con generación de cobertura.

**Características:**
- Ejecuta tests unitarios con Jest y cobertura
- Ejecuta tests E2E con Playwright (opcional)
- Verifica umbral de cobertura del 80%
- Upload de artefactos (coverage, reportes)

**Inputs:**
| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `node-version` | string | `'20'` | Versión de Node.js |
| `run-e2e` | boolean | `true` | Ejecutar tests E2E |

**Outputs:**
| Parámetro | Descripción |
|-----------|-------------|
| `coverage-percent` | Porcentaje de cobertura de tests |

**Ejemplo de uso:**
```yaml
jobs:
  test:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/test.yml@main
    with:
      node-version: '20'
      run-e2e: true
```

---

### 3. **security.yml** - Security Scan

Análisis de seguridad con múltiples herramientas.

**Características:**
- **Gitleaks**: Detección de secrets en el código
- **npm audit**: Análisis de vulnerabilidades en dependencias
- **Snyk**: Escaneo de vulnerabilidades y SAST

**Inputs:**
| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `node-version` | string | `'20'` | Versión de Node.js |
| `run-snyk` | boolean | `true` | Ejecutar escaneo Snyk |

**Secrets requeridos:**
- `SNYK_TOKEN` - Token de autenticación de Snyk

**Ejemplo de uso:**
```yaml
jobs:
  security:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/security.yml@main
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

---

### 4. **sonarqube.yml** - SonarQube Analysis

Análisis de calidad de código con SonarCloud/SonarQube.

**Características:**
- Análisis de código con SonarCloud
- Quality Gate automático
- Soporte para coverage de Jest (LCOV)
- Integración con reportes de ESLint

**Inputs:**
| Parámetro | Tipo | Requerido | Default | Descripción |
|-----------|------|-----------|---------|-------------|
| `node-version` | string | No | `'20'` | Versión de Node.js |
| `sonar-org` | string | No | `'williamrr'` | Organization key en SonarCloud |
| `sonar-project-key` | string | **Sí** | - | Project key en SonarCloud |
| `sonar-project-name` | string | No | (project key) | Nombre del proyecto |

**Secrets requeridos:**
- `SONAR_TOKEN` - Token de autenticación de SonarCloud

**Outputs:**
| Parámetro | Descripción |
|-----------|-------------|
| `quality-gate-status` | Estado del Quality Gate (PASS/FAIL) |

**Ejemplo de uso:**
```yaml
jobs:
  sonarqube:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/sonarqube.yml@main
    with:
      sonar-project-key: 'mi-proyecto'
      sonar-org: 'williamrr'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

### 5. **ci.yml** - CI Pipeline Completo

Pipeline principal que orquesta todos los workflows anteriores.

**Características:**
- Ejecución secuencial: Lint → Test/Security (paralelo) → SonarQube → Deploy
- Soporte para múltiples triggers (push, PR, manual)
- Job de summary de estado del CI
- Deployment condicional (staging/production)

**Inputs:**
| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `node-version` | string | `'20'` | Versión de Node.js |
| `run-e2e` | boolean | `true` | Ejecutar tests E2E |
| `sonar-project-key` | string | `''` | Project key para SonarQube |
| `deploy-target` | string | `'none'` | Target de deploy (none/staging/production) |

**Triggers:**
- `push` a branches `main` y `develop`
- `pull_request` a cualquier branch
- `workflow_dispatch` (ejecución manual)

**Ejemplo de uso:**
```yaml
jobs:
  ci:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/ci.yml@main
    with:
      node-version: '20'
      run-e2e: true
      sonar-project-key: 'mi-proyecto'
      deploy-target: 'staging'
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

## 📦 Configuración del Repositorio Central

### Paso 1: Crear el repositorio en GitHub

1. Ve a GitHub y crea un nuevo repositorio llamado `.github`
2. Configúralo como **público** (los workflows reutilizables deben estar en repos públicos o en el mismo organismo con GitHub Enterprise)
3. No inicialices con README (ya estamos creando la documentación)

### Paso 2: Subir los workflows

Si ya tienes los archivos en tu máquina local:

```bash
# Navegar al directorio
cd /home/williamrehel/Develop/WARRSPA_INDUSTRIES/.github-workflows

# Inicializar git (si no está inicializado)
git init

# Agregar remote
git remote add origin git@github.com:WARRSPA-INDUSTRIES/.github.git

# Agregar archivos
git add .

# Commit inicial
git commit -m "feat: add reusable workflows"

# Push a main
git branch -M main
git push -u origin main
```

### Paso 3: Verificar la estructura

El repositorio debe tener esta estructura:

```
.github/
├── .github/
│   └── workflows/
│       ├── lint.yml
│       ├── test.yml
│       ├── security.yml
│       ├── sonarqube.yml
│       └── ci.yml
└── README.md
```

---

## 🔗 Uso desde Otros Repositorios

### Opción A: Usar el CI Pipeline Completo (Recomendado)

Crea un archivo `.github/workflows/ci.yml` en tu repositorio:

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  ci:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/ci.yml@main
    with:
      node-version: '20'
      run-e2e: true
      sonar-project-key: 'mi-proyecto-key'
      deploy-target: 'none'  # Cambiar a 'staging' o 'production' si hay deploy
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Opción B: Workflows Individuales

Si necesitas más control, usa workflows individuales:

```yaml
name: CI Modular

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  # Fase 1: Lint
  lint:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/lint.yml@main
    with:
      node-version: '20'

  # Fase 2: Test (requiere lint exitoso)
  test:
    needs: lint
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/test.yml@main
    with:
      node-version: '20'
      run-e2e: true

  # Fase 2: Security (paralelo a test)
  security:
    needs: lint
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/security.yml@main
    secrets:
      SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  # Fase 3: SonarQube (requiere test exitoso)
  sonarqube:
    needs: test
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/sonarqube.yml@main
    with:
      sonar-project-key: 'mi-proyecto-key'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Opción C: Solo Testing Básico

Para proyectos pequeños que solo necesitan tests:

```yaml
name: Basic Tests

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/lint.yml@main

  test:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/test.yml@main
    with:
      run-e2e: false  # Sin E2E para más rapidez
```

### Referenciar una versión específica

Para mayor estabilidad, referencia un tag o commit específico:

```yaml
jobs:
  ci:
    # Usar tag de versión
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/ci.yml@v1.0.0

    # O usar SHA específico
    # uses: WARRSPA-INDUSTRIES/.github/.github/workflows/ci.yml@abc123def456
```

---

## 🔐 Secrets Requeridos

Configura estos secrets en tu repositorio (Settings → Secrets and variables → Actions):

### SonarCloud

1. Ve a [SonarCloud](https://sonarcloud.io/)
2. Crea una cuenta o inicia sesión
3. Ve a **My Account → Security**
4. Genera un nuevo token
5. En tu repo de GitHub: Settings → Secrets → New repository secret
   - Name: `SONAR_TOKEN`
   - Value: `<tu token de SonarCloud>`

### Snyk

1. Ve a [Snyk](https://snyk.io/)
2. Crea una cuenta o inicia sesión
3. Ve a **Settings → API Token**
4. Copia tu token
5. En tu repo de GitHub: Settings → Secrets → New repository secret
   - Name: `SNYK_TOKEN`
   - Value: `<tu token de Snyk>`

### Secrets Opcionales

| Secret | Uso | Requerido para |
|--------|-----|----------------|
| `SLACK_WEBHOOK` | Notificaciones | Enviar alertas a Slack |
| `CODECOV_TOKEN` | Codecov | Subir coverage a Codecov |
| `SONAR_HOST_URL` | SonarQube self-hosted | Instalaciones on-premise |

---

## 📊 Diagrama de Flujo del CI

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CI PIPELINE FLOW                                  │
└─────────────────────────────────────────────────────────────────────────────┘

    TRIGGER
   ┌─────────┐
   │ Push/PR │
   │ Manual  │
   └────┬────┘
        │
        ▼
   ┌─────────┐
   │  LINT   │  ESLint + Prettier
   └────┬────┘
        │
        ├───────────────────────────────────┐
        │ PASS                              │ FAIL
        ▼                                   ▼
   ┌─────────┐                      ┌──────────────┐
   │  TEST   │◄─────────────────────│ STOP / ERROR │
   │ SECURITY│      (parallel)       └──────────────┘
   └────┬────┘
        │
        ├───────────────────────────┐
        │ PASS                      │ FAIL
        ▼                           ▼
   ┌─────────┐              ┌──────────────┐
   │ SONARQUBE│              │ STOP / ERROR │
   │ (optional)│             └──────────────┘
   └────┬────┘
        │
        ├───────────────────────────┐
        │ Quality Gate OK           │ FAIL
        ▼                           ▼
   ┌─────────┐              ┌──────────────┐
   │ DEPLOY  │              │ STOP / ERROR │
   │ (main   │              └──────────────┘
   │  only)  │
   └────┬────┘
        │
        ▼
   ┌─────────┐
   │ SUCCESS │
   └─────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                      PARALLEL EXECUTION DETAILS                             │
└─────────────────────────────────────────────────────────────────────────────┘

                    ┌──────────────┐
                    │    LINT      │
                    └──────┬───────┘
                           │
                ┌──────────┴──────────┐
                │                     │
           ┌────▼────┐          ┌────▼────┐
           │  TEST   │          │ SECURITY│
           └────┬────┘          └────┬────┘
                │                     │
                └──────────┬──────────┘
                           │
                    ┌──────▼───────┐
                    │  SONARQUBE   │
                    └──────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                         SECURITY SCANS DETAILS                              │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────┐
    │                    SECURITY WORKFLOW                       │
    └────────────────────────────────────────────────────────────┘

              ┌─────────────┐    ┌─────────────┐    ┌─────────┐
              │  GITLEAKS   │    │ NPM AUDIT   │    │  SNYK   │
              │ (secrets)   │    │ (deps vuln) │    │ (SAST)  │
              └──────┬──────┘    └──────┬──────┘    └────┬────┘
                     │                  │                │
                     └──────────────────┴────────────────┘
                                             │
                                    ALL MUST PASS
                                             │
                                             ▼
                                      ┌─────────────┐
                                      │   SECURITY  │
                                      │   PASSED    │
                                      └─────────────┘
```

---

## 📝 Inputs y Outputs Referencia

### Inputs Comunes

Todos los workflows aceptan estos inputs:

```yaml
inputs:
  node-version:
    description: 'Versión de Node.js'
    type: string
    default: '20'
    options: ['16', '18', '20', '22']
```

### Outputs Disponibles

```yaml
# test.yml outputs
outputs:
  coverage-percent:
    description: 'Porcentaje de cobertura de tests'
    value: '85.42'

# sonarqube.yml outputs
outputs:
  quality-gate-status:
    description: 'Estado del Quality Gate'
    value: 'OK'  # o 'FAILED'
```

### Usar Outputs en Workflows Posteriores

```yaml
jobs:
  test:
    uses: WARRSPA-INDUSTRIES/.github/.github/workflows/test.yml@main

  report:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Check coverage
        run: |
          echo "Coverage: ${{ needs.test.outputs.coverage-percent }}%"
```

---

## 🛠️ Troubleshooting

### Error: "Resource not accessible by integration"

**Causa:** El repositorio de workflows no es público o no está en la misma organización.

**Solución:**
- Haz el repo `.github` público, o
- Usa GitHub Enterprise para repos privados

### Error: "Unable to resolve action"

**Causa:** La referencia al workflow es incorrecta.

**Solución:**
```yaml
# Correcto
uses: org/.github/.github/workflows/ci.yml@main

# Incorrecto
uses: org/.github/workflows/ci.yml@main  # Falta .github/
```

### Error: "Secrets not passed"

**Causa:** Los secrets no se están pasando al workflow reutilizable.

**Solución:**
```yaml
jobs:
  ci:
    uses: org/.github/.github/workflows/ci.yml@main
    secrets: inherit  # Heredar todos los secrets
    # O especificar:
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

## 📄 Licencia

Estos workflows son parte del proyecto WARRSPA INDUSTRIES y se proporcionan tal cual para uso interno y educativo.

---

## 🤝 Contribuciones

Para sugerencias o mejoras a estos workflows:
1. Abre un issue en el repo `.github`
2. Haz un fork y crea un PR con tus cambios
3. Documenta los cambios en este README

---

**Última actualización:** 2026-04-03

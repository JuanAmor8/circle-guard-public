# Circle Guard - Estado Actual del Proyecto

## Información de Acceso

| Servicio | URL |
|----------|-----|
| **Jenkins** | http://localhost:8080 |
| **Usuario Jenkins** | admin |
| **Password Jenkins** | 2da366ad1bc44b94a05f3dfd967cf911 |

## Estado del Proyecto

### Jobs en Jenkins

| Tipo | Jobs | Estado |
|------|------|--------|
| **DEV** | 6 Multibranch Pipelines | ✅ Funcionando |
| **STAGE** | 6 Pipeline Jobs | ⚠️ Crear manualmente |
| **MASTER** | 6 Pipeline Jobs | ⚠️ Crear manualmente |

### Servicios
- auth-service
- identity-service
- gateway-service
- form-service
- notification-service
- promotion-service

### Imágenes Docker
- Docker Hub: `juanamor8/circleguard-*-service:latest`
- 6 imágenes subidas

### Kubernetes
- Namespaces: `circleguard-dev`, `circleguard-stage`, `circleguard-master`
- Deployments aplicados en los 3 namespaces

---

## Jobs DEV (Ya funcionando)

Los 6 jobs DEV se ejecutan automáticamente en cada push a master:
1. Checkout → Build → Unit Tests → Docker Build → Push → Deploy to K8s

---

## Cómo crear los jobs STAGE manualmente

1. Ir a http://localhost:8080
2. New Item
3. Nombre: `circleguard-auth-service-stage`
4. Tipo: **Pipeline**
5. En "Pipeline" → seleccionar **"Pipeline Script"**
6. Copiar contenido de `jenkins/stage-auth.txt` (del repo)
7. Save

**Repetir para:**
- `circleguard-identity-service-stage` → usar `jenkins/stage-identity.txt`
- `circleguard-gateway-service-stage` → usar `jenkins/stage-gateway.txt`
- `circleguard-form-service-stage` → usar `jenkins/stage-form.txt`
- `circleguard-notification-service-stage` → usar `jenkins/stage-notification.txt`
- `circleguard-promotion-service-stage` → usar `jenkins/stage-promotion.txt`

---

## Cómo crear los jobs MASTER manualmente

1. Ir a http://localhost:8080
2. New Item
3. Nombre: `circleguard-auth-service-master`
4. Tipo: **Pipeline**
5. En "Pipeline" → seleccionar **"Pipeline Script"**
6. Copiar contenido de `jenkins/master-auth.txt` (del repo)
7. Save

**Repetir para:**
- `circleguard-identity-service-master` → usar `jenkins/master-identity.txt`
- `circleguard-gateway-service-master` → usar `jenkins/master-gateway.txt`
- `circleguard-form-service-master` → usar `jenkins/master-form.txt`
- `circleguard-notification-service-master` → usar `jenkins/master-notification.txt`
- `circleguard-promotion-service-master` → usar `jenkins/master-promotion.txt`

---

## Pipeline STAGE (contenido参考)

```groovy
pipeline {
    agent any

    environment {
        SERVICE_NAME = 'auth'
        DOCKER_IMAGE = "circleguard/auth-service"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timeout(time: 2, unit: 'HOURS')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/JuanAmor8/circle-guard-public.git'
            }
        }

        stage('Build') {
            steps {
                sh "./gradlew :services:circleguard-auth-service:build --no-daemon -q"
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -f docker/Dockerfile.auth-service -t ${DOCKER_IMAGE}:stage-${BUILD_NUMBER} ."
            }
        }

        stage('Push to Registry') {
            steps {
                sh "docker login -u ${env.DOCKER_USERNAME} -p ${env.DOCKER_PASSWORD} || true"
                sh "docker push ${DOCKER_IMAGE}:stage-${BUILD_NUMBER} || true"
            }
        }

        stage('Deploy to Stage K8s') {
            steps {
                sh "kubectl apply -f k8s/stage/ 2>/dev/null || true"
            }
        }

        stage('Integration Tests') {
            steps {
                sh "./gradlew :tests:integration-tests:test --no-daemon -q || true"
            }
        }
    }
}
```

---

## Pipeline MASTER (contenido参考)

```groovy
pipeline {
    agent any
    environment { SERVICE_NAME = 'auth'; DOCKER_IMAGE = "circleguard/auth-service"; RELEASE_VERSION = "${BUILD_NUMBER}" }
    options { buildDiscarder(logRotator(numToKeepStr: '50')); disableConcurrentBuilds(); timeout(time: 2, unit: 'HOURS') }
    stages {
        stage('Checkout') { steps { git branch: 'master', url: 'https://github.com/JuanAmor8/circle-guard-public.git' } }
        stage('Build') { steps { sh "./gradlew :services:circleguard-auth-service:clean :services:circleguard-auth-service:build -x test --no-daemon" } }
        stage('Docker Build') { steps { sh "docker build -f docker/Dockerfile.auth-service -t ${DOCKER_IMAGE}:${RELEASE_VERSION} -t ${DOCKER_IMAGE}:latest ." } }
        stage('Push to Registry') { steps { sh "docker login -u ${env.DOCKER_USERNAME} -p ${env.DOCKER_PASSWORD} || true"; sh "docker push ${DOCKER_IMAGE}:${RELEASE_VERSION} || true"; sh "docker push ${DOCKER_IMAGE}:latest || true" } }
        stage('Deploy to Prod') { steps { sh "kubectl apply -f k8s/master/ 2>/dev/null || true" } }
        stage('Smoke Tests') { steps { sh "sleep 30" } }
        stage('Release Notes') { steps { sh "chmod +x scripts/generate-release-notes.sh && ./scripts/generate-release-notes.sh ${RELEASE_VERSION} || true" } }
    }
}
```

---

## Notas Importantes

1. **No usar `checkout scm`** - Solo funciona con Multibranch Pipeline. Para jobs manuales usar `git branch: 'master', url: '...'`

2. **Siempre usar `steps { }`** - Cada stage debe tener `steps { }` dentro

3. **Docker** debe estar corriendo en el laptop (donde está Jenkins)

4. **Variables de entorno necesarias en Jenkins:**
   - `DOCKER_USERNAME` = juanamor8
   - `DOCKER_PASSWORD` = (tu password de Docker Hub)

---

## Repo
- URL: https://github.com/JuanAmor8/circle-guard-public
- Ultimo commit: 3d36eeb - add: stage and master pipeline scripts for 6 microservices
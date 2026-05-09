# Video Script - Circle Guard CI/CD Workshop (≤8 minutos)

## Duración Total: 8:00

---

### PARTE 1: INTRODUCCIÓN (0:00 - 0:30)
**[Pantalla: Logo Circle Guard + título]**

"Hola, en este video демuestro el pipeline CI/CD completo para Circle Guard, un sistema de 6 microservicios usando Jenkins, Docker y Kubernetes."

**[Corte rápido: Diagrama de arquitectura]**

"Arquitectura: auth → identity → form → promotion → notification → gateway, todo orquestado con Kafka y caches en Redis y Neo4j."

---

### PARTE 2: SETUP INFRAESTRUCTURA (0:30 - 1:30)
**[Mostrar terminal]**

1. "Primero, verificamos que Minikube está corriendo:"
```
kubectl get nodes
```
✅ Single node listo

2. "Vemos los pods del namespace dev después del deploy:"
```
kubectl get pods -n circleguard-dev
```
✅ 6 servicios + infra corriendo

3. "Services exposés:"
```
kubectl get svc -n circleguard-dev
```
✅ Servicios en puertos correctos

---

### PARTE 3: PIPELINE DEV (1:30 - 3:00)
**[Cambiar a Jenkins UI]**

1. "Pipeline DEV se ejecuta en cada push a develop. Mostrar jobs Multibranch Pipeline."

2. **Ejecutar job manualmente** (o mostrar build anterior)
   - Mostrar consola: stages ejecutándose
   - Build & Compile ✅
   - Unit Tests ✅ (27 tests)
   - Security Scan ✅
   - Docker Build ✅
   - Push to Registry ✅
   - Deploy to K8s ✅
   - Smoke Tests ✅

3. "Verificar en K8s:"
```
kubectl rollout status deployment/auth-service -n circleguard-dev
```
✅ Deployment actualizado

---

### PARTE 4: TESTS (3:00 - 4:30)
**[Mostrar consola + reportes]**

1. **Unit Tests (3:00-3:15)**
```
./gradlew :services:circleguard-auth-service:test
```
- Mostrar JUnit report HTML
- 27+ tests pasando

2. **Integration Tests (3:15-3:45)**
```
./gradlew :tests:integration-tests:test
```
- 8 tests contra servicios reales
- TestRestTemplate contra endpoints HTTP

3. **E2E Tests (3:45-4:15)**
```
cd tests/e2e && npx cypress run
```
- 23 tests Cypress API
- Login → Survey → Gate validation
- Mostrar результаты Cypress Dashboard o CLI output

4. **Performance Tests (4:15-4:30)**
```
locust -f locustfile.py --headless -u 50 -r 5 -t 2m --html report.html
```
- Mostrar HTML report generado
- Métricas: RPS, tiempos de respuesta

---

### PARTE 5: PIPELINE STAGE (4:30 - 6:00)
**[Jenkins UI]**

1. "Pipeline STAGE valida calidad antes de producción."

2. Mostrar job de stage:
   - Integration Tests ✅
   - E2E Tests ✅  
   - Performance Tests ✅
   - Approval Gate ⏳

3. "Approval Gate espera confirmación de QA."

4. **[Si hay tiempo]** Aprobar y mostrar:
   - Deploy a namespace stage
   - `kubectl get pods -n circleguard-stage`

---

### PARTE 6: PIPELINE MASTER + RELEASE NOTES (6:00 - 7:30)
**[Jenkins UI]**

1. "Pipeline MASTER es el último paso - desplega a producción."

2. Mostrar último build en master:
   - Security Scans ✅ (paralelo)
   - Push to Registry ✅
   - Deploy to Production ✅
   - Smoke Tests ✅

3. "Release Notes generadas automáticamente:"
```
cat RELEASE_NOTES.md
```

4. "Tag git creado:"
```
git tag -l
```
✅ v1.x.x

5. Mostrar changelog en GitHub releases

---

### PARTE 7: LOCUST REPORT + CIERRE (7:30 - 8:00)
**[Abrir report.html]**

1. Mostrar dashboard de Locust:
   - Total requests
   - Failures
   - Response time chart
   - RPS chart

2. **[Cierre]**
"Este taller cubrió los 6 puntos de la rúbrica:
- ✅ 10% Jenkins + Docker + Kubernetes
- ✅ 15% Pipeline DEV para 6 microservicios  
- ✅ 30% Unit + Integration + E2E + Performance tests
- ✅ 15% Pipeline STAGE en Kubernetes
- ✅ 15% Pipeline MASTER + Release Notes
- ✅ 15% Documentación + Video"

3. "Código disponible en el repositorio. ¡Gracias!"

**[Fin]**

# Laboratorio 3 - Despliegue CI/CD en Kubernetes

**Estudiante:** Rodrigo Videla  
**Curso:** Curso contenedores  
**Entorno de Validación:** Docker Desktop Kubernetes (Local)  
**Herramientas Clave:** Node.js (v22-alpine), pnpm, Jenkins (vía Docker local), Kubernetes  

---

## 1. Estructura del Proyecto
El laboratorio se organizó de forma separada manteniendo dos directorios principales al mismo nivel para no alterar el código base provisto:
* `/lab3`: Contiene el código fuente original de la aplicación desarrollado en NestJS.
* `/solucion-lab3`: Almacena todos los artefactos de infraestructura, automatización, manifiestos y evidencias requeridas para la entrega.

---

## 2. Componentes de Solución Incluidos

* **`Dockerfile`**: Configurado con un diseño multi-etapa utilizando la imagen base `node:22-alpine`. Resuelve la compilación con `pnpm` aislando dependencias de desarrollo de las de producción para optimizar el peso final de la imagen.
* **`.dockerignore`**: Evita la transferencia de archivos basura y la carpeta local `node_modules` hacia el contexto de Docker.
* **`entrega.yaml`**: Manifiesto unificado de Kubernetes parametrizado bajo el espacio de nombres (`Namespace`) `ns-rodrigo-videla`. Inyecta configuraciones dinámicas mediante un `ConfigMap` (`AMBIENTE`) y un `Secret` codificado en Base64 (`API_KEY`). Posee un `Deployment` con 2 réplicas y un `Service` tipo `ClusterIP`.
* **`agent.yaml`**: Definición de Pod de Kubernetes para que funcione como agente dinámico en Jenkins, aprovisionando contenedores individuales para `node`, `docker-cli` y `kubectl`.
* **`Jenkinsfile.Rodrigo-Videla`**: Pipeline declarativo que automatiza el flujo completo (Instalación -> Test -> Build -> Push -> Deploy) utilizando los contenedores del agente y gestionando de forma segura las credenciales de Docker Hub.

---

## 3. Instrucciones de Ejecución Manual y Validación

Para comprobar los artefactos de forma independiente antes de pasar por Jenkins, se ejecutaron los siguientes comandos dentro del directorio `solucion-lab3`:

### A. Construcción y Publicación en Docker Hub
```bash
# Iniciar sesión en el registro
docker login

# Compilar la imagen apuntando al contexto de la aplicación en ../lab3
docker build -f Dockerfile -t rodrigovidela/tarea-final:rodrigo-videla ../lab3

# Subir la imagen al repositorio público
docker push rodrigovidela/tarea-final:rodrigo-videla
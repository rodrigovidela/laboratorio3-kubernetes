# Laboratorio 3: Pipeline de CI/CD Automático con Jenkins y Kubernetes

**Estudiante:** Rodrigo Videla  
**Curso:** Curso contenedores  
**Entorno de Validación:** Docker Desktop Kubernetes (Local)  
**Herramientas Clave:** Node.js (v22-alpine), pnpm, Jenkins (vía Docker local), Kubernetes  

---

## 1. Estructura del Proyecto
El laboratorio se organizó manteniendo dos directorios principales al mismo nivel para no alterar el código base original provisto:
* `/lab3`: Contiene el código fuente original de la aplicación desarrollado en NestJS.
* `/solucion-lab3`: Almacena todos los artefactos de infraestructura, automatización, manifiestos y evidencias requeridas para la entrega.

---

## 2. Componentes de Solución Incluidos

* **`Dockerfile`**: Configurado con un diseño multi-etapa utilizando la imagen base `node:22-alpine`. Resuelve la compilación con `pnpm` aislando dependencias de desarrollo de las de producción para optimizar el peso final de la imagen.
* **`.dockerignore`**: Evita la transferencia de archivos basura y la carpeta local `node_modules` hacia el contexto de Docker.
* **`entrega.yaml`**: Manifiesto unificado de Kubernetes parametrizado bajo el espacio de nombres (`Namespace`) `ns-rodrigo-videla`. Inyecta configuraciones dinámicas mediante un `ConfigMap` (`config-rodrigo-videla`) y un `Secret` (`secret-rodrigo-videla`)[cite: 3]. Posee un `Deployment` con 2 réplicas y un `Service` tipo `ClusterIP`.
* **`agent.yaml`**: Definición de Pod de Kubernetes para que funcione como agente dinámico en Jenkins, aprovisionando contenedores individuales para `node`, `docker-cli` y `kubectl`.
* **`Jenkinsfile.Rodrigo-Videla`**: Pipeline declarativo que automatiza el flujo completo (Instalación ➡️ Test ➡️ Build ➡️ Push ➡️ Deploy) utilizando de manera eficiente los contenedores del agente y gestionando las credenciales de Docker Hub de forma segura.

---

## 3. Configuración de Credenciales en Jenkins

Para que la etapa de **Push** automatizada funcione de forma correcta, el pipeline requiere autenticarse de manera segura en Docker Hub sin exponer datos en texto plano. Registre las credenciales en Jenkins con los siguientes parámetros antes de ejecutar la compilación:

1. En el panel principal de Jenkins, diríjase a **Administrar Jenkins** -> **Credentials**.
2. Haga clic en el dominio **(global)** y seleccione **Add Credentials**.
3. Configure los campos con los siguientes datos exactos:
   * **Kind (Tipo):** Username with password
   * **Scope (Ámbito):** Global
   * **Username (Usuario):** `rvidela` *(Su usuario de Docker Hub)*
   * **Password (Contraseña):** *(Access Token personal de Docker Hub, si lo necesita solicitelo por favor)*
   * **ID:** `docker-hub-credentials`  
    *Nota : Este ID debe coincidir textualmente con el identificador invocado en el Jenkinsfile para evitar errores de resolución de credenciales.*

---

## 4. Etapas del Pipeline de Jenkins (CI/CD)

El flujo automatizado definido en el archivo `Jenkinsfile.Rodrigo-Videla` ejecuta de extremo a extremo las siguientes fases:

* **Declarative: Checkout SCM:** Sincronización automática con la rama `main` del repositorio de GitHub.
* **Install:** Preparación del entorno e instalación limpia de dependencias utilizando `pnpm install --frozen-lockfile`.
* **Test:** Ejecución automatizada de pruebas unitarias y de extremo a extremo (*E2E*) mediante Jest.
* **Build:** Construcción optimizada de la imagen Docker de producción.
* **Push:** Autenticación segura y carga de la imagen resultante en el registro público de Docker Hub bajo la etiqueta `rvidela/tarea-final:rodrigo-videla`.
* **Deploy:** Descarga e instanciación de un cliente nativo `kubectl` dentro del contenedor del agente para aplicar de forma segura las definiciones del archivo `entrega.yaml`.

---

## 5. Instrucciones de Ejecución Manual y Validación

### A. Construcción y Publicación Manual en Docker Hub
Para comprobar los artefactos de forma independiente antes de pasar por la automatización de Jenkins, se pueden ejecutar los siguientes comandos dentro del directorio `solucion-lab3`:
```bash
# Iniciar sesión en el registro
docker login

# Compilar la imagen apuntando al contexto de la aplicación en ../lab3
docker build -f Dockerfile -t rodrigovidela/tarea-final:rodrigo-videla ../lab3

# Subir la imagen al repositorio público
docker push rodrigovidela/tarea-final:rodrigo-videla
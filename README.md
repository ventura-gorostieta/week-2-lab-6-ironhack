# week-2-lab-6-ironhack
Analizando integración Docker en CI-CD


# Lab Instructions
## 1. Review Simulated CI/CD Pipeline Configuration:

### Build Stage:
* **Code Commit:** Developers commit code to a version control system, triggering the CI pipeline.
* **Docker Image Creation:** Dockerfiles define the environment and dependencies, and Docker builds an image which encapsulates the application and its runtime.

> [!TIP]
> **Beneficios:**
> * Consistencia del Entorno: Al usar contenedores (Docker), el entorno de desarrollo es el mismo que el de producción u otros ambientes, eliminando problemas de ambientación y aprovisionamiento.
> * Portabilidad: Las imágenes Docker son portátiles y pueden ejecutarse en cualquier plataforma que soporte Docker. Similar a como sucede en contendores de barcos.
>
> **Desafíos Potenciales:**
> * Vulnerabilidades de Seguridad en las Imágenes: Las imágenes pueden contener vulnerabilidades que podrían ser explotadas, por lo que antes de usar una imagen que exita en docker-hub, se debe analizar previamente para detectar posibles vulnerabilidades que comprometan la seguridad de aplicaciones y entornos. De igual manera al momento de hacer build de una imagen custom, se debe analizar las dependencias y la composición de la imagen y software, jars (analsis SCA) para poder detectar a tiempo issues de seguridad en imagenes docker.
> * Tamaño de las Imágenes: Imágenes grandes pueden ralentizar el proceso de CI/CD debido a los tiempos de descarga y subida, por lo que se debe tener cuidado de los archivos que se empaquetan a una imagen y de ser posible resolver dependencias en tiempo de construicción, por ejemplo de un servidor de depencias como maven o artifactory de Jfrog.



### Test Stage:
* **Automated Testing:** Docker images are used to spin up containers where automated tests are conducted, ensuring that the application behaves as expected in an environment identical to production.

> [!TIP]
> **Beneficios:**
> * **Aislamiento:** Cada conjunto de pruebas se ejecuta en un contenedor aislado, evitando conflictos entre pruebas, por que permite confinar y ejecutar de manera asincrona stages de un pipeline sin que estos dependan directamente uno de otro para ejecutarse.
> * **Repetibilidad:** Los tests se ejecutan en el mismo entorno cada vez, lo que reduce la variabilidad en los resultados, es decir cada evento de ejecución del pipeline siempre se ejecuta con el mismo entorno configurado.
>
> **Desafíos Potenciales:**
> * **Gestión de Dependencias:** Asegurar que todas las dependencias necesarias estén disponibles en las imágenes puede ser complejo, esto para los casos donde las pruebas sean muy especificas o requieran consumir herramientas de terceros o servicios expuestos, lo cual puede ser complicado de configurar en el stage.
> * **Performance:** Ejecutar muchos contenedores de prueba en paralelo puede consumir muchos recursos, impactando en el rendimiento del sistema de CI, lo cual se traduce en costos para la organización y desde luego degradando la experiencia de usuario a nivel CI-CD, por ejemplo runners de git actions se acaben y los jobs se quedan encolados esperando a que se liberen recursos para ejecutar un determinado pipeline.


### Deployment Stage:
* **Container Registry:** Successfully tested Docker images are tagged and pushed to a Docker registry.
* **Orchestration and Deployment:** Tools like Kubernetes or Docker Swarm manage the deployment of these images into containers across different environments, handling scaling and load balancing.

> [!TIP]
> **Beneficios:**
> * Escalabilidad: Docker facilita el escalado horizontal de la aplicación mediante la replicación de contenedores. Donde con la combinación de técnologias como AWS EKS, permite orquestar una gran cantidad de imagenes, contenedores y pods.
> * Gestión Eficiente de Recursos: Orquestadores como Kubernetes(Por ejemplo AWS EKS) optimizan la utilización de recursos y manejan automáticamente la distribución de cargas. 
> 
> **Desafíos Potenciales:**
>
> * **Complejidad en la Orquestación:** Gestionar un gran número de servicios y contenedores puede ser complejo si no se cuenta con herramientas de orquestación y monitoreo: AWS EKS, ECR o DataDog que permiten monitorear la salud de recursos como contenedores y pods.
> * **Seguridad:** Proteger los contenedores y la comunicación entre ellos es importante para evitar eventos de seguridad ya que los contenedores deben configurarse con los recursos y permisos necesarios para su funcionamiento, así mismo usando ALBs de kubernetes se puede limitar la comunicación y acceso a recursos entre pods y contenedores, adicional que frente a toda la infraestructura de contenedores se debe contar y adjunntar politicas a nivel security group y ACLs que filtran trafico entrante y saliente entre pods.

## 2. Analyze Enhancements and Potential Issues:
* **Enhancements:** Consider how Docker’s containerization benefits the build, test, and deployment processes by providing consistency, portability, and scalability.
* **Potential Issues:** Reflect on any possible challenges that might arise with Docker integration, such as security vulnerabilities in images, complexity in managing large numbers of services, or difficulties in orchestrating containers.
 
 ## 3. Write an Analysis Report:
### Task: Individually write a brief report that:
* Summarizes how Docker is integrated into each stage of the CI/CD pipeline.
* Analyzes the benefits and potential challenges identified during the review.
* Suggests theoretical solutions or best practices to overcome the challenges.

> [!TIP]
> **Soluciones y Mejores Prácticas:**
> * **Seguridad de Imágenes:**
> * Escaneo Regular: Implementar herramientas de escaneo de vulnerabilidades para imágenes Docker, como  Clair la cual es gratuita, de pago tener una opción popular llamada Snyk.
> * Imágenes Base Mínimas: Usar imágenes base mínimas y que estas se actualicen con regularidad, de esta manera se cumple con el principio de seguridag: "menor superficie de exposición".
>
> **Gestión de Dependencias:**
>
> * **Multi-stage Builds:** Utilizar construcciones multi-stage en Dockerfiles para reducir el tamaño de las imágenes y asegurar que solo las dependencias necesarias estén incluidas.
> * **Cacheo de Dependencias:** Subir a cache y obtener de la misma cache las dependencias durante la construcción de imágenes para acelerar el proceso de construicción de una imagen.
>
> * **Optimización del Rendimiento:**
> * **Autoscaling:** Configurar el autoscaling en el orquestador para manejar cargas variables sin consumir recursos innecesarios, esto lo podemos lograr configurando el HPA del contendor, regularmente el el configMap, indicando un número minimo de replicas y un maximo para escalar según la metrica de seguimiento, que tipicamente puede ser un % de uso de cpu.
> * **Monitoreo y Logging:** Implementar soluciones robustas de monitoreo y logging para identificar y resolver rápidamente problemas de rendimiento, esto podemos lograrlo indicando un paso de instalación de un agente  ya sea en la propia imagen docker o en el orquestador del pod, agregar un contenedor de logging y monitoreo, por ejemplo dynatrace o data dog.
>
> **Orquestación Eficiente:**
>
> * **Gestión de Configuraciones:** Usar herramientas de gestión de configuraciones como argoCd para manejar la complejidad del despliegue y administración.
> * **Políticas de Seguridad:** Implementar políticas de seguridad estrictas, como network policies en Kubernetes, para proteger la comunicación entre contenedores, tambien podemos adjuntar un security group al pod, un ACL donde se especifiquen las politicas de comunicación, puertos abiertos para consumo y quienes unicamente  pueden conectarse al pod, por ejemplo que todas las peticiones entrantes al POD B, solo se permitan por el puerto 80 y 443 u y que provengan del security group adjunto al balanceador de un FE.

## Ejemplo de docker file

```
FROM openjdk:8-jdk-alpine

RUN addgroup -S mygroup && adduser -S userApp -G mygroup

VOLUME /tmp

USER userApp

ARG JAR_FILE=target/*.jar
ARG SERVICE_NAME=myApp
ARG DEBUG=false

ADD ${JAR_FILE} app.jar

ADD security-agent.jar /security/

EXPOSE 8080

ENV SERVICE_NAME=${SERVICE_NAME}
ENV DEBUG=${DEBUG}

ENTRYPOINT ["java", "-javaagent:/security/security-agent.jar", "-jar", "/app.jar"]

```

## Ejemplo pipeline usando git actions, SAST,DAST y SCA

```
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    env:
      SERVICE_NAME: myApp
      DEBUG: false

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up JDK 8
      uses: actions/setup-java@v1
      with:
        java-version: 8

    - name: Build with Maven
      run: mvn clean package

    - name: Run Unit Tests
      run: mvn test

    - name: Static Code Analysis (SCA)
      run: mvn dependency-check:check

    - name: Static Application Security Testing (SAST)
      run: mvn sonar:sonar -Dsonar.projectKey=myApp -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=$SONAR_TOKEN

    - name: Dynamic Application Security Testing (DAST)
      run: |
        docker run --rm -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py -t http://localhost:8080

    - name: Build Docker Image
      run: |
        docker build . --file Dockerfile --tag ${{ secrets.DOCKER_USERNAME }}/myApp:${{ github.sha }}
        docker tag ${{ secrets.DOCKER_USERNAME }}/myApp:${{ github.sha }} ${{ secrets.DOCKER_USERNAME }}/myApp:latest

    - name: Log in to Docker Hub
      run: echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin

    - name: Push Docker Image
      run: |
        docker push ${{ secrets.DOCKER_USERNAME }}/myApp:${{ github.sha }}
        docker push ${{ secrets.DOCKER_USERNAME }}/myApp:latest

    - name: Deploy to stage-production
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.PRODUCTION_SERVER }}
        username: ${{ secrets.PRODUCTION_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          docker pull ${{ secrets.DOCKER_USERNAME }}/myApp:latest
          docker stop app || true
          docker rm app || true
          docker run -d --name app -p 8080:8080 ${{ secrets.DOCKER_USERNAME }}/myApp:latest

```

### Evaluation Criteria
The success of this lab will be assessed based on the following criteria:
* Quality of Analysis: Evaluated by the depth and accuracy with which Docker’s integration into each stage of the CI/CD pipeline is explained. This includes a clear identification of how Docker enhances processes and potential drawbacks it might introduce.
* Understanding of Docker Concepts: Assessed based on the ability to apply theoretical knowledge about Docker to identify its impacts on CI/CD pipeline efficiency and reliability.
* Solution Effectiveness: Measured by the appropriateness and potential effectiveness of the proposed solutions or improvements to address the identified * challenges within the pipeline setup.
* Report Clarity and Structure: Evaluated by the organization of the report, including how well the analysis is structured, the clarity of writing, and the logical flow of ideas from introduction to conclusion.

> [!IMPORTANT]
> # **Todas las respuestas estan encerradas en este  help tip que marca color verde:**

> [!TIP]
> **Así ...**

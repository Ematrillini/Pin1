# Pin1
Manual de Implementación de Jenkins en Docker
________________________________________
Chequeo de contenedores, imágenes y volúmenes existentes
Ejecute los siguientes comandos para asegurarse de que no haya contenedores, imágenes o volúmenes activos en Docker:
En la terminal ejecutar:


docker rm -f $(docker ps -aq)  
docker rmi -f $(docker images -aq)  
docker volume rm $(docker volume ls -q)
________________________________________
Creación del Dockerfile
Cree un archivo llamado Dockerfile con el siguiente contenido:
(Para verificar el contenido, utilice: cat Dockerfile)
Dockerfile:



FROM jenkins/jenkins:lts

USER root
RUN apt-get update && \
    apt-get install -y apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - && \
    echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y docker-ce && \
    usermod -aG docker jenkins

USER jenkins
________________________________________
Construcción de la imagen Docker
Ejecute el siguiente comando para construir la imagen:
En la terminal ejecutar:

docker build -t jenkins-docker .
________________________________________
Iniciar el contenedor
Inicie el contenedor de Jenkins con el siguiente comando:
En la terminal ejecutar:


docker run -d \
  --name jenkins \
  --privileged \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v jenkins_home:/var/jenkins_home \
  -p 8080:8080 \
  jenkins-docker
________________________________________
Acceso a Jenkins desde el navegador
Abra su navegador e ingrese a localhost:8080.
Obtener la contraseña de acceso inicial
Ejecute este comando para obtener la clave requerida:
En la terminal ejecutar:


docker exec -it jenkins cat d/var/jenkins_home/secrets/initialAdminPassword
________________________________________
Configuración de permisos en el socket de Docker
Para evitar problemas de permisos, ejecute el siguiente comando:
En la terminal ejecutar:


sudo chmod 666 /var/run/docker.sock
Nota: Este comando permite que cualquier usuario del sistema pueda leer y escribir en el socket de Docker. Úselo con precaución.
________________________________________
Configurar Docker Hub
•	Ingrese a Docker Hub y cree un repositorio:
•	En la esquina superior derecha, haga clic en "Create Repository".
•	Complete los siguientes campos:
o	Name (Nombre del repositorio): Ingrese un nombre relevante para la imagen (Ejemplo: mi-imagen-docker).
o	Description (Descripción): (Opcional) Agregue una breve descripción del propósito de la imagen.
o	Visibility:
	Public: La imagen estará disponible para cualquier usuario de Docker Hub.
	Private: Solo usted y sus colaboradores podrán acceder.


Guarde las credenciales de usuario y contraseña, ya que serán necesarias para integrarlo con Jenkins.
________________________________________
Acceso y configuración inicial en Jenkins
1.	Instale los plugins recomendados al inicio.
2.	Descargue los plugins de Docker y Docker Pipeline.
3.	Cree credenciales en Jenkins:
o	Usuario: Su usuario de Docker Hub
o	Contraseña: Su contraseña de Docker Hub
o	ID: Use un nombre simple que identificará las credenciales en los pipelines.
________________________________________
Configuración del script del pipeline
Modifique las siguientes variables según su entorno:
•	IMAGEN: "USUARIO_DOCKER_HUB/NOMBRE_REPO"
•	DOCKER_CREDENTIALS_ID: 'ID_CREDENCIALES_JENKINS'
Aplique estos valores en el siguiente script:
________________________________________
Script Jenkins
groovy




pipeline {
    environment {
        IMAGEN = "ematrillini/prueba1"  // Nombre del repositorio en DockerHub
        DOCKER_CREDENTIALS_ID = 'Newtriunf'  // Credenciales creadas en Jenkins
    }
    agent any
    stages {
        stage('Clone') {
            steps {
                script {
                    git branch: "main", url: 'https://github.com/josedom24/jenkins_docker.git'
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    newApp = docker.build "${env.IMAGEN}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image("${env.IMAGEN}:${env.BUILD_NUMBER}").inside('-u root') {
                        sh 'apache2ctl -v'  // **Cambia este comando según sea necesario**
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
                        newApp.push()
                    }
                }
            }
        }

        stage('Clean Up') {
            steps {
                script {
                    sh "docker rmi ${env.IMAGEN}:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
________________________________________
Aplicar y guardar la configuración del pipeline
•	Presione Aplicar (si aparece) y luego Guardar.
•	Con estos pasos completos, ya puede construir el pipeline desde Jenkins.

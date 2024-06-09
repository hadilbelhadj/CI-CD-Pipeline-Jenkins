# Jenkins Pipeline for HelloWorld-Springboot-App

This Jenkins pipeline automates the process of building, testing, packaging, and deploying the HelloWorld-Springboot-App, a Spring Boot application hosted on GitHub.

## Pipeline Overview

The pipeline is defined in a Jenkinsfile and consists of the following stages:

1. **Git clone**
2. **Compile**
3. **Test**
4. **Package**
5. **Deploy**

## Stages

### 1. Git clone

This stage clones the HelloWorld-Springboot-App repository from GitHub.

```groovy
stage('Git clone') {
    agent any
    steps {
        git 'https://github.com/shazforiot/HelloWorld-Springboot-App.git'
    }
}
```
2. Compile

This stage compiles the Java source code using Maven.

```groovy
stage('compile') {
    agent any
    steps {
        sh 'mvn compile'
    }
}
```
3. Test

This stage runs the unit tests using Maven.

```groovy
stage('test') {
    agent any
    steps {
        sh 'mvn test'
    }
}
```
4. Package

This stage packages the application into a WAR file using Maven.

```groovy
stage('package') {
    agent any
    steps {
        sh 'mvn package'
    }
}
```
5. Deploy

This stage deploys the WAR file to a Docker container running Tomcat. It involves the following steps:

- Create a Docker image directory
- Copy the WAR file to the directory
- Create a Dockerfile
- Build the Docker image
- Run a Docker container from the image

```groovy
stage('deploy') {
    agent any
    steps {
        sh '''
        rm -rf dockerimg
        mkdir dockerimg
        cd dockerimg
        cp /var/lib/jenkins/workspace/ContinuousDeploymentJob/target/helloworld-0.0.1.war .
        touch Dockerfile
        cat <<EOT >> Dockerfile
        FROM tomcat
        ADD helloworld.war /usr/local/tomcat/webapps/
        CMD ["catalina.sh", "run"]
        EXPOSE 8080
        EOT
        sudo docker build -t webimage:$BUILD_NUMBER .
        sudo docker container run -itd --name webserver$BUILD_NUMBER -p 8080:8080 webimage:$BUILD_NUMBER
        '''
    }
}
```

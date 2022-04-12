pipeline {
    agent any
    //environment { }
  	tools {
  	    maven 'mvn'
  	}

    stages {
        stage('Download Git') {
            steps {            
                git branch: "${env.branch}", credentialsId: 'redhat-gogs', url: "${GIT_URL}"
           }
        }
        stage('Asignación Variables'){
            steps{
                script{
                     rama = "${env.branch}"
                     archiva = ""
                     namespace = ""
                     //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
                    IMAGE = readMavenPom().getArtifactId()
	    			VERSION = readMavenPom().getVersion()
	    			echo "IMAGE = ${IMAGE}"
	    			echo "VERSION = ${VERSION}"
                    if(rama.contains("desarrollo")){
                        archiva = "bfa-desa"
                        namespace = "dev"
                    }
                    if(rama.contains("preproduccion")){
                        archiva = "bfa-pp"
                        namespace = "preprd"
                    }
                    if(rama.contains("master")){
                        archiva = "bfa-produccion"
                        namespace = "prod"
                    }
                    echo "archiva = ${archiva} namespace = ${namespace}"
                }
            }
        }
        stage('Build Maven') {
            steps {
                sh "mvn clean install -DskipTests -s configuration/settings.xml"
            }
        }
        stage('Copiar JAR'){
            steps{
                sh "cp target/${IMAGE}-${VERSION}.jar target/ROOT.jar"
            }
        }
        // stage('Test Maven') {
           // steps {
             // sh "mvn test -s configuration/settings.xml"
              //step([$class: 'JUnitResultArchiver', testResults: '*/target/surefire-reports/TEST-.xml'])
            //}
          //}
        /*stage('Code Analysis Sonar') {
            steps {
                script {
                    sh "mvn sonar:sonar -Dsonar.host.url=http://sonarqube:9000 -DskipTests=true -s configuration/settings.xml"
                }
            }
        }
        stage('Archive App') {
            steps {
                sh "mvn deploy -DskipTests=true -P ${archiva} -s configuration/settings.xml"
            }
        }*/
        stage('Build Openshift') {
            steps {
                sh "${OC_LOGIN}"
                sh "oc project ${namespace}"
                sh "oc start-build bc/${IMAGE} --from-file=target/ROOT.jar --follow --wait=true"
            }
        }
        /*stage('Deploy Rollout Openshift'){
            steps {
                sh "oc rollout latest dc/${IMAGE}"             
            }
        }*/
    }
}
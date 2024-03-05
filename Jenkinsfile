  pipeline {
      agent any
   //    tools {
    //     maven "MAVEN3"
    //     jdk "OracleJDK8"
    // }

      environment {
          registryCredential = 'ecr:us-east-2:awscreds'
          clientRegistry = "339712773002.dkr.ecr.us-east-1.amazonaws.com/clients"
          booksRegistry = "339712773002.dkr.ecr.us-east-1.amazonaws.com/books"
          mainRegistry = "339712773002.dkr.ecr.us-east-1.amazonaws.com/main"
          vprofileRegistry = "https://339712773002.dkr.ecr.us-east-1.amazonaws.com"
          // cluster = "vprofile"
          // service = "vprofileappsvc"
      }
    stages {
      stage('Fetch code'){
        steps {
          git branch: 'docker', url: 'https://github.com/subalekshmi2209/emartapp.git'
        }
      }


      stage('Test'){
        steps {
          sh 'mvn test'
        }
      }

      stage ('CODE ANALYSIS WITH CHECKSTYLE'){
              steps {
                  sh 'mvn checkstyle:checkstyle'
              }
              post {
                  success {
                      echo 'Generated Analysis Result'
                  }
              }
          }

          stage('build && SonarQube analysis') {
              environment {
               scannerHome = tool 'sonar4.7'
            }
              steps {
                  withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                     -Dsonar.projectName=vprofile-repo \
                     -Dsonar.projectVersion=1.0 \
                     -Dsonar.sources=src/ \
                     -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                     -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                  }
              }
          }

          stage("Quality Gate") {
              steps {
                  timeout(time: 1, unit: 'HOURS') {
                      // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                      // true = set pipeline to UNSTABLE, false = don't
                      waitForQualityGate abortPipeline: true
                  }
              }
          }

       stage('Build Angular Image') {
          when { changeset "client/*"}
         steps {
         
           script {
      dockerImage = docker.build( clientRegistry + ":$BUILD_NUMBER", "./client/")
                  
               }

       }
      
      }

      stage('Deploy Angular Image') {
      when { changeset "client/*"}

            steps{
              script {
                docker.withRegistry( vprofileRegistry, registryCredential ) {
                  dockerImage.push("$BUILD_NUMBER")
                  dockerImage.push('latest')
                }
              }
            }
          }
   

       stage('Build books Image) {
          when { changeset "javaapi/*"}
         steps {
         
           script {
      dockerImage = docker.build( booksRegistry + ":$BUILD_NUMBER", "./javaapi/")
                  
               }

       }
      
      }

      stage('Deploy Book Image') {
      when { changeset "javaapi/*"}

            steps{
              script {
                docker.withRegistry( vprofileRegistry, registryCredential ) {
                  dockerImage.push("$BUILD_NUMBER")
                  dockerImage.push('latest')
                }
              }
            }
          }

       stage('Build Main Image') {
          when { changeset "nodeapi/*"}
         steps {
         
           script {
      dockerImage = docker.build( mainRegistry + ":$BUILD_NUMBER", "./nodeapi/")
                  
               }

       }
      
      }

      stage('Deploy Main Image') {
      when { changeset "nodeapi/*"}

            steps{
              script {
                docker.withRegistry( vprofileRegistry, registryCredential ) {
                  dockerImage.push("$BUILD_NUMBER")
                  dockerImage.push('latest')
                }
              }
            }

       }
       
     

    }
  }

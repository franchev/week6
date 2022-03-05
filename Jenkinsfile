pipeline {
     agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: gradle
            image: gradle:6.3-jdk14
            command:
            - sleep
            args:
            - 99d
            volumeMounts:
            - name: shared-storage
              mountPath: /mnt        
          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug
            command:
            - sleep
            args:
            - 9999999
            volumeMounts:
            - name: shared-storage
              mountPath: /mnt
            - name: kaniko-secret
              mountPath: /kaniko/.docker
          restartPolicy: Never
          volumes:
          - name: shared-storage
            persistentVolumeClaim:
              claimName: jenkins-pv-claim
          - name: kaniko-secret
            secret:
                secretName: dockercred
                items:
                - key: .dockerconfigjson
                  path: config.json
        '''
    }
	}
     triggers {
          pollSCM('* * * * *')
     }
     stages {
          stage("Compile") {
               steps {
			      container('gradle') {
				      sh """
					  echo main branch
			          chmod +x gradlew
                      ./gradlew compileJava
					  """
                  }   
               }
          }
          stage("Unit test") {
		       when {
                  expression { env.BRANCH_NAME == 'main' && env.BRANCH_NAME == 'feature'}
               }
               steps {
			     container('gradle') {
                    sh "./gradlew test"
				  }
               }
          }
          stage("Code coverage") {
		       when {
                   expression { env.BRANCH_NAME == 'main' }
               }
               steps {
			     container('gradle') {
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"
					}
               }
          }
          stage("Static code analysis") {
		       when {
                    expression { env.BRANCH_NAME == 'main' && env.BRANCH_NAME == 'feature'}
               }
               steps {
			     container('gradle') {
                    sh "./gradlew checkstyleMain"
				 }
               }
          }
          stage("Package") {
               steps {
			       script {
                     try {
					   container('gradle') {
                        sh """
						  ./gradlew build
						  mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
						"""
					   }
                     } 
					 catch (Exception e) {
                       echo 'Exception occurred: ' + e.toString()
                       sh 'exit 1'
                      }
                    }
				}
          }

          stage("Docker build") {
               steps {
			        container('kaniko') {
					  sh '''
                        echo 'FROM openjdk:8-jre' > Dockerfile
                        echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                        echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                        ls /mnt/*jar
                        mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
					  '''
					  script {
					    if (env.BRANCH_NAME == 'main'){
						  sh "/kaniko/executor --context `pwd` --destination franchev/calculator:1.0"
						}
						if (env.BRANCH_NAME == 'feature'){
						  sh "/kaniko/executor --context `pwd` --destination franchev/calculator-feature:0.1"
						}
					  }
					}
               }
          }

          /*stage("Update version") {
               steps {
                    sh "sed  -i 's/{{VERSION}}/${BUILD_TIMESTAMP}/g' calculator.yaml"
               }
          }
		  
          /*stage("Deploy to staging") {
               steps {
                    sh "kubectl config use-context staging"
                    sh "kubectl apply -f hazelcast.yaml"
                    sh "kubectl apply -f calculator.yaml"
               }
          }

          stage("Acceptance test") {
               steps {
                    sleep 60
                    sh "chmod +x acceptance-test.sh && ./acceptance-test.sh"
               }
          }

          stage("Release") {
               steps {
                    sh "kubectl config use-context production"
                    sh "kubectl apply -f hazelcast.yaml"
                    sh "kubectl apply -f calculator.yaml"
               }
          }
          stage("Smoke test") {
              steps {
                  sleep 60
                  sh "chmod +x smoke-test.sh && ./smoke-test.sh"
              }
          } */
     }
}
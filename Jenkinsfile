podTemplate(yaml: '''
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
''') {
  node(POD_LABEL) {
    stage('Build a gradle project') {
      container('gradle') {
	    stage("Compile") {
               steps {
			        sh "chmod +x gradlew"
                    sh "./gradlew compileJava"
               }
          }
          stage("Unit test") {
		       when {
                  expression { env.BRANCH_NAME == 'main' && env.BRANCH_NAME == 'feature'}
               }
               steps {
                    sh "./gradlew test"
               }
          }
          stage("Code coverage") {
		       when {
                   expression { env.BRANCH_NAME == 'main' }
               }
               steps {
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"
               }
          }
          stage("Static code analysis") {
		       when {
                    expression { env.BRANCH_NAME == 'main' && env.BRANCH_NAME == 'feature'}
               }
               steps {
                    sh "./gradlew checkstyleMain"
               }
          }
        stage('Build') {
		  script {
		    try {   
                sh '''
                sed -i '4 a /** Main app */' src/main/java/com/leszko/calculator/Calculator.java
                ./gradlew build
                mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
                '''
			} catch(err) {
			    echo err.getMessage()
				echo "Error while building package, will not continue to build container image. Exiting"
				sh "exit 1"
			}
		  }
        }
      }
    }

    stage('Build Java Image') {
      container('kaniko') {
        stage('Build a container') {
		 when {
                 expression { env.BRANCH_NAME != 'playground' }
            }
          sh '''
          echo 'FROM openjdk:8-jre' > Dockerfile
          echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
          echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
          ls /mnt/*jar
		  if (env.BRANCH_NAME == "main") {
            mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
            /kaniko/executor --context `pwd` --destination franchev/calculator:1.0
		  }
		  if (env.BRANCH_NAME == "feature") {
		    mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
            /kaniko/executor --context `pwd` --destination franchev/calculator:0.1
		  }
          '''
        }
      }
    }

  }
}

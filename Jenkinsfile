pipeline {
     agent any
     triggers {
          pollSCM('* * * * *')
     }
     stages {
          stage("Compile") {
               steps {
                    sh "echo feature one"
                    sh "chmod +x gradlew"
                    sh "./gradlew compileJava"
               }
          }
          stage("Unit test") {
               steps {
                    sh "./gradlew test"
               }
          }
          stage("Code coverage") {
               steps {
                   script {
                     if (env.BRANCH_NAME == "main") {
                       sh "echo My CC branch is: ${env.BRANCH_NAME}"
                       sh "./gradlew jacocoTestReport"
                       sh "./gradlew jacocoTestCoverageVerification"
                     }
                   }
               }
          }
          stage("Static code analysis") {
               steps {
                    sh "./gradlew checkstyleMain"
               }
          }
          stage("Package") {
               steps {
                    sh "./gradlew build"
               }
          }

     }
}

pipeline{
  agent any
  stages{
    stage('build appication'){
      steps{
        
      script{
       bat'mvn clean package -Dmaven.test.skip=true'

        echo"checkout sucessfull"
        echo" congrats"

      }
    }
    } 
    // abcd
    
    stage('unit test'){
      steps{
        
      script{
     // bat'mvn test'
      // junit '**/surefire-reports/*.xml'
        echo"build sucessfull"

       /* publishHTML (target : [allowMissing: false,
         alwaysLinkToLastBuild: true,
         keepAll: true,
         reportDir: 'target/site/jacoco',
         reportFiles: 'index.html',
         reportName: 'coverage report',
         reportTitles: 'coverage report']) */
      }
    }
    }  
    
    
    
        stage('deploy'){
      steps{
        
      script{
        echo"deploy  sucessfull"
      }
    }
    }

    stage('monitor'){
          steps{

          script{
            echo"monitor  sucessfull"
          }
        }
        }
  }
}

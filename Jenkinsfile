pipeline{
  agent any
  stages{
    stage('build appication'){
      steps{
        
      script{
       bat'mvn clean package -Dmaven.test.skip=true'

        echo"checkout sucessfull"
      }
    }
    } 
    // abcd
    
    stage('unit test'){
      steps{
        
      script{
      bat'mvn test'
       junit '**/surefire-reports/*.xml'
        echo"build sucessfull"
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

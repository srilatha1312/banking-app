pipeline{
  agent any
  stages{
    stage('build appication'){
      steps{
        
      script{
       bat'mvn clean package'
        echo"checkout sucessfull"
      }
    }
    } 
    // abcd
    
    stage('build'){
      steps{
        
      script{
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

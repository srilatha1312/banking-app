pipeline{
  agent any
  parameters {
                booleanParam(name: 'PERFORM_RELEASE', defaultValue: false, description: 'Used by CI team to create release candidate. Works only on release branches and master. Please confirm with CI before selecting PERFORM_RELEASE')
            }
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
      bat'mvn test'
       junit '**/surefire-reports/*.xml'
        echo"build sucessfull"

        publishHTML (target : [allowMissing: false,
         alwaysLinkToLastBuild: true,
         keepAll: true,
         reportDir: 'target/site/jacoco',
         reportFiles: 'index.html',
         reportName: 'coverage report',
         reportTitles: 'coverage report'])
      }
    }
    }  
    
    
    
        stage('code quality check'){
      steps{
        
      script{
         bat 'mvn sonar:sonar \
          -Dsonar.host.url="http://localhost:9000" \
          -Dsonar.login=7ce69212e6dd7c108eef4520418ea47664d6c062'
          sonarQualityBreaker(false)
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


// function defninations

def qualityGateCheck(String projectKey,boolean returnStatus = false){

    def qualityGateUrl = "https://sonar.aib.pri/api/qualitygates/project_status?projectKey=${projectKey}"
    sh "curl -u 3c2968b5656907c53943c4f2725088d1efb35a49: $qualityGateUrl -o qualityGate.json"
    def qualitygate = jsonParse(readFile('qualityGate.json'))
    def qualitygateJson = JsonOutput.toJson(qualitygate)
    echo JsonOutput.prettyPrint(qualitygateJson)
    if ("ERROR".equals(qualitygate["projectStatus"]["status"])) {
        if (!returnStatus){
            error  "SonarQube Quality Gate failure"
        }
    }
    if(returnStatus){
        return qualitygate["projectStatus"]["status"]
    }
    echo  "SonarQube Quality Gate success"
}

def jsonParse(text) {
    return new groovy.json.JsonSlurperClassic().parseText(text);
}

def readStringValueFromFile(String fileName,String key){
    value = ''
    if(fileExists(fileName)) {
        String fileContent = readFile(fileName)
        for (String line : (fileContent.split('\n') as List)) {
            if (line.contains(key)) {
                value = line.split('=')[1].trim()
                return value
            }
        }
    }
    return value
}

@NonCPS
def sonarQualityBreaker(boolean ignoreSonarQube) {
    def groupId = ""
    def artefactId = ""
    if (fileExists('pom.xml')) {
        def pomXml = new XmlParser().parseText(readFile('pom.xml'))
        groupId = pomXml.get('groupId').text()
        artefactId = pomXml.get('artifactId').text()
    } else if (fileExists('build.gradle')) {
        groupId = readStringValueFromFile("gradle.properties", "group=").trim()
        artefactId = readStringValueFromFile("settings.gradle", "rootProject.name").trim().replace('\'','')
    }

    if (groupId != '' && artefactId != "") {
        sonarProjectKey = "${groupId}:${artefactId}"
        checkSonarAnalysisTaskStatus()
        status = qualityGateCheck(sonarProjectKey, true)
        if ("ERROR".equals(status)) {

            if (!ignoreSonarQube) {
                    displaySonarUrls(sonarProjectKey)
                    error "SonarQube Quality Gate failed"
            } else {
                    echo "SonarQube Quality Gate failed. But ignoring because IGNORE_SONAR_RESULTS enabled"
                   }
            } else {
                   echo "SonarQube Quality Gate success"
            }
                displaySonarUrls(sonarProjectKey)
        }
}

def displaySonarUrls(String projectKey){
    echo "SonarQube Dashboard Url: https://sonar.aib.pri/dashboard?id=${projectKey}"
    echo "SonarQube QualityGate API Url: https://sonar.aib.pri/api/qualitygates/project_status?projectKey=${projectKey}"
}

def getSonarTaskUrl(String filePath){
    def sonarTaskUrl = ''
    def sonarResults = readFile encoding: 'UTF-8', file: "${filePath}"
    def listResults = sonarResults.tokenize()
    for (String url in listResults) {
        if (url.contains('ceTaskUrl'))
            sonarTaskUrl = url.replace('ceTaskUrl=','').trim()
    }
    return sonarTaskUrl
}

def checkSonarAnalysisTaskStatus(){
    sonarAnalysisStatus = false
    def sonarTaskUrl = ''
    if (fileExists('build/sonar/report-task.txt')) {
        sonarTaskUrl = getSonarTaskUrl('build/sonar/report-task.txt')
    }else if (fileExists('target/sonar/report-task.txt')) {
        sonarTaskUrl = getSonarTaskUrl('target/sonar/report-task.txt')
    }
    def startTime = System.currentTimeMillis()
    sonarTimeOutInMilliSeconds = Long.valueOf(env.SONAR_TIMEOUT_IN_MILLISECONDS?:500000)
    if(sonarTaskUrl != ''){
        while (!sonarAnalysisStatus) {
            sh "curl -u 7ce69212e6dd7c108eef4520418ea47664d6c062: $sonarTaskUrl -o sonarTaskStatus.json"
            def sonarTaskStatus = jsonParse(readFile('sonarTaskStatus.json'))
            def sonarStatus = sonarTaskStatus["task"]["status"]
            if ("IN_PROGRESS".equals(sonarStatus)) {
                sh "sleep 20"
                sonarAnalysisStatus = false
            } else if ("SUCCESS".equals(sonarStatus)) {
                if (sonarTaskStatus["task"]["startedAt"].startsWith(checkCurrentTime())) {
                    sonarAnalysisStatus = true
                }
                else {
                    error "SonarQube Quality Gate Failed On Timestamp Check!!!"
                }
            } else if ("ERROR".equals(sonarStatus) || "FAILED".equals(sonarStatus) || "CANCELLED".equals(sonarStatus)) {
                 error "SonarQube task status failed. Please check error details inside url: ${sonarTaskUrl}"
            }
            if (System.currentTimeMillis() - startTime > sonarTimeOutInMilliSeconds) {
                   error "SonarQube task status timed out. Please check error details inside url: ${sonarTaskUrl}"
            }
        }
    }
}

def checkCurrentTime() {
    def timeNow = new Date()
    String currentTime = timeNow.format("yyyy-MM")
    return currentTime
}

//develop
pipeline {
    agent any
    tools {
        maven "maven-default"
        jdk "java-default"
    }
    environment{
        DEVELOPERS_EMAIL="musema.hassen@gmail.com"
       
        REPO_URL="https://github.com/musema/jenkins-pipeline-examples-2.git"

        MAVEN_TOOL="maven-default"
        BUILD_FROM_BRANCH="develop"
    }
    options{
        timeout(time:48,unit:'HOURS')
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timestamps()
    }
    stages {
        stage('Prepare'){
            parallel{
                stage('Code-Checkout'){
                    steps{
                       echo "Source branch${env.BRANCH_NAME}"
                       checkoutCode()
                    }
                }
                stage('Prepare-Workspace'){
                    steps{
                        prepareWorkspace()
                    }
                }

            }
        }
          stage('What-Is-Happening'){
                    steps{
                       echo "Change detected?:${whatIsHappening()}"
                    }
                }
        stage("Package") {
            steps {
                packageArtifact()
            }
        }
        stage("Test"){
            steps{
                runTest()
            }
        }
        stage("Upload to Artifactory"){
            steps{
                uploadToArtifactory();
            }
        }
        stage("Decide-Deployment"){
            agent none
            steps{
                script{
                    env.DEPLOY_TO_DEV= input message: 'Approval is required',
							parameters: [
                                choice(name: 'Do you want to deploy to DEV?', choices: 'no\nyes', 
                                description: 'Choose "yes" if you want to deploy the DEV server')
                            ]
                }
            }
        }
        stage('Deploy To Dev'){
            when{
                environment name:'DEPLOY_TO_DEV', value:'yes'
            }
            steps{
                deployToServer("DEV")
            }
        }
        stage('Archive'){
            steps{
                archiveTheBuild()
            }
        }
    }
    post {
        always {
            finalizeWorkflow()

        }
            success {
                successMessage()
                 }
        failure {
                failureMessage()
            }
    }
}

//define groovy functions here
def prepareWorkspace(){
    echo 'Check here if everything is ready to make a build'
    if(isUnix()){
        echo 'Build is running on Unix family node '
            sh 'mvn --version'
            sh 'java -version'
            sh 'git --version'
    }
    else{
        echo 'Build is running on Windows family node '
            bat 'mvn --version'
            bat 'java -version'
            bat 'git --version'
    }
   
}
def checkoutCode(){
    count=1
    retry(3){
        echo "trying to checkout the code from SCM, Trial:${count}"
        //git branch: "${BUILD_FROM_BRANCH}",credentialsId: "${GITHUB_CREDENTIALS_ID}",url: "${REPO_URL}"
        git branch: "${BUILD_FROM_BRANCH}",url: "${REPO_URL}"
    }
}
def packageArtifact(){
    if(isUnix()){
          sh 'mvn clean package -Dmaven.test.failure.ignore=true'
    }
    else{
       bat 'mvn clean package -Dmaven.test.failure.ignore=true'
    }
}
def runTest(){
    if(isUnix()){
          sh 'mvn test'
    }
    else{
         bat 'mvn test'
    }
  
}

def archiveTheBuild(){
    archiveArtifacts '/target/**/*'
    junit '/target/surefire-reports/*.xml'

}
def deployToServer(deployTo){
    echo "We are deploying to : ${deployTo}"
}
def successMessage(){
    echo "Sending SUCCESS email to ${env.DEVELOPERS_EMAIL}"
    notifyBuild('SUCCESS')
}
def failureMessage(){
    echo "Notifying build failure"
    notifyBuild('FAILED')
}
def finalizeWorkflow(){
    echo "Cleaning up the workspace"
}

def notifyBuild(bStatus) {
    def subject = "${bStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def details = """STARTED: Job ${env.JOB_NAME} [${env.BUILD_NUMBER}]  \n Check console output at: ${env.BUILD_URL}"""
    echo "email sent"
    //mail to:"${env.DEVELOPERS_EMAIL}",subject:"${subject}",body:"${details}"
}

def uploadToArtifactory(){
echo "Publishing to Artifactory"

    def server = Artifactory.server 'artifactory-default'
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo = Artifactory.newBuildInfo()

    //Let's configure maven
    rtMaven.tool = MAVEN_TOOL
    rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
    rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server

    buildInfo.env.capture = true

    rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
    buildInfo.retention maxBuilds: 10, maxDays: 15, deleteBuildArtifacts: true
   
    server.publishBuildInfo buildInfo 

echo "Artifact is uploaded to Artifactory"
}

def whatIsHappening(){
    if(env.CHANGE_ID==null){
         echo "Nothing is happening, don't sweat it :D"
         return false
     }
     else{
        echo "Changes are detected, "
        echo "Changes made by${env.CHANGE_AUTHOR}"
        echo "Changes made by${env.CHANGE_TITLE}"
     }
}



pipeline {
    agent any
    tools {
        maven "maven-default"
        jdk "java-default"
    }
    environment{
        DEVELOPERS_EMAIL="musema.hassen@gmail.com"
        BUILD_FROM_BRANCH="master"
        REPO_URL="https://github.com/musema/jenkins-pipeline-examples-2.git"
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
        stage("Upload to Repository"){
            steps{
                uploadToRepository();
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
    bat 'mvn --version'
    bat 'java -version'
    bat 'git --version'
}
def checkoutCode(){
    count=1
    retry(3){
        echo "trying to checkout the code from SCM, Trial:${count}"
        //git branch: "${BUILD_FROM_BRANCH}",credentialsId: "${GITHUB_CREDENTIALS_ID}",url: "${REPO_URL}"
        git url:"${REPO_URL}"
    }
}
def packageArtifact(){
    bat 'mvn clean install -Dmaven.test.failure.ignore=true'
}
def runTest(){
    bat 'mvn test'
}
def uploadToRepository(){
    echo "We are about to publish artifacts to remote repository"
    bat 'mvn deploy'
    echo "Check if deployed to artifactory"
}
def archiveTheBuild(){
    archive '*/target/**/*'
    //junit '*/target/surefire-reports/*.xml'

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
   // mail to:"${env.DEVELOPERS_EMAIL}",subject:"${subject}",body:"${details}"
}


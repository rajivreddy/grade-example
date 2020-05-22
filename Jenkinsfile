def customImage = ""

def notifyviaMail(String buildStatus = 'SUCCESS') {
    def decodedJobName = env.JOB_NAME.replaceAll("%2F", "/")
    emailext attachLog: true, body: "${decodedJobName} - Build # ${env.BUILD_NUMBER} - ${buildStatus}: \n SSH URL:${ssh_url} \nCheck console output at ${env.BUILD_URL} to view the results in Attachments \n \n Thank you.", subject: "${decodedJobName}- Report", to: 'admin@example.com'
}

pipeline {
    agent any
    environment {
    registry = "ID.dkr.ecr.REGION.amazonaws.com"
    image = "example"
    registryCredential = "ecr_creds"
     }
    options {
    buildDiscarder(logRotator(numToKeepStr: '20'))
    disableConcurrentBuilds()
    }

    parameters {
        string(name: 'projectName', defaultValue: 'developersportal', description: 'Name of the project')
        string(name: 'sourceCodeRepo', defaultValue: 'https://github.com/rajivreddy/gradel-example.git', description: 'Source Code Repository')
        string(name: 'buildBranch', defaultValue: 'master', description: 'Which branch should be built , this could be the parent or the feature branch')
        string(name: 'gitRepoCredentials', defaultValue: 'git-creds', description: 'Build Script Branch')
    }
    triggers {
        GenericTrigger(
            genericVariables: [
            [key: 'ref', value: '$.ref'],
            [key: 'commit_msg', value: '$.head_commit[\'message\']']
            ],
            causeString: 'Triggered on $ref',
     
            token: 'developersportal-dev-build',
     
            printContributedVariables: false,
            printPostContent: false,
    
            regexpFilterText: '$ref',
            regexpFilterExpression: 'refs/heads/master'
        )
    }
    stages{
        stage("Pull from repo"){
            steps{
                script{
                    try {
                        cleanWs()
                        echo "Prepared local repo"
                        git branch: "${buildBranch}", credentialsId: "${gitRepoCredentials}", url: "${sourceCodeRepo}"
                        gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                        shortCommitHash = gitCommitHash.take(7)
                        // calculate a sample version tag
                        VERSION = shortCommitHash
                        // set the build display name
                        currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                        IMAGE_TAG = "$VERSION"
                    }
                    catch (Exception e) {
                        echo "Build failed!"
                        currentBuild.result = "FAILURE"
                        sh "exit 1"
                    }
                }
            }
        }
         stage('Unit Testcases') {
            steps {
                sh 'Running Unti test cases'

            }
        }
         stage('Code Quality Analysis') {
            steps {
                sh 'Invoking Sonar Scanner'
  
            }
        }
        stage('Build') {
            steps {
                try {
                sh 'docker run --rm -v $PWD:/app -w /app gradle gradle build'
                }
                catch (Exception e) {
                        echo "Build failed!"
                        currentBuild.result = "FAILURE"
                        sh "exit 1"
                    }
            }
        }

        stage('Build Docker Imahe') {

            steps {
                script {
                    try {
                        customImage = docker.build image + ":$IMAGE_TAG"
                    }
                        catch (Exception e) {
                        echo "Build failed!"
                        currentBuild.result = "FAILURE"
                        sh "exit 1"
                    }
                 }

            }
        }
        stage(""){
            steps{
                script{
                    try{
                    docker.withRegistry("${registry}", "ecr:us-west-2:${registryCredential}") {
                    //push image
                    customImage.push()
        }
                }
                }
                catch (Exception e) {
                        echo "Build failed!"
                        currentBuild.result = "FAILURE"
                        sh "exit 1"
                    }
            }
        }

    }
    post {
        always {
            notifyviaMail(currentBuild.result)
        }
    }
        
}

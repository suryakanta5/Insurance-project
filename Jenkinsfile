node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName

    stage('Prepare Environment') {
        echo 'Initialize all the variables'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName = "3.0"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checkout the code from Git repository'
            git 'https://github.com/suryakanta5/Insurance-project.git'
        } catch (Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
The Jenkins job ${JOB_NAME} has failed. Request you to please have a look at it immediately by clicking on the link below:
${BUILD_URL}''', 
            subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} failed', 
            to: 'suryakanta.it2019m@gmail.com'
        }
    }

    stage('Build the Application') {
        echo "Cleaning... Compiling... Testing... Packaging..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: false,
            reportDir: 'target/surefire-reports',
            reportFiles: 'index.html',
            reportName: 'HTML Report',
            reportTitles: ''
        ])
    }

    stage('Containerize the Application') {
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t surya556/insurance:${tagName} ."
    }

    stage('Push to DockerHub') {
        echo 'Pushing the Docker image to DockerHub'
        withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
            sh "${dockerCMD} login -u surya556 -p ${dockerhub}"
            sh "${dockerCMD} push surya556/insurance:${tagName}"
        }
    }

    stage('Configure and Deploy to Test Server') {
        ansiblePlaybook become: true, credentialsId: 'ansible', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml', vaultTmpPath: ''
    }
}

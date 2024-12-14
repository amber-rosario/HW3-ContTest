 pipeline 
{
    agent none
    stages 
    {
        stage('CLONE GIT REPOSITORY') 
        {
            agent 
            {
                label 'ubuntu-AppServer-2140-newest'
            }
            steps 
            {
                checkout scm
            }
        }  
 
        stage('SCA-SAST-SNYK') 
        {
            agent any
            steps 
            {
                script 
                {
                    catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS')
                    {
                        snykSecurity(snykInstallation: 'Snyk', snykTokenId: 'my-org-snyk-api-token', severity: 'high')
                    }
                    
                }
            }
        }
 
        stage('SonarQube Analysis') 
        {
            agent 
            {
                label 'ubuntu-AppServer-2140-newest'
            }
            steps 
            {
                script 
                {
                    def scannerHome = tool 'sonarqube'
                    withSonarQubeEnv('sonarqube') 
                    {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=chatapp \
                            -Dsonar.sources=."
                    }
                }
            }
        }
 
        stage('BUILD-AND-TAG') 
        {
            agent 
            {
                label 'ubuntu-AppServer-2140-newest'
            }
            steps 
            {
                script 
                {
                    def app = docker.build("ambrsrio0812/nodejschatapp")
                    app.tag("latest")
                }
            }
        }
 
        stage('POST-TO-DOCKERHUB') 
        {    
            agent 
            {
                label 'ubuntu-AppServer-2140-newest'
            }
            steps {
                script 
                {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_credentials') 
                    {
                        def app = docker.image("ambrsrio0812/nodejschatapp")
                        app.push("latest")
                    }
                }
            }
        }
 
        stage('DEPLOYMENT') 
        {    
            agent 
            {
                label 'ubuntu-AppServer-2140-newest'
            }
            steps 
            {
                sh "docker-compose down"
                sh "docker-compose up -d"   
            }
        }
    }
}
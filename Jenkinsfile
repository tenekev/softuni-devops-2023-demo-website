pipeline 
{
    agent 
    {
        label 'docker'
    }
    environment 
    {
        DOCKERHUB_CREDENTIALS=credentials('dockerhub')
    }
    stages 
    {
        stage('Clone Repo') 
        {
            steps 
            {
                git branch: 'main', url: 'http://192.168.99.102:3000/gitea_user/diabetes'
            }
        }
        stage('Build and Run')
        {
            steps
            {
                sh 'docker compose -f docker-compose-build.yaml up -d --force-recreate'

            }
        }
        stage('Test WEB Availibility')
        {
            steps
            {
                script 
                {
                    echo 'Wait 20 seconds';
                    sh 'sleep 20';
                    
                    echo 'Testing reachability'
                    sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8081) | grep 200'
                }
            }
        }
        stage('Test DB Availibility')
        {
            steps
            {
                script 
                {
                    echo 'Wait 10 seconds'
                    sh 'sleep 10'

                    echo 'Testing content'
                    sh "curl --silent http://localhost:8081 | grep захар"
                }
            }
        }
        stage('Teardown')
        {
            steps
            {
                sh 'docker compose -f docker-compose-build.yaml down'
            }
        }
        stage('Login Docker Hub')
        {
            steps 
            {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Build and Push') 
        {
            steps 
            {
                sh 'docker image build -t $DOCKERHUB_CREDENTIALS_USR/php-nginx-full -f Dockerfile.web.full .'
                sh 'docker image push $DOCKERHUB_CREDENTIALS_USR/php-nginx-full' 
                sh 'docker image build -t $DOCKERHUB_CREDENTIALS_USR/mariadb-full -f Dockerfile.db.full .'
                sh 'docker image push $DOCKERHUB_CREDENTIALS_USR/mariadb-full'
            }
        }
        stage('Deploy')
        {
            steps
            {
                sh 'docker compose -f docker-compose-deploy.yaml up -d --force-recreate'
            }
        }
    }
    post 
    { 
        always 
        { 
            cleanWs()
        }
    }
}

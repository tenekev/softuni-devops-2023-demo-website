pipeline 
{
    agent 
    {
        label 'docker'
    }
    environment 
    {
        DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    stages 
    {
        stage('Clone Repo') 
        {
            steps 
            {
                git branch: 'main', url: 'http://192.168.99.102:3000/vagrant/diabetes'
            }
        }
        stage('Build and Run')
        {
            steps
            {
                sh 'docker compose -f docker-compose-build.yaml up -d --force-recreate'

            }
        }
        stage('Test availibility')
        {
            steps
            {
                script 
                {
                    echo 'Wait 30 seconds';
                    sh 'sleep 30';
                    
                    echo 'Testing reachability'
                    sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080) | grep 200'
                    
                    echo 'Wait 10 seconds'
                    sh 'sleep 10'

                    echo 'Testing content'
                    sh "curl --silent http://localhost:8080 | grep захар"
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
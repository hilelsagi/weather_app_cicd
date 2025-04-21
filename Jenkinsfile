pipeline 
{
    agent {label "my_baby_slave"}
    options 
    {
        skipDefaultCheckout(true)
    }
    environment 
    {
        DOCKER_HUB_CREDENTIALS = credentials('hilelsagi-dockerhub')
        DOCKER_IMAGE_NAME = 'hilelsagi/sagi_weather_app_img'
        DOCKER_IMAGE_TAG = "latest"
    }

    stages 
    {
        stage('source') 
        {
            steps 
            {
                cleanWs()
                checkout scm
            }
        }
        stage ('install requirements')
        {
            steps
            {
                sh "python3 -m venv .venv"
                sh ". .venv/bin/activate"
                sh ".venv/bin/pip install --upgrade pip"
                sh ".venv/bin/pip install --no-cache-dir --break-system-packages -r requirements.txt"
            }
        }
        stage ('pylint')
        {
            steps 
            {
                sh ". .venv/bin/activate"
                sh ".venv/bin/pylint --fail-under=5 * > pylint.log"
            }
        }
        stage ('build')
        {
            steps
            {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE_NAME} ."
                echo "Finished building"
            }
        }
        stage ('run and test reachabillity')
        {
            steps
            {
                sh ". .venv/bin/activate"
                sh "docker-compose up -d"
                echo "Waiting for the app to start..."
                sh "sleep 5"
                echo "Running reachability test..."
                sh ".venv/bin/pytest tests/reachability_test.py"

            }
        }
        stage('Push to Docker Hub')
        {
            steps 
            {
                echo "Logging in to Docker Hub..."
                sh 'docker login -u ${DOCKER_HUB_CREDENTIALS_USR} -p ${DOCKER_HUB_CREDENTIALS_PSW}'
                echo "Pushing Docker image..."
                sh 'docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}'
            }
        }
        
        stage ('deploy') {
            steps {
                echo "deploying"
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh_key', keyFileVariable: 'EC2_KEY')]) {
                        sh "scp -o StrictHostKeyChecking=no -i $EC2_KEY docker-compose.yaml ec2-user@16.16.222.25:"
                        sh "ssh -o StrictHostKeyChecking=no -i $EC2_KEY ec2-user@16.16.222.25"
                        sh"docker pull hilelsagi/sagi_weather_app_img:latest"
                        sh"docker-compose up -d"
                    }
                echo "SCP completed successfully"
            }
        }
    }
    post
    {
        success
        {
            slackSend channel: '#succeeded-build', message: "Build SUCCESSFUL: ✅"
        }
        failure
        {
             slackSend channel: '#devops-alerts', message: "Build FAILED: ❌"
        }
    }
}

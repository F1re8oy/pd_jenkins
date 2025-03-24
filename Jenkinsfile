pipeline {
    agent any
    environment {
        PM2_HOME = "${WORKSPACE}\\.pm2" // Darba direktorija norādīšana lai pm2 var veidot log failus, bija kļūda, ka pm2 nevarēja veidot log failus, jo mēģināja darīt ārpus kenkins direktorijas
    }
    stages {
        stage('install-pip-deps') {
            steps {
                script {
                    echo "Cloning python-greetings repo..."
                    git branch: 'main', poll: false, url: 'https://github.com/mtararujs/python-greetings.git'
                    dir('python-greetings') {
                        echo "Displaying repo content..."
                        bat 'dir'
                        echo "Installing pip packages..."
                        bat 'pip3 install -r requirements.txt'
                    }
                }
            }
        }
        stage('deploy-to-dev') {
            steps {
                script {
                    deploy('dev', 7001)
                }
            }
        }
        stage('tests-on-dev') {
            steps {
                script {
                    test('dev')
                }
            }
        }
        stage('deploy-to-staging') {
            steps {
                script {
                    deploy('staging', 7002)
                }
            }
        }
        stage('tests-on-staging') {
            steps {
                script {
                    test('staging')
                }
            }
        }
        stage('deploy-to-preprod') {
            steps {
                script {
                    deploy('preprod', 7003)
                }
            }
        }
        stage('tests-on-preprod') {
            steps {
                script {
                    test('preprod')
                }
            }
        }
        stage('deploy-to-prod') {
            steps {
                script {
                    deploy('prod', 7004)
                }
            }
        }
        stage('tests-on-prod') {
            steps {
                script {
                    test('prod')
                }
            }
        }
    }
}

def deploy(String env, int port) {
    echo "Deploying to ${env} environment..."
    echo "Cloning python-greetings repo..."
    git branch: 'main', poll: false, url: 'https://github.com/mtararujs/python-greetings.git'
    dir('python-greetings') {
        echo "Deleting service if exists: greetings-app-${env}..."
        bat "npx pm2 delete greetings-app-${env} || exit 0" // norādīts npx, jo savādāk pm2 neņēma pretī
        echo "Starting service ${env} on port ${port}..."
        bat "npx pm2 start app.py --name greetings-app-${env} -- --port ${port}"
    }
}

def test(String env) {
    echo "Testing ${env} environment..."
    echo "Cloning course-js-api-framework repo..."
    git branch: 'main', poll: false, url: 'https://github.com/mtararujs/course-js-api-framework.git'
    dir('course-js-api-framework') {
        echo "Installing npm packages..."
        bat "npm install"
        echo "Running test greetings_${env}..."
        bat "npm run greetings greetings_${env}"
        echo "Deleting service if exists: greetings-app-${env}..." // pievienots cleanup pēc testa pabeigšanas
        bat "npx pm2 delete greetings-app-${env} || exit 0"
    }
}

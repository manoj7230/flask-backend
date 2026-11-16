pipeline {
    agent any
    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }
    environment {
        APP_DIR = '/opt/jenkins-cicd/flask-backend'
        VENV = '/opt/jenkins-cicd/venvs/flask'
        SERVICE_NAME = 'flask-backend'
        HEALTH_URL = 'http://127.0.0.1:5000/health'
    }
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Install dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/python -m pip install --upgrade pip
                    .venv/bin/pip install -r requirements.txt pytest
                '''
            }
        }
        stage('Unit tests') { steps { sh '.venv/bin/pytest -q' } }
        stage('Deploy to EC2') {
            steps {
                sh '''
                    sudo mkdir -p "$APP_DIR" "$VENV"
                    sudo rsync -a --delete --exclude=.git --exclude=.venv ./ "$APP_DIR"/
                    sudo python3 -m venv "$VENV"
                    sudo "$VENV/bin/pip" install --upgrade pip
                    sudo "$VENV/bin/pip" install -r "$APP_DIR/requirements.txt"
                    sudo systemctl restart "$SERVICE_NAME"
                    sudo systemctl is-active --quiet "$SERVICE_NAME"
                '''
            }
        }
        stage('Health verification') {
            steps {
                sh '''
                    for attempt in 1 2 3 4 5 6 7 8 9 10; do
                      if curl --fail --silent "$HEALTH_URL"; then exit 0; fi
                      sleep 3
                    done
                    echo "Flask health endpoint did not become healthy"
                    sudo journalctl -u "$SERVICE_NAME" -n 80 --no-pager || true
                    exit 1
                '''
            }
        }
    }
    post {
        success { echo 'Flask tests passed and EC2 service is healthy.' }
        failure { echo 'Flask pipeline failed. Review the stage logs and systemd journal.' }
        always { cleanWs() }
    }
}

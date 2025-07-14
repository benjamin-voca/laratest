pipeline {
    agent any

    environment {
        APP_DIR = "${WORKSPACE}"
        DEPLOY_BASE = "/var/www/laravel"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://your-repo.git'
            }
        }

        stage('Setup Node & PHP') {
            steps {
                sh 'corepack enable' // enables pnpm
                sh 'pnpm install'
                sh 'composer install --no-interaction --prefer-dist'
            }
        }

        stage('Laravel Setup') {
            steps {
                sh '''
                    cp .env.example .env
                    php artisan key:generate
                    php artisan migrate --force || true
                '''
            }
        }

        stage('Deploy to NGINX') {
            steps {
                sh '''
                    BRANCH_CLEAN=$(echo ${BRANCH_NAME} | tr '/' '_')
                    TARGET_DIR="${DEPLOY_BASE}/${BRANCH_CLEAN}"
                    mkdir -p "$TARGET_DIR"
                    rsync -a --delete ${APP_DIR}/ "$TARGET_DIR/"
                    chown -R www-data:www-data "$TARGET_DIR"
                '''
            }
        }

        stage('Reload NGINX') {
            steps {
                sh 'sudo nginx -s reload'
            }
        }
    }
}

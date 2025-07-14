pipeline {
    agent any

    environment {
        APP_DIR = "${WORKSPACE}"
        DEPLOY_BASE = "${WORKSPACE}/deploy"
        PATH = "/Users/benjamin/Library/pnpm:/Users/benjamin/.ghcup/bin:/Users/benjamin/.config/composer/vendor/bin:/usr/local/opt/openjdk/bin:/Users/benjamin/.cargo/bin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/benjamin-voca/laratest'
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
                    rsync -a --delete ${WORKSPACE}/ "$TARGET_DIR/"
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


pipeline {
    agent any

    environment {
        APP_DIR     = "${WORKSPACE}"
        DEPLOY_BASE = "${WORKSPACE}/deploy"
        DOMAIN      = "myproj.local"
        NGINX_SITES = "/usr/local/etc/nginx/sites-enabled"
        PHP_FPM     = "127.0.0.1:9000"
        PORT        = "9000"
        PATH        = "/Users/benjamin/Library/pnpm:/Users/benjamin/.ghcup/bin:/Users/benjamin/.config/composer/vendor/bin:/usr/local/opt/openjdk/bin:/Users/benjamin/.cargo/bin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/benjamin-voca/laratest'
            }
        }

        stage('Setup Node & PHP') {
            steps {
                sh 'corepack enable'
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

        stage('Generate NGINX Config')     stage('Generate NGINX Config') {
        steps {
            sh("""
                # sanitize branch name
                BRANCH_CLEAN=\$(echo ${BRANCH_NAME} | tr '/' '_')

                # add hosts entry if missing
                if ! grep -q "\${BRANCH_CLEAN}.${DOMAIN}" /etc/hosts; then
                  echo "127.0.0.1 \${BRANCH_CLEAN}.${DOMAIN}" | sudo tee -a /etc/hosts > /dev/null
                fi

                # write nginx conf
                CONFIG_FILE="${NGINX_SITES}/\${BRANCH_CLEAN}.conf"
                sudo tee "\${CONFIG_FILE}" > /dev/null << 'NGINXCONF'
server {
    listen ${PORT};
    server_name ${BRANCH_CLEAN}.${DOMAIN};

    root ${DEPLOY_BASE}/${BRANCH_CLEAN}/public;
    index index.php index.html;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    # this block now writes: location ~ \.php\$ {
    location ~ \\.php\\$ {
        include fastcgi_params;
        fastcgi_pass ${PHP_FPM};
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
    }

    # and for static assets:
    location ~* \\.(js|css|png|jpg|jpeg|gif|ico|svg|ttf|woff|woff2|eot)\$ {
        access_log off;
        expires max;
    }
}
NGINXCONF

                echo "✔ Generated Nginx config for branch \${BRANCH_CLEAN} at \${CONFIG_FILE}"
            """.stripIndent())
        }
    }


        stage('Reload NGINX') {
            steps {
                sh 'sudo nginx -t && sudo nginx -s reload'
            }
        }
    }
}

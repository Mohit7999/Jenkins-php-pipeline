pipeline {

    agent any

    environment {
        DEPLOY_PATH = "/var/www/laravel-app"
    }

    stages {

        stage('Clean Workspace') {

            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {

            steps {

                git branch: 'main',
                url: 'https://github.com/Mohit7999/Jenkins-php-pipeline.git'
            }
        }

        stage('Install Composer Dependencies') {

            steps {

                sh '''
                composer install --no-dev --optimize-autoloader
                '''
            }
        }

        stage('Laravel Environment Setup') {

            steps {

                sh '''
                if [ ! -f .env ]; then
                    cp .env.example .env
                fi
                '''
            }
        }

        stage('Generate Laravel Key') {

            steps {

                sh '''
                php artisan key:generate || true
                '''
            }
        }

        stage('Laravel Optimization') {

            steps {

                sh '''
                php artisan config:clear
                php artisan cache:clear
                php artisan route:clear
                php artisan view:clear

                php artisan config:cache
                php artisan route:cache
                php artisan view:cache
                php artisan optimize
                '''
            }
        }

        stage('Deploy Application') {

            steps {

                sh '''
                rsync -av --delete \
                --exclude='.git' \
                --exclude='node_modules' \
                ./ $DEPLOY_PATH
                '''
            }
        }

        stage('Fix Permissions') {

            steps {

                sh '''
                chmod -R 775 $DEPLOY_PATH/storage
                chmod -R 775 $DEPLOY_PATH/bootstrap/cache
                '''
            }
        }

        stage('Restart Services') {

            steps {

                sh '''
                sudo systemctl restart nginx
                sudo systemctl restart php-fpm
                '''
            }
        }
    }

    post {

        success {

            echo 'Laravel Deployment Successful!'
        }

        failure {

            echo 'Laravel Deployment Failed!'
        }
    }
}

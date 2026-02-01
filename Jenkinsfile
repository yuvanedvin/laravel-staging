pipeline {
    agent any

    stages {
        stage('Sync Laravel Env') {
            steps {
                withCredentials([file(credentialsId: 'laravel-env-prod', variable: 'ENV_FILE')]) {
                    sh '''
                      set -e

                      rsync -avz "$ENV_FILE" deploy@10.0.0.20:/tmp/.env

                      ssh deploy@10.0.0.20 "
                        sudo -u apache mv /tmp/.env /var/www/laravel-app/.env &&
                        sudo -u apache chmod 600 /var/www/laravel-app/.env
                      "
                    '''
                }
            }
        }
    }
}

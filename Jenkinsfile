pipeline {
    agent any

    stages {
        stage('Sync Laravel Env & Reload Config') {
            steps {
                withCredentials([file(credentialsId: 'laravel-env-prod', variable: 'ENV_FILE')]) {
                    sh '''
                      set -e

                      TARGET_USER=root
                      TARGET_HOST=172.31.43.5
                      APP_PATH=/var/web-laravel/frontend/
                      echo ">>> Syncing .env as apache"
                      rsync -avz \
                        --rsync-path="sudo -u apache rsync" \
                        "$ENV_FILE" \
                        ${TARGET_USER}@${TARGET_HOST}:${APP_PATH}/.env

                      echo ">>> Setting permissions"
                      ssh ${TARGET_USER}@${TARGET_HOST} "
                        sudo -u apache chmod 600 ${APP_PATH}/.env
                      "

                      # echo ">>> Reloading Laravel config"
                      # ssh ${TARGET_USER}@${TARGET_HOST} "
                      #   cd ${APP_PATH} &&
                      #   sudo -u apache php artisan config:clear &&
                      #   sudo -u apache php artisan config:cache
                      # "

                      echo ">>> DONE"
                    '''
                }
            }
        }
    }
}

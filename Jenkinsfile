pipeline {
    agent any

    stages {
        stage('Sync Laravel Env') {
            steps {
                sh '''
                  set -e

                  TARGET_USER=root
                  TARGET_HOST=172.31.43.5
                  APP_PATH=/var/web-laravel/frontend/
                  ENV_FILE=.env   # from repo

                  echo ">>> Syncing .env from repo"
                  rsync -avz \
                    --rsync-path="sudo -u apache rsync" \
                    "$ENV_FILE" \
                    ${TARGET_USER}@${TARGET_HOST}:${APP_PATH}/.env

                  echo ">>> Setting permissions"
                  ssh ${TARGET_USER}@${TARGET_HOST} "
                    sudo -u apache chmod 600 ${APP_PATH}/.env
                  "

                  echo ">>> DONE"
                '''
            }
        }
    }
}

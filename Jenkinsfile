pipeline {
    agent any

    parameters {
        choice(
            name: 'TARGET',
            choices: ['frontend', 'my-project'],
            description: 'Deployment target'
        )
    }

    stages {
        stage('Resolve Target Configuration') {
            steps {
                script {
                    if (params.TARGET == 'frontend') {
                        env.TARGET_HOST = '172.31.43.5'
                        env.APP_PATH   = '/var/web-laravel/frontend'
                        env.ENV_FILE   = '.env'
                    } else if (params.TARGET == 'my-project') {
                        env.TARGET_HOST = '172.31.43.5'
                        env.APP_PATH   = '/var/web-laravel/my-project'
                        env.ENV_FILE   = '.env'
                    } else {
                        error "Unknown TARGET: ${params.TARGET}"
                    }
                }
            }
        }

        stage('Sync Laravel Env & Reload Config') {
            steps {
                sh '''
                  set -e

                  TARGET_USER=apache

                  echo ">>> Deploying to: ${TARGET}"
                  echo ">>> Host: ${TARGET_HOST}"
                  echo ">>> App path: ${APP_PATH}"

                  echo ">>> Syncing env file"
                  rsync -avz \
                    --rsync-path="sudo -u apache rsync" \
                    "${ENV_FILE}" \
                    ${TARGET_USER}@${TARGET_HOST}:${APP_PATH}/.env

                  echo ">>> Setting permissions"
                  ssh ${TARGET_USER}@${TARGET_HOST} \
                    "sudo -u apache chmod 600 ${APP_PATH}/.env"

                  echo ">>> Reloading Laravel config"
                  ssh ${TARGET_USER}@${TARGET_HOST} "
                    cd ${APP_PATH} &&
                    sudo -u apache php artisan config:clear &&
                    sudo -u apache php artisan config:cache
                  "

                  echo ">>> DONE"
                '''
            }
        }
    }
}

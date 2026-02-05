pipeline {
    agent any

    parameters {
        choice(
            name: 'TARGET',
            choices: ['my-project', 'frontend2'],
            description: 'Select deployment target'
        )
    }

    stages {

        stage('Resolve Configuration') {
            steps {
                script {
                    // SSH login user (ALWAYS a login user)
                    env.TARGET_USER = 'deploy'

                    if (params.TARGET == 'my-project') {
                        env.TARGET_HOST = '3.69.167.61'
                        env.APP_PATH   = '/var/web-laravel/my-project'
                        env.ENV_FILE   = '.env.my-project'

                    } else if (params.TARGET == 'frontend2') {
                        env.TARGET_HOST = '3.69.167.61'
                        env.APP_PATH   = '/var/web-laravel/frontend2'
                        env.ENV_FILE   = '.env.frontend2'

                    } else {
                        error "❌ Unknown TARGET: ${params.TARGET}"
                    }
                }
            }
        }

        stage('Deploy Env & Reload Config') {
            steps {
                sh '''
                  set -e

                  echo "===================================="
                  echo ">>> Deploying TARGET : ${TARGET}"
                  echo ">>> SSH User         : ${TARGET_USER}"
                  echo ">>> Host             : ${TARGET_HOST}"
                  echo ">>> App Path         : ${APP_PATH}"
                  echo ">>> Env File         : ${ENV_FILE}"
                  echo "===================================="

                  # Safety check
                  if [ ! -f "${ENV_FILE}" ]; then
                    echo "❌ Env file not found: ${ENV_FILE}"
                    exit 1
                  fi

                  echo ">>> Syncing .env as apache"
                  rsync -avz \
                    --rsync-path="sudo -u apache rsync" \
                    "${ENV_FILE}" \
                    ${TARGET_USER}@${TARGET_HOST}:${APP_PATH}/.env

                  echo ">>> Fixing permissions"
                  ssh ${TARGET_USER}@${TARGET_HOST} "
                    sudo -u apache chmod 600 ${APP_PATH}/.env
                  "

                  echo ">>> Reloading Laravel config"
                  ssh ${TARGET_USER}@${TARGET_HOST} "
                    cd ${APP_PATH} &&
                    sudo -u apache php artisan config:clear &&
                    sudo -u apache php artisan config:cache
                  "

                  echo ">>> DEPLOYMENT COMPLETED"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: Deployment completed for ${TARGET}"
        }
        failure {
            echo "❌ FAILURE: Deployment failed for ${TARGET}"
        }
    }
}

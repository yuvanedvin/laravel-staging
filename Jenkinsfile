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
        stage('Checkout Env Repository') {
            steps {
                dir('env-repo') {
                    git(
                        url: 'https://github.com/yuvanedvin/Laravel-ENV-repo.git',
                        branch: 'master',
                        credentialsId: 'github-pat' // Github PAT ID
                    )
                }
            }
        }

        stage('Resolve Configuration') {
            steps {
                script {
                    env.TARGET_USER = 'deploy'
                    env.TARGET_HOST = '3.69.167.61'

                    if (params.TARGET == 'my-project') {
                        env.APP_PATH = '/var/web-laravel/my-project'
                        env.ENV_FILE = 'env-repo/my-project/.env'
                    } else if (params.TARGET == 'frontend2') {
                        env.APP_PATH = '/var/web-laravel/frontend2'
                        env.ENV_FILE = 'env-repo/frontend2/.env'
                    } else {
                        error "Unknown TARGET: ${params.TARGET}"
                    }
                }
            }
        }

        stage('Deploy Env & Reload Config') {
            steps {
                sh '''
                  set -e
                  rsync -avz --rsync-path="sudo -u apache rsync" \
                    "${ENV_FILE}" \
                    ${TARGET_USER}@${TARGET_HOST}:${APP_PATH}/.env

                  ssh ${TARGET_USER}@${TARGET_HOST} "
                    sudo -u apache chmod 600 ${APP_PATH}/.env &&
                    cd ${APP_PATH} &&
                    sudo -u apache php artisan config:clear &&
                    sudo -u apache php artisan config:cache
                  "
                '''
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: Deployment completed for ${params.TARGET}"
        }
        failure {
            echo "❌ FAILURE: Deployment failed for ${params.TARGET}"
        }
    }
}

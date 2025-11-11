pipeline {

    agent any

    environment {

        DEPLOY_PATH = "/var/www/html/laravel_jenkins"

        VM_1 = "bruno@10.255.255.51"

        VM_2 = "bruno@10.255.255.64"

    }

    stages {

        stage("Fazer o Checkout") {

            steps {

                checkout scm

            }

        }

        stage("Instalar as Dependências") {

            steps {

                sh "composer install --no-interaction --prefer-dist --optimize-autoloader"

                sh "npm ci && npm run build"

            }

        }

        stage("Realizar os Testes") {

            steps {

                sh "php artisan test"

            }

        }

        stage("Aplicar o Merge na 'main'") {

            when {

                not {
                    
                    branch "main" 
                    
                }

            }

            steps {

                sshagent (
                    
                    [
                        
                        "git-ssh-key"
                        
                    ]) 
                    
                    {

                        sh '''

                            git config --global user.email "jenkins@brdsoft.com"

                            git config --global user.name "Jenkins"

                            git checkout main

                            git merge origin/${BRANCH_NAME} --no-ff -m "Merge automático da branch ${BRANCH_NAME}"

                            git push origin main

                        '''

                }

            }

        }

        stage("Realizar o Deploy nas VMs") {

            when {

                branch "main"

            }

            steps {

                sshagent([
                    
                    "vm-ssh-key"
                    
                    ]) {

                    sh '''

                        for VM in ${VM_1} ${VM_2}; do

                            ssh -o StrictHostKeyChecking=no $VM "

                                cd ${DEPLOY_PATH} &&

                                git pull origin main &&

                                composer install --no-interaction --prefer-dist --optimize-autoloader &&

                                php artisan migrate --force &&

                                php artisan cache:clear &&

                                php artisan config:cache

                            "

                        done

                    '''
                }

            }

        }

    }

    post {

        success {

            echo "Pipeline concluído com sucesso!"

        }

        failure {

            echo "Erro ao executar a Pipeline!"

        }

    }

}
pipeline {

    // Definimos o Agente padrão para o Jekins, sendo este configurado para utilizar qualquer nó online.
    agent any

    // Setamos as Variáveis de Ambiente que serão utilizadas durante o processo.
    environment {

        // Definimos a Variável de Ambiente do caminho do repositório no servidor.
        DEPLOY_PATH = "/var/www/html/laravel_jenkins"

        // Definimos a Variável de Ambiente para a conexão SSH da primeira VM.
        VM_1 = "bruno@10.255.255.51"

        // Definimos a Variável de Ambiente para a conexão SSH da segunda VM.
        VM_2 = "bruno@10.255.255.64"

        // Definimos a Variável de Ambiente para o NodeJS.
        PATH = "/home/bruno/.nvm/versions/node/v24.3.0/bin:$PATH"

        // Definimos a Variável de Ambiente para a Branch solicitante.
        FEATURE_BRANCH = "${env.GIT_BRANCH.replace('origin/', '')}"

    }

    // Abrimos o Bloco aonde serão configuradas as etapas.
    stages {

        // Primeira Etapa: Fazer o checkout da Branch solicitante.
        stage("Fazer o Checkout da Branch") {

            steps {

                // Realiza o "git clone" e o "git checkout" apartir da branch solicitante.
                checkout scm

            }

        }


        // Segunda Etapa: Instalamos as dependências do PHP Composer e do Node NPM.
        stage("Instalar as Dependências") {

            steps {

                // Realizamos a instalação das dependências do PHP Composer.
                sh "composer install --no-interaction --prefer-dist --optimize-autoloader"

                // Realizamos a instação das dependências do Node NPM.
                sh "npm ci && npm run build"

            }

        }

        // Terçeira Etapa: Realizamos os testes da aplicação apartir do Laravel Test.
        stage("Realizar os Testes") {

            steps {

                // Executamos o comando de testes do Laravel.
                sh '''

                    cd ${DEPLOY_PATH}

                    php artisan optimize:clear

                    php artisan test

                '''

            }

        }

        // Quarta Etapa: Aplicamos o Merge na "main".
        stage("Aplicar o Merge na 'main'") {

            // Definimos o Merge apenas para as branchs que não são a própria "main".
            when {

                not {
                    
                    branch "main" 
                    
                }

            }

            steps {

                // Executamos o Merge na "main".
                sshagent (
                    
                    [
                        
                        // Passamos a credencial da Chave SSH cadastrada no Jenkins.
                        "git-ssh-key"
                        
                    ]) 
                    
                    {

                        // Aplicamos a Configuração do Git para o Jenkins; Realizamos o Checkout na Branch "main"; Realizamos o merge da Branch solicitante na "main"; e Enviamos as mudanças para o Branch Remota "main".
                        sh '''

                            git config --global user.email "jenkins@brdsoft.com"

                            git config --global user.name "Jenkins"

                            git remote set-url origin git@github.com:BrunoGsouza90/laravel_jenkins.git

                            git fetch origin ${FEATURE_BRANCH}

                            git checkout main -f

                            git reset --hard origin/main

                            git merge origin/${FEATURE_BRANCH} --no-ff -m "Merge automático da branch ${FEATURE_BRANCH}"

                            echo "A branch é: ${FEATURE_BRANCH}"
                            
                            git push origin main -f

                        '''

                }

            }

        }

        // Quinta Etapa: Realizamos o Deploy nas VMs.
        stage("Realizar o Deploy nas VMs") {

            // Quando ouver modificações na Branch remota "main".
            when {

                expression {

                    env.GIT_BRANCH == "origin/main"

                }

            }

            steps {

                // Cadastramos a Chave SSH do GitHub apartir da Credêncial Cadastrada no Jenkins; Realizamos um "for" em ambas as VMs.
                sshagent(['vm-ssh-key']) {

                    sh """

                        for VM in ${VM_1} ${VM_2}; do
                            ssh -o StrictHostKeyChecking=no "\$VM" '
                                cd ${DEPLOY_PATH} &&
                                git pull origin main &&
                                composer install --no-interaction --prefer-dist --optimize-autoloader &&
                                php artisan migrate --force &&
                                php artisan cache:clear &&
                                php artisan config:cache
                            '
                        done
                    """
                }

            }

        }

    }

    // Sexta Etapa: Indenficamos no Arquivo de Log do Jenkins a reposta do Pipeline.
    post {

        // Caso tenha dado tudo certo.
        success {

            // Imprimimos no Log a mensagem de sucesso.
            echo "Pipeline concluído com sucesso!"

        }

        // Caso teha dado algo errado.
        failure {

            // Informamos no Log que a execução teve falhas ou erros.
            echo "Erro ao executar a Pipeline!"

        }

    }

}

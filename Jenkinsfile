pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL',
               defaultValue: 'https://github.com/Spalumn/cicd-demo-app',
               description: 'HTTPS-ссылка на репозиторий')
        string(name: 'GIT_BRANCH',
               defaultValue: 'master',
               description: 'Ветка для сборки')
        booleanParam(name: 'RUN_TESTS',
                     defaultValue: true,
                     description: 'Выполнять unit-тесты')
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                git url: params.REPO_URL, branch: params.GIT_BRANCH
                script {
                    env.GIT_SHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                }
                echo "Ветка: ${params.GIT_BRANCH}"
                echo "Полный SHA коммита: ${env.GIT_SHA}"
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    for f in README.md requirements.txt; do
                        [ -f "$f" ] || { echo "ОШИБКА: нет файла $f"; exit 1; }
                    done
                    for d in src tests; do
                        [ -d "$d" ] || { echo "ОШИБКА: нет каталога $d"; exit 1; }
                    done
                    echo "Структура проекта в порядке"
                '''
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS }
            }
            steps {
                sh 'python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Build number: ${BUILD_NUMBER}" > build-info.txt
                    echo "Commit SHA: ${GIT_SHA}" >> build-info.txt
                    tar -czf app.tar.gz src tests README.md requirements.txt build-info.txt
                    tar -tzf app.tar.gz
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "Сборка #${env.BUILD_NUMBER} завершилась УСПЕШНО"
        }
        failure {
            echo "Сборка #${env.BUILD_NUMBER} завершилась НЕУСПЕШНО"
        }
    }
}

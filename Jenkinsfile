pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/Dimkatf/cicd-demo-app.git', description: 'URL репозитория')
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Ветка для сборки')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Запускать ли unit-тесты?')
    }

    stages {
        stage('Checkout') {
            steps {
                // Получаем код по параметрам
                git url: "${params.REPO_URL}", branch: "${params.GIT_BRANCH}"
                
                // Выводим имя ветки и SHA в консоль
                script {
                    def commitSha = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Ветка: ${params.GIT_BRANCH}"
                    echo "Полный SHA коммита: ${commitSha}"
                }
            }
        }

        stage('Validate') {
            steps {
                script {
                    // Проверка обязательных файлов и папок
                    def required = ['README.md', 'requirements.txt', 'src', 'tests']
                    def missing = []
                    required.each { item ->
                        if (!fileExists(item)) {
                            missing.add(item)
                        }
                    }
                    if (!missing.isEmpty()) {
                        error "Отсутствуют обязательные элементы: ${missing.join(', ')}"
                    }
                    echo "Структура проекта валидна."
                }
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                // Запуск тестов
                sh 'python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                script {
                    def commitSha = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    // Создаём файл с информацией о сборке
                    writeFile file: 'build-info.txt', text: "Номер сборки Jenkins: ${env.BUILD_NUMBER}\nSHA коммита: ${commitSha}\n"
                    
                    // Создаём архив (включая скрытые файлы, если есть)
                    sh 'tar -czf app.tar.gz * .[!.]* 2>/dev/null || tar -czf app.tar.gz *'
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
                echo "Артефакт app.tar.gz сохранён."
            }
        }
    }

    post {
        success {
            echo "Сборка успешно завершена."
        }
        failure {
            echo "Сборка завершилась с ошибкой. Проверьте логи стадий."
        }
    }
}

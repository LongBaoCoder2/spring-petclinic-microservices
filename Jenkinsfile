pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token') 
    }

    stages {
        stage('Check if PR to main') {
            when {
                expression { return env.CHANGE_ID && env.CHANGE_TARGET == 'main' }
            }
            steps {
                echo "Pull request vào main, tiếp tục kiểm tra!"
            }
        }

        stage('Determine changed services') {
            steps {
                script {
                    def changedFiles
                    if (env.CHANGE_TARGET) {  
                        changedFiles = sh(returnStdout: true, script: "git diff --name-only origin/${env.CHANGE_TARGET}...HEAD").trim().split('\n')
                    } else {  
                        changedFiles = sh(returnStdout: true, script: "git diff --name-only HEAD^ HEAD").trim().split('\n')
                    }

                    def affectedServices = changedFiles.collect { file ->
                        if (file.startsWith('spring-petclinic-')) {
                            return file.split('/')[0]
                        }
                    }.unique()

                    env.AFFECTED_SERVICES = affectedServices.join(',')
                    echo "Affected services: ${env.AFFECTED_SERVICES}"
                }
            }
        }

        stage('Test affected services') {
            steps {
                script {
                    if (env.AFFECTED_SERVICES) {
                        sh "mvn -pl ${env.AFFECTED_SERVICES} test"
                        junit '**/target/surefire-reports/*.xml'
                    } else {
                        echo 'Không có dịch vụ nào bị ảnh hưởng, bỏ qua giai đoạn test'
                    }
                }
            }
        }

        stage('Build affected services') {
            steps {
                script {
                    if (env.AFFECTED_SERVICES) {
                        sh "mvn -pl ${env.AFFECTED_SERVICES} package -DskipTests"
                    } else {
                        echo 'Không có dịch vụ nào bị ảnh hưởng, bỏ qua giai đoạn build'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline hoàn tất!'
        }
        success {
            script {
                if (env.CHANGE_ID) {
                    sh """
                    curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
                         -H "Accept: application/vnd.github.v3+json" \
                         https://api.github.com/repos/LongBaoCoder2/spring-petclinic-microservices/statuses/${env.GIT_COMMIT} \
                         -d '{"state": "success", "description": "Tests passed!", "context": "Jenkins CI"}'
                    """
                }
            }
        }
        failure {
            script {
                if (env.CHANGE_ID) { 
                    sh """
                    curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \
                         -H "Accept: application/vnd.github.v3+json" \
                         https://api.github.com/repos/LongBaoCoder2/spring-petclinic-microservices/statuses/${env.GIT_COMMIT} \
                         -d '{"state": "failure", "description": "Tests failed!", "context": "Jenkins CI"}'
                    """
                }
            }
        }
    }
}

pipeline {
    agent any

    stages {
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
        failure {
            echo 'Pipeline thất bại!'
        }
    }
}
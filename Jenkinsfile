pipeline {
    agent any

    stages {
        stage('Determine changed services') {
            steps {
                script {
                    def changedFiles
                    if (env.CHANGE_TARGET) {
                        // Trường hợp chạy trong PR
                        changedFiles = sh(returnStdout: true, script: "git diff --name-only origin/${env.CHANGE_TARGET}...HEAD").trim().split('\n')
                    } else {
                        // Trường hợp chạy trên nhánh chính (push trực tiếp)
                        changedFiles = sh(returnStdout: true, script: "git diff --name-only HEAD^ HEAD").trim().split('\n')
                    }
                    
                    // Xác định các dịch vụ bị ảnh hưởng
                    def affectedServices = changedFiles.collect { file ->
                        if (file.startsWith('spring-petclinic-')) {
                            return file.split('/')[0]
                        }
                    }.unique()
                    
                    // Lưu danh sách dịch vụ bị ảnh hưởng
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
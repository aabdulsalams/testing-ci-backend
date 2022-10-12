pipeline {
    agent any
    
    environment {
        app_name_basic = 'api-360-client-portal-frontend'
    }
    
    stages {
        stage("Set Variables") {
            steps {
                script {
                    env.feature_name = scm.getUserRemoteConfigs()[0].getUrl().tokenize('/').last().split("\\.")[0]
                    if ("${env.BRANCH_NAME}" == "master") {
                        env.service_dir = "${feature_name}/prod"
                        env.automationdir = "automation-testing-backend/prod"
                        env.postman_api_key = "PMAK-63453627151d4515982198df-f708876fa18c1323f129f5964bd74609d7 "
                        env.postman_collection_id = "23807514-49c16453-e011-4092-a862-69c997a18985"
                        env.postman_environment_id = "23807514-09130740-1238-4391-a6ad-fb9a79a6e83a"
                    }
                    else if ("${env.BRANCH_NAME}" == "testing") {
                        env.service_dir = "${feature_name}/testing"
                        env.automationdir = "automation-testing-backend/testing"
                    }
                    else if ("${env.BRANCH_NAME}" == "release") {
                        env.service_dir = "${feature_name}/prerelase"
                        env.automationdir = "automation-testing-backend/prerelease"
                    }
                    else {
                        env.automationdir = "automation-testing-backend/unknown"
                        env.service_dir = "${feature_name}/unknown"
                    }
                }
            }
        }
        stage('Pull Automation') {
            // '${BRANCH_NAME}' ganti sementara 'master'
            steps {
                sh "mkdir -p ${automationdir} || true"
                dir("${automationdir}") {
                    git branch: 'master', url: 'https://github.com/aabdulsalams/testing-ci-postman.git'
                }
            }
        }
        stage("Run Automation Testing") {
            steps {
              dir("${automationdir}") {
                  sh "pwd && ls -al"
//                 sh "chmod +x run-testing.sh"
//                 sh "./run-testing.sh ${postman_api_key} ${postman_collection_id} ${postman_environment_id}| tee output.log"
//                 sh '''
//                     if grep -q "Failures: 0, Skips: 0" output.log; then 
//                         echo "Test run successfully! :)"
//                     else
//                         echo "There are failure/skip! :("
//                         exit 1
//                     fi
//                 '''         
               }
            }
        }
    }
    
    post {
        always {
            echo "Release finished do cleanup"
            deleteDir()
        }
        success {
            echo "Release Success"
        }
        failure {
            echo "Release Failed"
        }
        cleanup {
            echo "Clean up in post work space"
            cleanWs()
        }
    }
}

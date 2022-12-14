pipeline {
    agent any
    
    environment {
        app_name_basic = 'api-360-client-portal-frontend'
        basic_storage_endpoint = 'nanda-test/testing-postman-backend'
    }
    
    stages {
        stage("Set Variables") {
            steps {
                script {
                    env.feature_name = scm.getUserRemoteConfigs()[0].getUrl().tokenize('/').last().split("\\.")[0]
                    if ("${env.BRANCH_NAME}" == "master") {
                        env.service_dir = "${feature_name}/prod"
                        env.automationdir = "automation-testing-backend/prod"
                        env.storage_endpoint = "${basic_storage_endpoint}/${feature_name}/prod"
                        env.postman_api_key = "PMAK-63453627151d4515982198df-f708876fa18c1323f129f5964bd74609d7"
                        env.postman_collection_id = "23807514-49c16453-e011-4092-a862-69c997a18985"
                        env.postman_environment_id = "23807514-09130740-1238-4391-a6ad-fb9a79a6e83a"
                    }
                    else if ("${env.BRANCH_NAME}" == "testing") {
                        env.service_dir = "${feature_name}/testing"
                        env.automationdir = "automation-testing-backend/testing"
                        env.storage_endpoint = "${basic_storage_endpoint}/${feature_name}/testing"
                    }
                    else if ("${env.BRANCH_NAME}" == "release") {
                        env.service_dir = "${feature_name}/prerelase"
                        env.automationdir = "automation-testing-backend/prerelease"
                        env.storage_endpoint = "${basic_storage_endpoint}/${feature_name}/prerelease"
                    }
                    else {
                        env.automationdir = "automation-testing-backend/unknown"
                        env.service_dir = "${feature_name}/unknown"
                        env.storage_endpoint = "${basic_storage_endpoint}/${feature_name}/unknown"
                    }
                }
            }
        }
        stage('Pull Automation') {
            // '${BRANCH_NAME}' ganti sementara 'master'
            steps {
                sh "mkdir -p ${automationdir} || true"
                dir("${automationdir}") {
                    // jika sudah pakai bit bucket tambahkan credentialsId
                    git branch: 'master', url: 'https://github.com/aabdulsalams/testing-ci-postman.git'
                }
            }
        }
        stage("Run Automation Testing") {
            steps {
              dir("${automationdir}") {
                  sh "pwd && ls -al"
                  sh "chmod +x run-testing.sh"
                  sh "./run-testing.sh ${postman_api_key} ${postman_collection_id} ${postman_environment_id} ${BUILD_NUMBER} | tee output.log"        
               }
            }
        }
        stage("Store Automation Result to Cloud Storage") {
           steps {
               dir("${automationdir}") {
                   withCredentials([file(credentialsId: 'cloud-storage-object-admin', variable: 'GC_KEY')]) {
                       sh "gcloud auth activate-service-account --key-file=${GC_KEY}"
                       sh "gsutil cp report-${BUILD_NUMBER}.html gs://${storage_endpoint}/"
                   }
               }
           }
        }
        // create stage to send link to qa
//         stage("Send Automation Result to Discord") {
//            steps {
//                sh """
//                     curl -i -H "Accept: application/json" -H "Content-Type:application/json" -X POST \
//                     --data '{"content": "Result = https://storage.googleapis.com/${storage_endpoint}/report-${BUILD_NUMBER}.html"}' \
//                     ${webhook_url}
//                """
//            }
//         }
        stage("Check Automation Result") {
           steps {
               dir("${automationdir}") {
                   // Add "exit 1" after echo "There are failure/skip! :(" if need to add blocker
                   withCredentials([string(credentialsId: 'discord-webhook-url', variable: 'WEBHOOK_URL')]) {
                        sh """
                            if grep -q "Failures: 0, Skips: 0" report-${BUILD_NUMBER}.log; then 
                                echo "Test run successfully! :)"
                                curl -i -H "Accept: application/json" -H "Content-Type:application/json" -X POST --data '{"content": "Test run successfully! :)"}' ${WEBHOOK_URL}
                            else
                                echo "There are failure/skip! :("
                                curl -i -H "Accept: application/json" -H "Content-Type:application/json" -X POST --data '{"content": "There are failure/skip! :("}' ${WEBHOOK_URL}
                            fi
                        """       
                    }
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

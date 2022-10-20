pipeline {
    agent any
    
    environment {
        app_name_basic = 'api-360-client-portal-frontend'
        basic_storage_endpoint = 'nanda-test/testing-postman-backend'
        webhook_url = 'https://discord.com/api/webhooks/1030375663921803275/kLGXXrD2JY3WHnW3jBj2jKZ2zy6tQO2z6AcAp0dX6VfdutWbNREWFmbCjlzGmKV1T8Xj'
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
                  sh "hxnormalize -l 240 -x report-"$4" 2>/dev/null | hxselect -s '\n' -c ".card-success > div:nth-child(1) > h1:nth-child(3)" > test-summaries.log"
                  sh "cat test-summaries.log"
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
        stage("Store Automation Result to Cloud Storage") {
           steps {
               dir("${automationdir}") {
                   withCredentials([file(credentialsId: 'cloud-storage-object-admin', variable: 'GC_KEY')]) {
                       sh "gcloud auth activate-service-account --key-file=${GC_KEY}"
                       sh "gsutil cp report-${BUILD_NUMBER}.html gs://${storage_endpoint}/"
//                        sh "echo 'gs://${storage_endpoint}/report-${BUILD_NUMBER}.html' >  "
                    }
               }
           }
        }
        // create stage to send link to qa
        stage("Send Automation Result to Discord") {
           steps {
               sh """
                    curl -i -H "Accept: application/json" -H "Content-Type:application/json" -X POST \
                    --data '{"content": "Result = https://storage.googleapis.com/${storage_endpoint}/report-${BUILD_NUMBER}.html"}' \
                    ${webhook_url}
               """
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

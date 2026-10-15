pipeline {
    agent any

    environment {
        // Project number of this repo in the adso admin page
        ADSO_PROJECT_ID = '4'
    }

    stages {
        stage('Build') {
            steps {
                script { env.FAILED_STAGE = 'Build' }
                sh 'mvn -B -DskipTests package'
            }
        }
        stage('Test') {
            steps {
                script { env.FAILED_STAGE = 'Test' }
                sh 'mvn -B test'
            }
        }
    }

    post {
        failure {
            // Tell adso so it can analyse the failure and mail the cause and fix
            script {
                def body = groovy.json.JsonOutput.toJson([
                    build_url: env.BUILD_URL,
                    commit   : env.GIT_COMMIT ?: '',
                    branch   : env.BRANCH_NAME ?: (env.GIT_BRANCH ?: ''),
                    stage    : env.FAILED_STAGE ?: ''
                ])
                writeFile file: 'adso-failure.json', text: body
                withCredentials([string(credentialsId: 'adso-webhook-secret', variable: 'ADSO_SECRET')]) {
                    sh '''
                        SIG=$(openssl dgst -sha256 -hmac "$ADSO_SECRET" adso-failure.json | sed 's/^.* //')
                        curl -sS -X POST "$ADSO_URL/webhook/jenkins/$ADSO_PROJECT_ID" \
                          -H "Content-Type: application/json" \
                          -H "X-Adso-Signature: sha256=$SIG" \
                          --data-binary @adso-failure.json
                    '''
                }
            }
        }
    }
}

pipeline {

   environment {
        // Define timestamp variable at the top-level environment block
        TIMESTAMP = sh(script: "date +'%b-%d-t-%H-%M'", returnStdout: true).trim()
    }
    stages {
        stage('Cloning the repo') {
            steps {
                script {
                    // Clone the repository
                    git branch: 'main', url: 'https://github.com/amanravi-squareops/hystrix-dashboard'
                }
            }
        }
        stage('kaniko build & push') {
            steps {
                container('kaniko') {
                    script {
                        sh '''
                        ls
                        pwd 
                        /kaniko/executor --dockerfile /Dockerfile \
                        --context=$(pwd) \
                        --destination=amanravi12/hystrix-dashboard:build-${BUILD_NUMBER}-${TIMESTAMP}

                        '''
                    }
                }
            }
        }

        stage('Update values.yaml') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-cre', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        git branch: 'main', 
                            url: "https://${USERNAME}:${PASSWORD}@github.com/amanravi-squareops/springboot-helm.git"
                    }
                    sh '''
                    cd hystrix-dashboard
                    sed -i "s/tag: .*/tag: build-${BUILD_NUMBER}-${TIMESTAMP}/" values.yaml
                    cat values.yaml
                    git config --global user.email "aman.ravi@squareops.com"
                    git config --global user.name "amanravi-squareops"
                    git add values.yaml
                    git commit -m "Update imageTag in values.yaml"
                    git push origin main
                    '''
                }
            }
        }
    }
}

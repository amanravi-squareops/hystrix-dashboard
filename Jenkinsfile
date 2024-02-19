pipeline {
    environment {
        TIMESTAMP = sh(script: "date +'%b-%d-t-%H-%M'", returnStdout: true).trim()
    }
    stages {
        stage('Cloning the repo') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/amanravi-squareops/hystrix-dashboard.git'
                }
            }
        }
        stage('kaniko build & push') {
            agent {
                kubernetes {
                    yaml """
                    apiVersion: v1
                    kind: Pod
                    meta
                        name: kaniko
                    spec:
                        restartPolicy: Never
                        volumes:
                        - name: kaniko-secret
                          secret:
                            secretName: kaniko-secret
                        containers:
                        - name: kaniko
                          image: gcr.io/kaniko-project/executor:debug
                          command:
                            - /busybox/cat
                          tty: true
                          volumeMounts:
                          - name: kaniko-secret
                            mountPath: /kaniko/.docker
                    """
                }
            }
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
                            url: "https://github.com/amanravi-squareops/hystrix-dashboard.git"
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

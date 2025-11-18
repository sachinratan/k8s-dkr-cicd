Jenkins pipeline to build, push image to ECR and update helm for image tag

##### Jenkins Pipeline (testing):
```
pipeline {
    agent {
        kubernetes {
        cloud 'Amazon_EKS'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  #serviceAccountName: jenkins
  containers:
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run
  - name: kubectl
    image: alpine/k8s:1.28.3
    command:
    - cat
    tty: true
  - name: git
    image: alpine/git:latest
    command:
    - cat
    tty: true
  volumes:
  - name: docker-sock
    emptyDir: {}
"""
        }
    }
    
    environment {
        AWS_ACCOUNT_ID = 'xxxxxxxx'
        AWS_REGION = 'eu-central-1'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_REPOSITORY = 'git-jenkins-pipeline-repo'
        GITHUB_APP_REPO = 'https://github.com/sachinratan/SLRS-Admin-DevOps-Integration-Website.git'
        GITHUB_CREDENTIALS = 'github-ssh-key'
        AWS_CREDENTIALS = 'srs-cli-creds'
        GITHUB_HELM_REPO = 'https://github.com/sachinratan/jnks-k8s-integration-repo.git'
        HELM_VALUES_PATH = 'helm/values.yaml'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }
    
    stages {
        stage('Checkout') {
            steps {
                container('git') {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[
                            url: "${GITHUB_APP_REPO}"
                            //credentialsId: "${GITHUB_CREDENTIALS}"
                        ]]
                    ])
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}")
                    }
                }
            }
        }
        
        stage('Push to ECR') {
            steps {
                container('docker') {
                    script {
                        // Login to ECR using AWS CLI
                        withAWS(credentials: "${AWS_CREDENTIALS}", region: "${AWS_REGION}") {
                            def login = ecrLogin()
                            sh "${login}"
                        }
                
                        // Push images
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Checkout Helm Charts') {
            steps {
                container('git') {
                    dir('jnks-k8s-integration-repo') {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/main']],
                            userRemoteConfigs: [[url: "${GITHUB_HELM_REPO}"]]
                      ])
                    }
                }
            }
        }
        stage('verify Helm Charts files') {
            steps {
                container('git') {
                    dir('jnks-k8s-integration-repo') {
                        sh "pwd"
                        sh "ls -lrth"
                    }
                }
            }
        }
        stage('Update Helm Values') {
            steps {
                container('git') {
                    dir('jnks-k8s-integration-repo') {
                        script {
                            // Read the values.yaml file
                            def valuesYaml = readYaml file: 'values.yaml'
                        
                            // Update the image tag
                            valuesYaml.image.tag = "${IMAGE_TAG}"
                        
                            // Write back to values.yaml
                            writeYaml file: 'values.yaml', data: valuesYaml, overwrite: true
                        
                            // Commit and push changes
                            //withCredentials([sshUserPrivateKey(credentialsId: 'github-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                            //    sh """
                            //        git config --global --add safe.directory /home/jenkins/agent/workspace/test-0000/jnks-k8s-integration-repo
                            //        git status
                            //        git branch -a
                            //        git config user.email "sachinshinde741@gmail.com"
                            //        git config user.name "sachinratan"
                            //        git add values.yaml
                            //        git commit -m "Update image tag to ${IMAGE_TAG}"
                            //        # Use git branch to determine the current branch
                            //        CURRENT_BRANCH=\$(git rev-parse --abbrev-ref HEAD)
                            //        git push origin remotes/origin/main
                            //    """
                            withCredentials([sshUserPrivateKey(credentialsId: 'github-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                            sh """
                                # Setup SSH
                                mkdir -p ~/.ssh
                                cp \${SSH_KEY} ~/.ssh/id_rsa
                                chmod 600 ~/.ssh/id_rsa
                                ssh-keyscan github.com >> ~/.ssh/known_hosts

                                # Configure Git
                                git config --global --add safe.directory /home/jenkins/agent/workspace/test-0000/jnks-k8s-integration-repo
                                git remote add origin git@github.com:sachinratan/jnks-k8s-integration-repo.git
                                git remote -v
                                git config user.email "sachinshinde741@gmail.com"
                                git config user.name "sachinratan"

                                # Get current branch
                                CURRENT_BRANCH=\$(git rev-parse --abbrev-ref HEAD)

                                # Check for changes
                                git status
                                git add values.yaml

                                if git diff --staged --quiet; then
                                    echo "No changes to commit"
                                else
                                    git commit -m "Update image tag to ${IMAGE_TAG}"
                                    #git push origin \${CURRENT_BRANCH}
                                    git push origin main
                                    echo "Successfully pushed changes"
                                fi
                            """
                            }
                        }
                    }
                }
            }
        }
        
        //stage('Push to ECR') {
        //    steps {
        //        container('docker') {
        //            withAWS(credentials: "${AWS_CREDENTIALS}", region: "${AWS_REGION}") {
        //                script {
        //                    docker.withRegistry("${ECR_REGISTRY}", "ecr:${AWS_REGION}:${AWS_CREDENTIALS}") {
        //                        dockerImage.push()
        //                        dockerImage.push('latest')
        //                    }
        //                }
        //            }
        //        }
        //    }
        //}
        
 //       stage('Update Helm Values') {
 //           steps {
 //               container('kubectl') {
 //                   withCredentials([usernamePassword(
 //                       credentialsId: "${GITHUB_CREDENTIALS}",
 //                       usernameVariable: 'GIT_USERNAME',
 //                       passwordVariable: 'GIT_PASSWORD'
 //                   )]) {
 //                       script {
 //                           def valuesFile = readFile("${HELM_VALUES_PATH}")
 //                           def updatedValues = valuesFile.replaceAll(
 //                               /(image:\s*
//\s*tag:\s*).*/, 
//                                "\$1${IMAGE_TAG}"
//                            )
//                            writeFile file: "${HELM_VALUES_PATH}", text: updatedValues
//                            
//                            def gitUrl = "${GITHUB_REPO}".replaceAll('', "${GIT_USERNAME}:${GIT_PASSWORD}@")
//                            
//                            sh """
//                                git config user.email "jenkins@example.com"
//                                git config user.name "Jenkins CI"
//                                git add ${HELM_VALUES_PATH}
//                                git commit -m "Update image tag to ${IMAGE_TAG} [skip ci]"
//                                git push ${gitUrl} HEAD:main
//                            """
//                        }
//                    }
//                }
//            }
//        }
//    }
    }
    post {
        success {
            echo "Pipeline completed successfully! Image ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} pushed to ECR and Helm values updated."
        }
        failure {
            echo "Pipeline failed. Please check the logs for details."
        }
        cleanup {
            cleanWs()
        }
    }
}
```

Jenkins Pipeline (Working n tested):
```
pipeline {
    agent {
        kubernetes {
            cloud 'Amazon_EKS'
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  containers:
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run
  - name: kubectl
    image: alpine/k8s:1.28.3
    command:
    - cat
    tty: true
  - name: git
    image: alpine/git:latest
    command:
    - cat
    tty: true
    tty: true
  volumes:
  - name: docker-sock
    emptyDir: {}
'''
        }
    }
    
    environment {
        AWS_ACCOUNT_ID = '559781698655'
        AWS_REGION = 'eu-central-1'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_REPOSITORY = 'git-jenkins-pipeline-repo'
        GITHUB_APP_REPO = 'git@github.com:sachinratan/SLRS-Admin-DevOps-Integration-Website.git'
        GITHUB_CREDENTIALS = 'jnks-github-ssh-auth-creds'
        AWS_CREDENTIALS = 'srs-cli-creds'
        GITHUB_HELM_REPO = 'git@github.com:sachinratan/jnks-k8s-integration-repo.git'
        IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_SSH_COMMAND = 'ssh -o StrictHostKeyChecking=no'
        GIT_BRANCH = 'main'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }
    
    stages {
        stage('Setup Git and SSH') {
            steps {
                container('git') {
                    withCredentials([sshUserPrivateKey(credentialsId: 'jnks-github-ssh-auth-creds', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                            # Configure Git
                            git config --global user.name "sachinratan"
                            git config --global user.email "sachinshinde741@gmail.com"
                            git config --global --add safe.directory '*'

                            # Setup SSH
                            mkdir -p ~/.ssh
                            cp $SSH_KEY ~/.ssh/id_rsa
                            chmod 600 ~/.ssh/id_rsa
                            ssh-keyscan github.com >> ~/.ssh/known_hosts
                        '''
                    }
                }
            }
        }
        stage('Clone application code repo') {
            steps {
                container('git') {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[
                            url: "${GITHUB_APP_REPO}",
                            credentialsId: "${GITHUB_CREDENTIALS}"
                        ]]
                    ])
                }
            }
        }
        
        stage('Build App Docker Image') {
            steps {
                container('docker') {
                    script {
                        dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}")
                    }
                }
            }
        }
        
        stage('Push to App Image to ECR') {
            steps {
                container('docker') {
                    script {
                        withAWS(credentials: "${AWS_CREDENTIALS}", region: "${AWS_REGION}") {
                            def login = ecrLogin()
                            sh "${login}"
                        }
                        
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Checkout Helm Charts Repo') {
            steps {
                container('git') {
                    dir('jnks-k8s-integration-repo') {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: 'main']],
                            userRemoteConfigs: [[
                                url: "${GITHUB_HELM_REPO}",
                                credentialsId: "${GITHUB_CREDENTIALS}"
                            ]]
                        ])
                    }
                }
            }
        }

        stage('Verify Helm Charts files') {
            steps {
                container('git') {
                    dir('jnks-k8s-integration-repo') {
                        sh 'pwd'
                        sh 'ls -lrth'
                    }
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                container('git') {
                    dir('jnks-k8s-integration-repo') {
                        script {
                            def valuesYaml = readYaml file: 'values.yaml'
                            valuesYaml.image.tag = "${IMAGE_TAG}"
                            writeYaml file: 'values.yaml', data: valuesYaml, overwrite: true
                            
                            sh '''
                                git checkout ${GIT_BRANCH}
                                git add values.yaml
                                git commit -m "Update image tag to ${IMAGE_TAG}"
                                CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
                                git push origin ${CURRENT_BRANCH}
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Image ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} pushed to ECR and Helm values updated."
        }
        failure {
            echo "Pipeline failed. Please check the logs for details."
        }
        cleanup {
            cleanWs()
        }
    }
}
```

##### Note:
- The `writeFile/writeYaml` and `readFile/readYaml` requires the Jenkins plugin `Pipeline Utility Steps`.
- The ` docker.withRegistry("${ECR_REGISTRY}", "ecr:${AWS_REGION}:${AWS_CREDENTIALS}") {` require the Jenkins `Amazon ECR plugin` plugin
- The `docker.build` and `docker.push` instruction required the `Docker Pipeline` Jenking plugin

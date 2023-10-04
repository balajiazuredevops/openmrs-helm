pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = "balajikolle/openmrs-core"
        DOCKER_TAG = "1.0"
        // DOCKER_REGISTRY_CREDENTIALS = credentials('dockercredentials')
        DOCKER_HUB_USERNAME = "balajikolle"
        DOCKER_HUB_PASSWORD = credentials('docker-passwd')
        NEW_VERSION = "$BUILD_NUMBER"
        CHART_FILE =  "${WORKSPACE}/open-mrs-core/Chart.yaml"
        // AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY')
        // AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = 'us-east-1' // Update with your desired region
        EKS_CLUSTER_NAME = 'eks-cluster'
        HELM_CHART_NAME = 'open-mrs-core'
        //NAMESPACE = 'develop'
        MYSQL_PASSWORD = "openmrs"
        MYSQL_ROOT_PASSWORD = "openmrs"
        OMRS_ADMIN_USER_PASSWORD = "Admin123" 
    }
    parameters {
        choice(
            choices: 'develop\nreview\npre-prod\nproduction',
            description: 'Select the target namespace',
            name: 'TARGET_NAMESPACE'
        )
        string(
            defaultValue: 'open-mrs-core',
            description: 'Name of the Helm chart',
            name: 'HELM_CHART_NAME'
        )
        // Add more dynamic parameters as needed
    }
    
    stages {
        // stage('Checkout') {
        //     steps {
        //         git branch: 'master',
        //         credentialsId: 'git_credt',
        //         url: 'https://github.com/balajiazuredevops/openmrs-core.git'
        //     }
        // }
        
        // stage('Build Docker Image') {
        //     steps {
        //         script {
        //             //def buildNumber = ${BUILD_NUMBER}
        //             // Build the Docker image
        //             sh "docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} ."
        //         }
        //     }
        // }
        // stage('Push to Docker Hub') {
        //     steps {
        //         sh "docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_PASSWORD}"
        //         sh "docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"

        //         // script {
        //         //     //def buildNumber = ${BUILD_NUMBER}
        //         //     // def registryCredentials = ${DOCKER_REGISTRY_CREDENTIALS}
        //         //     // def dockerRegistryUrl = env.DOCKER_REGISTRY_URL
        //         //     //def dockerImageTag = "${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
        //         //     docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_PASSWORD} 
        //         //     docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}
        //         //     // docker.withRegistry('', DOCKER_REGISTRY_CREDENTIALS) {
        //         //     //     sh "docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} "
                        
        //         //     // }
        //         // }
        //     }   
        // }
        stage('Checkout helm template') {
            steps {
                git branch: 'main',
                credentialsId: 'git_credt',
                url: 'https://github.com/balajiazuredevops/openmrs-helm.git'
            }
        }
        stage('Setup') {
            steps {
                script {
                    sh "aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${EKS_CLUSTER_NAME}"
                }
            }
        }
        stage('Replace App Version') {
            steps {
                script {
                    def chartContent = readFile(file: "${CHART_FILE}")
                    def updatedChartContent = chartContent.replaceAll(/appVersion:\s+.*/, "appVersion: ${NEW_VERSION}")
                    writeFile(file: "${CHART_FILE}", text: updatedChartContent)
                    echo "App Version in Chart.yaml has been updated to ${NEW_VERSION}"
                }
            }
        }
        stage('Create or Use Existing Namespace') {
            steps {
                script {
                    def helmChart = params.HELM_CHART_NAME
                    def targetNamespace = params.TARGET_NAMESPACE
                    def namespaceExists = sh(
                        script: "/var/lib/jenkins/bin/kubectl get namespace $targetNamespace",
                        returnStatus: true
                    )
                    //def namespaceExists = sh(script: "/var/lib/jenkins/bin/kubectl get namespace $targetNamespace --ignore-not-found", returnStatus: true)
                    if (namespaceExists != 0) {
                        echo "Namespace $targetNamespace does not exist. Creating..."
                        sh "/var/lib/jenkins/bin/kubectl  version --client=false"
                        sh "/var/lib/jenkins/bin/kubectl create namespace $targetNamespace"
                    } else {
                        echo "Using existing namespace $targetNamespace"
                    }
                }
            }
        }
        stage('Deploy Helm Chart') {
            steps {
                script {
                    def helmChart = params.HELM_CHART_NAME
                    def targetNamespace = params.TARGET_NAMESPACE
                    
                    echo "Deploying ${helmChart} to ${targetNamespace}"
                    //sh "cd ${WORKSPACE}/open-mrs-core"
                    sh "helm package open-mrs-core --version=$BUILD_NUMBER"
                    sh "helm repo index --url https://raw.githubusercontent.com/balajiazuredevops/openmrs-helm/main --merge index.yaml ."
                    //sh "helm repo index --url https://raw.githubusercontent.com/balajiazuredevops/openmrs-helm/main "
                    sh "helm repo add --username balajiazuredevops --password thimmaiah@1998 open-mrs-core https://raw.githubusercontent.com/balajiazuredevops/openmrs-helm/main"
                    sh "helm --debug upgrade --install ${helmChart} open-mrs-core/open-mrs-core --reuse-values --version 0.1.0 --set openmrs.imagename=${DOCKER_IMAGE_NAME} --set openmrs.imagetag=${DOCKER_TAG} --set mysql.MYSQL_PASSWORD=${MYSQL_PASSWORD} --set mysql.MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} --set mysql.OMRS_ADMIN_USER_PASSWORD=${OMRS_ADMIN_USER_PASSWORD} --namespace ${targetNamespace}"
                }
            }
        }  
    }
    post {
        // Clean after build
        always {
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: true,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                               [pattern: '.propsfile', type: 'EXCLUDE']])
        }
    }
    
    // post {
    //     always {
    //         // Clean up by logging out from Docker Hub
    //         sh "docker logout"
    //         sh "aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${EKS_CLUSTER_NAME} --unset"
    //     }
    // }
}

For creating the infra by using terraform:

pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'git_credt',
                url: 'https://github.com/balajiazuredevops/terraformforeks.git'
            }
        }
        stage('Bild EKS'){
            steps {

                sh '''#!/usr/bin/env bash
                    cd ${WORKSPACE}/Terraform/
                    mkdir outplans
                    terraform init 
                    terraform validate
                    terraform plan  -out ${JOB_NAME}-${BUILD_NUMBER}
                    terraform show -no-color ${JOB_NAME}-${BUILD_NUMBER} > outplans/${JOB_NAME}-${BUILD_NUMBER}.txt
                    terraform apply -auto-approve ${JOB_NAME}-${BUILD_NUMBER} 
                    #-var access_key='${AWS_ACCESS_KEY_ID}' -var secret_key '{AWS_SECRET_ACCESS_KEY}' ${JOB_NAME}-${BUILD_NUMBER}
                '''

            }
        }
        // stage(destroy){
        //     steps{
        //         sh 'cd ${WORKSPACE}/Terraform/'
        //         sh 'terraform destroy'
        //     }

        // }
        // stage('Initialize') {
        //     steps {
        //         dir(Terraform){
        //             sh 'terraform init'
        //         }
        //     }
        // }
        // stage('Plan') {
        //     steps {
        //         dir(Terraform){
        //             sh 'terraform plan -out '
        //         }
        //     }
        // }
        // stage('Apply') {
        //     steps {
        //         withCredentials([[
        //         $class: 'AmazonWebServicesCredentialsBinding',
        //         credentialsId: "eksuser",
        //         accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        //         secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        //         ]]) {

        //         }
        //         dir(Terraform){
        //             sh 'terraform apply -auto-approve -var='
        //         }
        //     }
        // }
    }
    // post {
    //     always {
    //         archiveArtifacts artifacts: '${WORKSPACE}/Terraform/outplans/${JOB_NAME}-${BUILD_NUMBER}.txt'
    //     }
    // }
}


For the deploying the application into the eks cluster:

pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = "balajikolle/openmrs-core"
        DOCKER_REGISTRY_CREDENTIALS = credentials('dockercredentials')
       // NEW_VERSION = "env.BUILD_NUMBER"
        CHART_FILE =  "${WORKSPACE}/OpenmrsCore/Chart.yaml"
        // AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY')
        // AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = 'us-east-1' // Update with your desired region
        EKS_CLUSTER_NAME = 'eks-cluster'
        HELM_CHART_NAME = 'openmrs-core'
        NAMESPACE = 'develop'
        MYSQL_PASSWORD = "openmrs"
        MYSQL_ROOT_PASSWORD = "openmrs"
        OMRS_ADMIN_USER_PASSWORD = "Admin123" 
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                credentialsId: 'git_creds',
                url: 'https://github.com/balajiazuredevops/openmrs-core.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                   // def buildNumber = env.BUILD_NUMBER
                    // Build the Docker image
                    // sh "docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} ."
                    sh "docker build -t "${DOCKER_IMAGE_NAME}":$"{BUILD_NUMBER}" ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                   // def buildNumber = env.BUILD_NUMBER
                    // def registryCredentials = env.DOCKER_REGISTRY_CREDENTIALS
                    // def dockerRegistryUrl = env.DOCKER_REGISTRY_URL
                    def dockerImageTag = "${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"

                    docker.withRegistry('', ${DOCKER_REGISTRY_CREDENTIALS}) {
                        docker.image(dockerImageTag).push()
                    }
                }
            }   
        }
        stage('Checkout helm template') {
            steps {
                git branch: 'main',
                credentialsId: 'git_creds',
                url: 'https://github.com/balajiazuredevops/terraformforeks.git'
            }
        }
        stage('Setup') {
            steps {
                script {
                    sh "aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${EKS_CLUSTER_NAME}"
                }
            }
        }
        stage('Replace App Version') {
            steps {
                script {
                    def chartContent = readFile(file: "${CHART_FILE}")
                    def updatedChartContent = chartContent.replaceAll(/appVersion:\s+.*/, "appVersion: ${NEW_VERSION}")
                    writeFile(file: "${CHART_FILE}", text: updatedChartContent)
                    echo "App Version in Chart.yaml has been updated to ${NEW_VERSION}"
                }
            }
        }
        stage('Create or Use Existing Namespace') {
            steps {
                script {
                    def namespaceExists = sh(script: "kubectl get namespace ${NAMESPACE} --ignore-not-found", returnStatus: true)
                    if (namespaceExists != 0) {
                        echo "Namespace ${NAMESPACE} does not exist. Creating..."
                        sh "kubectl create namespace ${NAMESPACE}"
                    } else {
                        echo "Using existing namespace ${NAMESPACE}"
                    }
                }
            }
        }
        stage('Deploy Helm Chart') {
            steps {
                script {
                    sh "cd ${WORKSPACE}/openmrs/OpenmrsCore"
                    sh "helm upgrade --install ${HELM_CHART_NAME} ${WORKSPACE}/openmrs/openmrs-core --set openmrs.imagename=${DOCKER_IMAGE_NAME}, openmrs.imagetag=${BUILD_NUMBER},mysql.MYSQL_PASSWORD=${MYSQL_PASSWORD},mysql.MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD},mysql.OMRS_ADMIN_USER_PASSWORD=${OMRS_ADMIN_USER_PASSWORD} --namespace ${NAMESPACE}"
                }
            }
        }  
    }
    
    // post {
    //     always {
    //         // Clean up by logging out from Docker Hub
    //         sh "docker logout"
    //         sh "aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${EKS_CLUSTER_NAME} --unset"
    //     }
    // }
}
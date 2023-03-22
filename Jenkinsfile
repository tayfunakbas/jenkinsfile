pipeline {
    agent any
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Get Source Code From GitLab') {
            when { expression { params.GIT_PROJECT_URL } }
            steps {
                sh '''#!/bin/bash -e
                if [[ "${IMAGE_VERSION}" = "" ]]; then
                  echo "IMAGE_VERSION parameter should be set!"
                  exit 99
                fi
                echo STEP: checkout git scm. Source Code URL: ${GIT_PROJECT_URL}, Source Code Version: ${IMAGE_VERSION} 
                '''
                checkout scm: ([
                    $class: 'GitSCM',  
                    userRemoteConfigs:  [[credentialsId:'gitlab_ee_credential', url: '${GIT_PROJECT_URL}' ]],
                    branches: [[name: '${IMAGE_VERSION}']]
                ]
                )
            }
        }
        //stage('debug openshift'){
        //    steps {
        //    sh '''#!/bin/bash -e
        //    oc login -u ${OCP_USER_CREDENTIAL_USR} -p ${OCP_USER_CREDENTIAL_PSW} https://api.${OCP_REGION_IST}.paas.turktelekom.intra:6443 --insecure-skip-tls-verify
        //    oc project smarttbill-fixed-dev
        //    oc get all -o yaml
        //    '''
        //    }
        //}

        stage('Clone Helm Chart Files From GitLab'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'gitlab_ee_credential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''#!/bin/bash -e
                        git clone https://${USERNAME}:${PASSWORD}@${GIT_HELM_CHART_PROJECT_URL}
                    '''
                }
            }
        }

        stage('Clone Helm Values From GitLab'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'gitlab_ee_credential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''#!/bin/bash -e
                        git clone https://${USERNAME}:${PASSWORD}@${GIT_HELM_VALUES_PROJECT_URL}
                    '''
                }
            }
        }

        stage('Sonarqube Static Analysis') {
            when { allOf {
                expression { params.SONARQUBE_ENABLED }
                expression { params.GIT_PROJECT_URL }
            } }
            environment {
                scannerHome = tool 'sonarqube_scanner'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                sh '''${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=${SONAR_PROJECT_NAME} \
                    -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                    -Dsonar.projectVersion=${IMAGE_VERSION} \
                    -Dsonar.sourceEncoding=UTF-8 \
                    -Dsonar.sources=$WORKSPACE \
                    -Dsonar.java.binaries=$WORKSPACE \
                -X
                '''
                }
            timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Stage'){
            when { expression { params.GIT_PROJECT_URL } }
            steps {
            sh '''#!/bin/bash -e

            #echo "${JAVA_HOME}"
            #echo "${PATH}"

            echo "Building ${IMAGE_NAME}:${IMAGE_VERSION} container"
            
            echo "mvn build"
            mvn -version
            mvn -B -DskipTests clean

            echo "podman image build step"
            podman image build . -t ${IMAGE_NAME}:${IMAGE_VERSION} 
            echo "${IMAGE_NAME}:${IMAGE_VERSION} version build finished"
            '''
            }
        }

        stage('Push to Gitlab CR'){
            when { expression { params.GIT_PROJECT_URL } }
            steps {
            sh '''#!/bin/bash -e
            podman login -u ${IMAGE_REGISTRY_USERNAME} -p ${IMAGE_REGISTRY_PWD} --tls-verify=false ${IMAGE_REGISTRY_URL}
            # Image push while tagging
            podman push ${IMAGE_NAME}:${IMAGE_VERSION} docker://${IMAGE_REGISTRY_URL}/${IMAGE_REGISTRY_PATH}/${IMAGE_NAME}:${IMAGE_VERSION} #--log-level=debug
            podman push ${IMAGE_NAME}:${IMAGE_VERSION} docker://${IMAGE_REGISTRY_URL}/${IMAGE_REGISTRY_PATH}/${IMAGE_NAME}:latest #--log-level=debug
            '''
            }
        }

        stage('Deploy to OpenShift Istanbul'){
            when { expression { params.OCP_REGION_IST } }
            steps {
            sh '''#!/bin/bash -e
            oc login -u ${OCP_USER_CREDENTIAL_USR} -p ${OCP_USER_CREDENTIAL_PSW} https://api.${OCP_REGION_IST}.paas.turktelekom.intra:6443 --insecure-skip-tls-verify
            oc project ${OCP_NAMESPACE}

            echo "Deploying the ${HELM_RELEASE_NAME} chart to ${OCP_REGION_IST} on ${OCP_NAMESPACE} project..."
            /usr/local/bin/helm upgrade --install -f ${SERVICE_NAME}/${OCP_REGION_IST}-${ENV_NAME}.yaml ${HELM_RELEASE_NAME} \
            --set image.repository=${IMAGE_REGISTRY_URL}/${IMAGE_REGISTRY_PATH}/${IMAGE_NAME} \
            --set image.tag=${IMAGE_VERSION} \
            --set imageCredentials.registry=${IMAGE_REGISTRY_URL} \
            --set imageCredentials.username=${IMAGE_REGISTRY_USERNAME} \
            --set imageCredentials.password=${IMAGE_REGISTRY_PWD} \
            --set fullnameOverride="${PROJECT_NAME}-${SERVICE_NAME}-${ENV_NAME}" \
            --namespace ${OCP_NAMESPACE} \
            ./tt-common --debug
            '''
            }
        }
        stage('Deploy to OpenShift Ankara'){
            when { expression { params.OCP_REGION_ANK } }
            steps {
            sh '''#!/bin/bash -e
            oc login -u ${OCP_USER_CREDENTIAL_USR} -p ${OCP_USER_CREDENTIAL_PSW} https://api.${OCP_REGION_ANK}.paas.turktelekom.intra:6443 --insecure-skip-tls-verify
            oc project ${OCP_NAMESPACE}
            
            echo "Deploying the ${HELM_RELEASE_NAME} chart to ${OCP_REGION_ANK} on ${OCP_NAMESPACE} project..."
            /usr/local/bin/helm upgrade --install -f ${SERVICE_NAME}/${OCP_REGION_ANK}-${ENV_NAME}.yaml ${HELM_RELEASE_NAME} \
            --set image.repository=${IMAGE_REGISTRY_URL}/${IMAGE_REGISTRY_PATH}/${IMAGE_NAME} \
            --set image.tag=${IMAGE_VERSION} \
            --set imageCredentials.registry=${IMAGE_REGISTRY_URL} \
            --set imageCredentials.username=${IMAGE_REGISTRY_USERNAME} \
            --set imageCredentials.password=${IMAGE_REGISTRY_PWD} \
            --set fullnameOverride="${PROJECT_NAME}-${SERVICE_NAME}-${ENV_NAME}" \
            --namespace ${OCP_NAMESPACE} \
            ./tt-common --debug
            '''
            }
        }  


    }
}

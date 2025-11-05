/* Generated from templates/source-repo/Jenkinsfile.njk. Do not edit directly. */
pipeline {
    agent {
        kubernetes {
            yaml """
              apiVersion: v1
              kind: Pod
              spec:
                containers:
                - name: 'runner'
                  image: 'quay.io/redhat-appstudio/rhtap-task-runner:latest'
                  securityContext:
                    privileged: true
            """
        }
        
    }
    environment {
        HOME = "${WORKSPACE}"
        DOCKER_CONFIG = "${WORKSPACE}/.docker"
        ROX_API_TOKEN = credentials('ROX_API_TOKEN')
        ROX_CENTRAL_ENDPOINT = credentials('ROX_CENTRAL_ENDPOINT')
        GITOPS_AUTH_PASSWORD = credentials('GITOPS_AUTH_PASSWORD')
        GITOPS_AUTH_USERNAME = credentials('GITOPS_AUTH_USERNAME')
        QUAY_IO_CREDS = credentials('QUAY_IO_CREDS')
        COSIGN_SECRET_PASSWORD = credentials('COSIGN_SECRET_PASSWORD')
        COSIGN_SECRET_KEY = credentials('COSIGN_SECRET_KEY')
        COSIGN_PUBLIC_KEY = credentials('COSIGN_PUBLIC_KEY')
        QUAY_IO_CREDS = credentials('QUAY_IO_CREDS')
        IMAGE_REGISTRY_USER = credentials('IMAGE_REGISTRY_USER')
        IMAGE_REGISTRY_PASSWORD = credentials('IMAGE_REGISTRY_PASSWORD')
        TUF_MIRROR = credentials('TUF_MIRROR')
        REKOR_HOST = credentials('REKOR_HOST')
        TRUSTIFICATION_BOMBASTIC_API_URL = credentials('TRUSTIFICATION_BOMBASTIC_API_URL')
        TRUSTIFICATION_OIDC_ISSUER_URL = credentials('TRUSTIFICATION_OIDC_ISSUER_URL')
        TRUSTIFICATION_OIDC_CLIENT_ID = credentials('TRUSTIFICATION_OIDC_CLIENT_ID')
        TRUSTIFICATION_OIDC_CLIENT_SECRET = credentials('TRUSTIFICATION_OIDC_CLIENT_SECRET')
        TRUSTIFICATION_SUPPORTED_CYCLONEDX_VERSION = credentials('TRUSTIFICATION_SUPPORTED_CYCLONEDX_VERSION')
    }
    stages {
        stage('init') {
            steps {
                container('runner') {
                    sh '''
                        cp -R /work/* .
                        env
                        echo running init
                        ./rhtap/init.sh
                    '''
                }
            }
        }

        stage('build') {
            steps {
                container('runner') {
                    sh '''
                        git config --global --add safe.directory $WORKSPACE
                        echo "running bildah"
                        ./rhtap/buildah-rhtap.sh
                        echo "running attestation"
                        ./rhtap/cosign-sign-attest.sh
                    '''
                }
            }
        }

        stage('deploy') {
            steps {
                container('runner') {
                    sh '''
                        echo "updating deployment"
                        ./rhtap/update-deployment.sh
                    '''
                }
            }
        }

        stage('scan') {
            steps {
                container('runner') {
                    sh '''
                        echo "checking deployment"
                        ./rhtap/acs-deploy-check.sh
                        echo "checking image"
                        ./rhtap/acs-image-check.sh
                        echo "scanning image"
                        ./rhtap/acs-image-scan.sh
                    '''
                }
            }
        }

        stage('summary') {
            steps {
                container('runner') {
                    sh '''
                        echo "showing sbom"
                        ./rhtap/show-sbom-rhdh.sh
                        echo "Summarizing"
                        ./rhtap/summary.sh
                    '''
                }
            }
        }
    }
}

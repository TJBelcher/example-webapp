def builderImage
def productionImage
def ACCOUNT_REGISTRY_PREFIX
def GIT_COMMIT_HASH

pipeline {
    agent any
    stages {
        stage('Checkout Source Code and Logging Into Registry') {
            steps {
                echo 'Logging Into the Private ECR Registry'
                script {
                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
                    ACCOUNT_REGISTRY_PREFIX = "206799381603.dkr.ecr.us-east-1.amazonaws.com"
                    sh """
                    \$(aws ecr get-login --no-include-email --region us-east-1)
                    """
                }
            }
        }

        stage('Make A Builder Image') {
            steps {
                echo 'Starting to build the project builder docker image'
                script {
                    builderImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp-builder:${GIT_COMMIT_HASH}", "-f ./Dockerfile.builder .")
                    builderImage.push()
                    builderImage.push("${env.GIT_BRANCH}")
                    builderImage.inside('-v $WORKSPACE:/output -u root') {
                        sh """
                           cd /output
                           lein uberjar
                        """
                    }
                }
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'running unit tests in the builder image.'
                script {
                    builderImage.inside('-v $WORKSPACE:/output -u root') {
                    sh """
                       cd /output
                       lein test
                    """
                    }
                }
            }
        }

        stage('Build Production Image') {
            steps {
                echo 'Starting to build docker image'
                script {
                    productionImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp:${GIT_COMMIT_HASH}")
                    productionImage.push()
                    productionImage.push("${env.GIT_BRANCH}")
                }
            }
        }


        stage('Deploy to Production fixed server') {
            when {
                branch 'release'
            }
            steps {
                echo 'Deploying release to production'
                script {
                    productionImage.push("deploy")
                    sh """
                       aws ec2 reboot-instances --region us-east-1 --instance-ids i-05693134e83b1af02
                    """
                }
            }
        }

        stage('Integration Tests') {
            when {
                   branch 'master'
            }
            steps {
                echo 'Deploy to test environment and run integration tests'
                script {
                    TEST_ALB_LISTENER_ARN="arn:aws:elasticloadbalancing:us-east-1:206799381603:listener/app/testing-website/df52c225707943f6/1cf82c20d9cbb36b"
                    sh """
                        ls -l
                        chmod +x run-stack.sh
                        ls -l
                        ./run-stack.sh example-webapp-test ${TEST_ALB_LISTENER_ARN}
                    """
                }
                echo 'Running tests on the integration test environment'
                script {
                    sh """
                       curl -v http://testing-website-956734829.us-east-1.elb.amazonaws.com | grep '<title>Welcome to example-webapp</title>'
                       if [ \$? -eq 0 ]
                       then
                           echo tests pass
                       else
                           echo tests failed
                           exit 1
                       fi
                    """
                }
            }
        }


        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                script {
                    PRODUCTION_ALB_LISTENER_ARN="arn:aws:elasticloadbalancing:us-east-1:206799381603:listener/app/production-website/b2724a4c38a6b318/272f9d3ca73ac765"
                    sh """
                        ls -l
                        chmod +x run-stack.sh
                        ls -l
                        ./run-stack.sh example-webapp-production ${PRODUCTION_ALB_LISTENER_ARN}
                    """
                }
            }
        }
    }
}


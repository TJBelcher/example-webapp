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


pipeline {
    agent {
        label 'linux'
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        NODE_VERSION = '18.17'
    }

    stages {
        stage('Pre-commit Checks') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'jenkins') {
                        echo 'Running on master branch'

                        sh './check_license.sh'

                        sh """
                          NODE_VERSION=\$(cat .nvmrc) && NODE_VERSION=\${NODE_VERSION:-${env.NODE_VERSION}}
                          docker build -t yunikorn/yunikorn-website:2.0.0 . --build-arg NODE_VERSION=\${NODE_VERSION}
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pre-commit Checks completed'
        }
        failure{
            echo 'Build failed'
        }
    }
}

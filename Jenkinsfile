pipeline {
    agent none // 不在默认环境中运行

    stages {
        stage('Pre-commit Checks') {
            agent {
                kubernetes {
                    // 定义 Docker 客户端和 Git 容器
                    containerTemplate(
                        name: 'docker-git',
                        image: 'docker:19.03-git',
                        command: 'cat',
                        ttyEnabled: true
                    )
                }
            }
            steps {
                // 使用 Docker 客户端和 Git 容器
                container('docker-git') {
                    script {
                        if (env.BRANCH_NAME == 'jenkins') {
                            // 设置环境变量，指向 Docker 守护进程
                            sh "export DOCKER_HOST=tcp://localhost:2376"
                            // 设置 Node 版本
                            sh "NODE_VERSION=\$(cat .nvmrc) && NODE_VERSION=\${NODE_VERSION:-${env.NODE_VERSION}}"
                            // 记录提交信息
                            // sh "git log master --pretty=format:'Auto refresh: %s' -n 1 > ../asf-site-commit.txt"
                            // 构建并运行 Docker 容器
                            sh "docker build -t yunikorn/yunikorn-website:2.0.0 . --build-arg NODE_VERSION=\${NODE_VERSION}"
                            sh "docker run --name yunikorn-site -d -p 3000:3000 yunikorn/yunikorn-website:2.0.0"
                            // 等待网站响应
                            sh "bash -c 'while true; do curl -Is http://localhost:3000 | head -n 1 | grep \"OK\"; [ \$? -eq 0 ] && break; sleep 3; done'"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pre-commit Checks completed'
        }
        failure {
            echo 'Build failed'
        }
    }
}

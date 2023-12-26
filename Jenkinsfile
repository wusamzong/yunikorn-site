pipeline {
    agent none // 不在默认环境中运行

    stages {
        stage('Pre-commit Checks') {
            agent {
                kubernetes {
                yaml """
apiVersion: v1
kind: Pod
metadata:
  annotations:
    buildUrl: "http://jenkins.devops-tools.svc.cluster.local:8080/job/YuniKorn%20Site/job/jenkins/7/"
    runUrl: "job/YuniKorn%20Site/job/jenkins/7/"
  labels:
    jenkins/jenkins-jenkins-agent: "true"
    jenkins/label-digest: "270a7ec06b2ae63d7cad75f5aa0514cb812c57ca"
    jenkins/label: "YuniKorn_Site_jenkins_7-9t9wh"
  name: "yunikorn-site-jenkins-7-9t9wh-cfscl-6vsd5"
  namespace: "devops-tools"
spec:
  containers:
  - name: dind
    image: docker:dind
    securityContext:
      privileged: true
    command:
    - dockerd-entrypoint.sh
    args:
    - --host=unix:///var/run/docker.sock
    - --host=tcp://0.0.0.0:2376
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - name: docker-git
    image: docker:19.03-git
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_TUNNEL"
      value: "jenkins-agent.devops-tools.svc.cluster.local:50000"
    - name: "JENKINS_AGENT_NAME"
      value: "yunikorn-site-jenkins-7-9t9wh-cfscl-6vsd5"
    - name: "JENKINS_NAME"
      value: "yunikorn-site-jenkins-7-9t9wh-cfscl-6vsd5"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.devops-tools.svc.cluster.local:8080/"
    image: "jenkins/inbound-agent:3142.vcfca_0cd92128-1"
    name: "jnlp"
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - emptyDir:
      medium: ""
    name: "workspace-volume"
"""
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

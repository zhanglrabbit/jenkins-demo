node('haimaxy-jnlp') {
    stage('Prepare') {
        echo "1.Prepare Stage"
   	environment { 
            def http_proxy="http://8.210.101.255:8000"
        }
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
	echo "${build_tag}"
	echo "${env.BRANCH_NAME}"
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t registry.cn-shanghai.aliyuncs.com/xiniao/k8s/jenkins-demo:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'aliyunharbor', passwordVariable: 'aliyunharborPassword', usernameVariable: 'aliyunharborUser')]) {
            sh "docker login registry.cn-shanghai.aliyuncs.com -u ${aliyunharborUser} -p ${aliyunharborPassword}"
            sh "docker push registry.cn-shanghai.aliyuncs.com/xiniao/k8s/jenkins-demo:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        sh "kubectl apply -f k8s.yaml --record"
    }
}

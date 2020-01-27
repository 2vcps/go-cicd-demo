podTemplate(yaml: """
kind: Pod
serviceAccountName: jenkins-k8s
spec:
  containers:
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug-539ddefcae3fd6b411a95982a830d987f4214251
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    env:
    - name: DOCKER_CONFIG
      value: /kaniko/.docker/
    volumeMounts:
      - name: harbor-config
        mountPath: /kaniko/.docker
  volumes:
    - name: harbor-config
      configMap:
        name: harbor-config
"""
) {

node(POD_LABEL) {
  def myRepo = checkout scm
  def gitCommit = myRepo.GIT_COMMIT
  def gitBranch = myRepo.GIT_BRANCH
  stages {
    stage('Build and push webapp image with Container Builder') {
      steps {
        container('kaniko') {
        git 'https://github.com/2vcps/go-cicd-demo.git'
        container(name: 'kaniko') {
            sh '''
            /kaniko/executor --skip-tls-verify --dockerfile `pwd`/gowebapp/Dockerfile --context `pwd`/gowebapp --destination=harbor.newstack.local/jowings/gowebapp:latest --destination=harbor.newstack.local/jowings/gowebapp:v$BUILD_NUMBER
            '''
      }
    }
      }
    }
    stage('Build and push webapp sql image with Container Builder') {
      steps {
        container('kaniko') {
        git 'https://github.com/2vcps/go-cicd-demo.git'
        container(name: 'kaniko') {
            sh '''
            /kaniko/executor --single-snapshot --skip-tls-verify --dockerfile `pwd`/gowebapp-mysql/Dockerfile --context `pwd`/gowebapp-mysql --destination=harbor.newstack.local/jowings/gowebapp-mysql:latest --destination=harbor.newstack.local/jowings/gowebapp-mysql:v$BUILD_NUMBER
            '''
      }
    }
      }
    }
    stage('Deploy with kubectl') {
      steps{
      container('kubectl') {
        sh "kubectl get ns go-demo-${BRANCH_NAME} || kubectl create ns ns go-demo-${BRANCH_NAME}"
        sh "kubectl -n ns go-demo-${BRANCH_NAME} apply -f deployment.yaml"
        sh "kubectl -n ns go-demo-${BRANCH_NAME} get pod"
      }
      }
    }
    // steps('Deploy Canary') {
    //   // Canary branch
    //   when { branch 'canary' }
    //   steps {
    //     container('kubectl') {
    //         sh('kubectl get pod')
    //       // Change deployed image in canary to the one we just built
    //     //   sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/canary/*.yaml")
    //     //   step([$class: 'KubernetesEngineBuilder',namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
    //     //   step([$class: 'KubernetesEngineBuilder',namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/canary', credentialsId: env.JENKINS_CRED, verifyDeployments: true])
    //     //   sh("echo http://`kubectl --namespace=production get service/${FE_SVC_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${FE_SVC_NAME}")
    //     }
    //   }
    // }
    // steps('Deploy Production') {
    //   // Production branch
    //   when { branch 'master' }
    //   step{
    //     container('kubectl') {
    //         sh('kubectl get pod')
    //     // Change deployed image in canary to the one we just built
    //     //   sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/production/*.yaml")
    //     //   step([$class: 'KubernetesEngineBuilder',namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
    //     //   step([$class: 'KubernetesEngineBuilder',namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/production', credentialsId: env.JENKINS_CRED, verifyDeployments: true])
    //     //   sh("echo http://`kubectl --namespace=production get service/${FE_SVC_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${FE_SVC_NAME}")
    //     }
    //   }
    // }
  //   steps('Deploy Dev') {
  //     // Developer Branches
  //     when {
  //       not { branch 'master' }
  //       not { branch 'canary' }
  //     }
  //     step {
  //       container('kubectl') {
  //           sh('kubectl get pod')
  //         // Create namespace if it doesn't exist
  //       //   sh("kubectl get ns ${env.BRANCH_NAME} || kubectl create ns ${env.BRANCH_NAME}")
  //       //   // Don't use public load balancing for development branches
  //       //   sh("sed -i.bak 's#LoadBalancer#ClusterIP#' ./k8s/services/frontend.yaml")
  //       //   sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/dev/*.yaml")
  //       //   step([$class: 'KubernetesEngineBuilder',namespace: "${env.BRANCH_NAME}", projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
  //       //   step([$class: 'KubernetesEngineBuilder',namespace: "${env.BRANCH_NAME}", projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/dev', credentialsId: env.JENKINS_CRED, verifyDeployments: true])
  //       //   echo 'To access your environment run `kubectl proxy`'
  //       //   echo "Then access your service via http://localhost:8001/api/v1/proxy/namespaces/${env.BRANCH_NAME}/services/${FE_SVC_NAME}:80/"
  //       }
  //     }
  //   }
   }
  }
}
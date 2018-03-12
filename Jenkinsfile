def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: 'label', containers: [
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]) {
    node('label') {

        stage('do some Docker work') {
            container('docker') {

                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'lpmxm-acr',
                        usernameVariable: 'ACR_USER',
                        passwordVariable: 'ACR_PASSWORD']]) {
                    withEnv(["ACR_SERVER=${env.ACR_LOGINSERVER}"]) {
                        sh """
                            printenv
                            docker pull ubuntu
                            docker tag ubuntu ${env.ACR_SERVER}/ubuntu:${env.BUILD_NUMBER}
                            """
                        printenv
                        sh "docker login ${env.ACR_SERVER} -u ${env.ACR_USER} -p ${env.ACR_PASSWORD}"
                        sh "docker push ${env.ACR_SERVER}/ubuntu:${env.BUILD_NUMBER}"
                    } //end withEnv
                } //end withCredentials
            }
        } //end stage

        stage('do some kubectl work') {
            container('kubectl') {

                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'lpmxm-acr',
                        usernameVariable: 'ACR_USER',
                        passwordVariable: 'ACR_PASSWORD']]) {

                    sh "kubectl get nodes"
                }
            }
        }
        stage('do some helm work') {
            container('helm') {

               sh "helm ls"
            }
        }
    }
}
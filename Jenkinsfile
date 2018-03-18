def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(
    label: 'label',
    containers: [
        containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true),
    ],
    envVars: [
        envVar(key: 'ACR_LOGINSERVER', value: 'devc3acr01.azurecr.io')
    ],
    volumes: [
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    ]) {
    node('label') {

        stage('Misc. Docker Work') {
            container('docker') {
                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'devc3acr01-acr',
                        usernameVariable: 'ACR_USER',
                        passwordVariable: 'ACR_PASSWORD']]) {
                    sh "docker pull ubuntu"
                    /*
                    This nonsense is necessary because withEnv doesn't seem to be working.
                    I couldn't figure out a way to use both the shell variable $ACR_LOGINSERVER
                    and things like ${env.BUILD_NUMBER} so we have to resort to $/ syntax in order
                    to get the environment variables from both the shell e.g. ACR_LOGINSERVER and
                    Jenkins e.g. BUILD_NUMBER
                    */
                    def tagcmd = $/docker tag ubuntu $$ACR_LOGINSERVER/ubuntu:$$BUILD_NUMBER/$
                    sh tagcmd

                    def logincmd = $/docker login $$ACR_LOGINSERVER -u $$ACR_USER -p $$ACR_PASSWORD/$
                    sh logincmd

                    def pushcmd = $/docker push $$ACR_LOGINSERVER/ubuntu:$$BUILD_NUMBER/$
                    sh pushcmd

                } //end withCredentials
            } //end container
        } //end stage

        stage('do some kubectl work') {
            container('kubectl') {

                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'devc3acr01-acr',
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
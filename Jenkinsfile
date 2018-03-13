def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(
    label: 'label',
    containers: [
        containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true),
    ],
    envVars: [
        envVar(key: 'ACR_LOGINSERVER', value: 'lpmxmacr.azurecr.io')
    ],
    volumes: [
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    ]) {
    node('label') {
        stage ('Shell Interpolation') {
            container('docker') {
                echo 'No quotes in single backticks'
                sh 'echo $BUILD_NUMBER'
                echo 'Double quotes are silently dropped'
                sh 'echo "$BUILD_NUMBER"'
                echo 'Even escaped with a single backslash they are dropped'
                sh 'echo \"$BUILD_NUMBER\"'
                echo 'Using two backslashes, the quotes are preserved'
                sh 'echo \\"$BUILD_NUMBER\\"'
                echo 'Using three backslashes still results in preserving the single quotes'
                sh 'echo \\\"$BUILD_NUMBER\\\"'
                echo 'To end up with \" use \\\\\\\" (yes, seven backticks)'
                sh 'echo \\\\\\"$BUILD_NUMBER\\\\\\"'
                echo 'This is fine and all, but we cannot substitute Jenkins variables in single quote strings'
                def foo = 'bar'
                sh 'echo "${foo}"'
                echo 'This does not interpolate the string but instead tries to look up "foo" on the command line, so use double quotes'
                sh "echo \"${foo}\""
                echo 'Great, more escaping is needed now. How about just concatenate the strings? Well that gets kind of ugly'
                sh 'echo \\\\\\"' + foo + '\\\\\\"'
                echo 'We still needed all of that escaping and mixing concatenation is hideous!'
                echo 'There must be a better way, enter dollar slashy strings (actual term)'
                def command = $/echo \\\"${foo}\\\"/$
                sh command
                echo 'String interpolation works out of the box as well as environment variables, escaped with double dollars'
                def vash = $/echo \\\"$$BUILD_NUMBER\\\" ${foo}/$
                sh vash
                echo 'It still requires escaping the escape but that is just bash being bash at that point'
                echo 'Slashy strings are the closest to raw shell input with Jenkins, although the non dollar variant seems to give an error but the dollar slash works fine'
            } //end container
        } //end stage



        stage('Misc. Docker Work') {
            container('docker') {
                environment {
                    ACR_LOGINSERVER = lpmxmacr.azurecr.io
                }
                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'lpmxm-acr',
                        usernameVariable: 'ACR_USER',
                        passwordVariable: 'ACR_PASSWORD']]) {
                    sh """
                        printenv
                        docker pull ubuntu
                        docker tag ubuntu lpmxmacr.azurecr.io/ubuntu:${env.BUILD_NUMBER}
                        """
                    sh 'echo $ACR_LOGINSERVER'
                    sh "echo ${env.ACR_LOGINSERVER}"
                    def cmd = $/echo \\\"$$ACR_LOGINSERVER\\\" \\\"$$BUILD_NUMBER\\\"/$
                    sh cmd
                    sh "docker login ${env.ACR_LOGINSERVER} -u ${env.ACR_USER} -p ${env.ACR_PASSWORD}"
                    sh "docker push ${env.ACR_LOGINSERVER}/ubuntu:${env.BUILD_NUMBER}"
                } //end withCredentials
            } //end container
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
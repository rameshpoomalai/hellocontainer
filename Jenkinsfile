def label = "worker-${UUID.randomUUID().toString()}"
podTemplate(label: label ,
    volumes: [
        hostPathVolume(hostPath: '/etc/docker/certs.d', mountPath: '/etc/docker/certs.d'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        secretVolume(secretName: 'registry-account', mountPath: '/var/run/secrets/registry-account'),
        configMapVolume(configMapName: 'registry-config', mountPath: '/var/run/configs/registry-config')
    ],
    containers: [
        containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm', command: 'cat', ttyEnabled: true)
  ]) {

    node(label) {
        checkout scm
        container('docker') {
            stage('Build Docker Image') {
                sh """
                #!/bin/bash
                NAMESPACE=`cat /var/run/configs/registry-config/namespace`
                REGISTRY=`cat /var/run/configs/registry-config/registry`

                docker build -t \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER} .
                """
            }
            stage('Push Docker Image to Registry') {
                sh """
                #!/bin/bash
                NAMESPACE=`cat /var/run/configs/registry-config/namespace`
                REGISTRY=`cat /var/run/configs/registry-config/registry`

                set +x
                DOCKER_USER=`cat /var/run/secrets/registry-account/username`
                DOCKER_PASSWORD=`cat /var/run/secrets/registry-account/password`
                docker login -u=\${DOCKER_USER} -p=\${DOCKER_PASSWORD} \${REGISTRY}
                set -x

                docker push \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER}
                """
            }
        }

        container('helm') {
            stage('Deploy new helm release') {
                sh """
                #!/bin/bash
                set +e
                NAMESPACE=`cat /var/run/configs/registry-config/namespace`
                REGISTRY=`cat /var/run/configs/registry-config/registry`
                CHARTNAME=`helm list --deployed --short hello-container`

                helm list \${CHARTNAME}

                #if [ \${?} -ne "0" ]; then
                    # No chart release to update
                #    echo "install new application "
                #    helm install hello-container ./hellocontainer-chart/ --set image.repository=\${REGISTRY}/\${NAMESPACE}/hello-container --set image.tag=${env.BUILD_NUMBER}
                #else
                #    echo "Upgrade the existing application "
                #    helm upgrade hello-container ./hellocontainer-chart/ --set image.repository=\${REGISTRY}/\${NAMESPACE}/hello-container --set image.tag=${env.BUILD_NUMBER}

                #fi
                ls
                helm upgrade mydemo hellocontainer-chart --set image.repository=\${REGISTRY}/\${NAMESPACE}/hello-container --set image.tag=${env.BUILD_NUMBER}
                """
            }
        }


    }
}

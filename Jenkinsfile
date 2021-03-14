/*
# for Jenkins Agent Pod
VOLUME_CONFIG_MAP_NAME=
VOLUME_CONFIG_MAP_MOUNT=
VOLUME_PVC_NAME=
VOLUME_PVC_MOUNT=
SERVICE_ACCOUNT=default

# for Build
# - Gradle
GRADLE_USER_HOME=
# - Docker
DOCKER_CRED=DOCKER_CRED
DOCKER_REGISTRY=registry-1.docker.io

# Deploy
# - kubectl
K8S_NAMESPACE=default
KUBECONFIG=/var/secret/kube.conf
# - Skaffold
SKAFFOLD_CMD=build
SKAFFOLD_OPTS=--dry-run

# Pipeline
CMD=skaffold build --dry-run
*/

/*
 * Pod Template Spec
 */
def spec = [
    label: "jenkins-${JOB_BASE_NAME}",
    serviceAccount: env.SERVICE_ACCOUNT ?: "default",
    containers: []
        << containerTemplate(name: 'skaffold', image: 'gcr.io/k8s-skaffold/skaffold:v1.15.0', ttyEnabled: true, command: 'cat', resourceRequestCpu: '1', resourceRequestMemory: '1Gi'),
    volumes: []
        << configMapVolume(mountPath: "${VOLUME_CONFIG_MAP_MOUNT}", configMapName: "${VOLUME_CONFIG_MAP_NAME}")
    << persistentVolumeClaim(mountPath: "${VOLUME_PVC_MOUNT}", claimName: "${VOLUME_PVC_NAME}")
]

/*
 * Main Pipeline
 */
def script = {
    node(spec.label) {
        container('skaffold') {
            
            stage('*environment') {
                // TODO: Upgrade to helm v3
                // - https://github.com/helm/helm/tree/v2.17.0
                // - https://github.com/GoogleContainerTools/skaffold/blob/master/deploy/skaffold/Dockerfile.deps#L24
                sh """
                    HELM_VERSION=v2.16.3
                    curl -s -o helm.tar.gz https://get.helm.sh/helm-\${HELM_VERSION}-linux-amd64.tar.gz
                    tar -xf helm.tar.gz --strip-components 1
                    mv helm /usr/local/bin/
                """
                
                sh """
                    skaffold version
                    helm version
                """
                if (env.DEBUG)
                    sh "env | grep -vE 'GITEA|HARBOR|ZCP' | sort"
            }
            
            stage('*checkout') {
                checkout scm
            }
            
            stage('Skaffold') {
                withCred("${env.DOCKER_CRED ?: 'DOCKER_CRED'}") {
                    sh "docker login -u '$USERNAME' -p '$PASSWORD' ${env.DOCKER_REGISTRY ?: 'registry-1.docker.io'}"
                }
                
                def CMD_DEFAULT = env.CMD ?: "skaffold ${env.SKAFFOLD_CMD ?: 'build'} ${env.SKAFFOLD_OPTS ?: '--dry-run'}"
                sh """
                    # skaffold run -f skaffold-release.yaml --profile develop
                    # skaffold ${env.SKAFFOLD_CMD ?: 'build'} ${env.SKAFFOLD_OPTS ?: '--dry-run'}
                    ${ env.CMD ?: CMD_DEFAULT }
                """
            }
        }
    }
}

/*
 * Execute Pipeline
 */
timestamps {
    podTemplate spec, script
}

/*
 * Functions
 * - https://www.jenkins.io/doc/pipeline/steps/credentials-binding/
 */
def withCred(id, block) {
    def args = [credentialsId: id, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']
    withCredentials([usernamePassword(args)], block)
}

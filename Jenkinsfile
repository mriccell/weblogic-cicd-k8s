pipeline {
    options {
        disableConcurrentBuilds()
    }
    agent any
    tools {
        jdk 'jdk11'
    }

    environment {
        WLSIMG_BLDDIR = "${env.WORKSPACE}/resources/build"
        WLSIMG_CACHEDIR = "${env.WORKSPACE}/resources/cache"
        IMAGE_TAG = "phx.ocir.io/weblogick8s/onprem-domain-image:1"
    }

    stages {
        stage ('Environment') {
            steps {
                sh '''
                    mkdir -p  ${WLSIMG_BLDDIR} ${WLSIMG_CACHE_DIR}
                    echo "IMAGE_TAG = ${IMAGE_TAG}"
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "JAVA_HOME = ${JAVA_HOME}"
                '''
            }
        }
        stage ('Build Image') {
            steps {
                sh '''
                    curl -SLO  https://github.com/oracle/weblogic-image-tool/releases/download/release-1.8.1/imagetool.zip
                    curl -SLO  https://github.com/oracle/weblogic-deploy-tooling/releases/download/release-1.7.1/weblogic-deploy.zip
                    ls *
                    cp weblogic-deploy.zip /scratch/artifacts/imagetool
                    jar xvf imagetool.zip
                    chmod +x imagetool/bin/*
                    rm -rf ${WLSIMG_CACHEDIR}
                    imagetool/bin/imagetool.sh cache addInstaller --type wdt --path /scratch/artifacts/imagetool/weblogic-deploy.zip --version 1.7.1
                    imagetool/bin/imagetool.sh cache addInstaller --type wls --path /scratch/artifacts/imagetool/fmw_12.2.1.4.0_wls_Disk1_1of1.zip --version 12.2.1.4.0
                    imagetool/bin/imagetool.sh cache addInstaller --type jdk --path /scratch/artifacts/imagetool/jdk-8u212-linux-x64.tar.gz --version 8u212
                    imagetool/bin/imagetool.sh create --tag ${IMAGE_TAG} --version 12.2.1.4.0 --jdkVersion=8u212 --wdtArchive=./DiscoveredDemoDomain.zip --wdtModel=./DiscoverDomain-v1.yaml --wdtDomainHome=/u01/oracle/user_projects/domains/onprem-domain --wdtVariables=./domain.properties --wdtVersion=1.7.1
                '''
            }
        }
        stage ('Push Image') {
            steps {
                sh '''
                    docker login phx.ocir.io -u="weblogick8s/monica.riccelli@oracle.com" -p="WebLogic!PM1"
                    docker push ${IMAGE_TAG}
                '''
            }
        }
        stage('Roll Updates') {
            steps {
                sh '''
                    export KUBECONFIG=/scratch/k8s-demo/mrcluster_kubeconfig
                    export OCI_CLI_PROFILE=MONICA
                    export OCI_CONFIG_FILE=/var/lib/jenkins/.oci/config
                    export PATH=/var/lib/jenkins/bin:$PATH
                    kubectl get nodes
                    ls ./domain.yaml
                    kubectl create secret generic onprem-domain-weblogic-credentials --from-literal=username=weblogic --from-literal=password=welcome1 -n onprem-domain-ns
                    kubectl apply -f ./domain.yaml
                    kubectl get po -n onprem-domain-ns
                '''
            }
     }
  }
}

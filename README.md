
# Requirement untuk supervisor
1.	Memahami docker & Kubernetes
2.	Kubernets cluster
3.	Helm
4.	Jenkins
5.	Docker registry
# Steps untuk supervisor
1.	Membuat repository
2.	Membuat kubernets cluster
3.	Membuat docker-compose untuk local development
4.	Membuat helm chart
5.	Membuat Jenkins job
6.	Menginput detail aplikasi di modul ITSSO
# Detail steps untuk supervisor
1. membuat docker-compose, berikut adalah contoh template docker-compose

        version: "3.1"
        services:
            mariadb:
              container_name: c-db
              restart: always
              image: mariadb:latest
              ports:
                - "3306:3306"
              volumes: 
                - ./mysql:/var/lib/mysql
              environment:
                - MYSQL_DATABASE=laravel
                - MYSQL_ROOT_PASSWORD=root
            pma:
              container_name: c-pma
              restart: always
              image: phpmyadmin/phpmyadmin:latest
              ports: 
                - "8080:80"
              environment: 
                - PMA_HOST=mariadb
            app:
              container_name: c-app
              build: application
              restart: always
              ports:
                - "80:80"
              volumes:
                - ./application:/var/www/html

2. membuat deployment, service, volume, secrets, dan configmap yaml untuk keperluan helmchart

3. membuat jenkins job, berikut template jenkins job

            def Img
            node{
                stage('Checkout'){
                    checkout([$class: 'GitSCM', branches: [[name: 'NAMA_BRANCH']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'URL_GITHUB']]])
                }
                stage('Build Image'){
                    Img = docker.build('DOCKER REGISTRY','DIREKTORI DOCKERFILE')
                }
                stage('Push to Docker Registry'){
                    sh 'docker login --username USERNAME -p ACCESS TOKEN'
                    Img.push()
                }
                stage('Deploy to Rancher'){
                    withKubeConfig(caCertificate: '', clusterName: 'NAMA_CLUSTER', contextName: 'NAMA_CONTEXT', credentialsId: 'ID_CREDENTIAL', namespace: 'NAMESPACE', serverUrl: 'UR_CLUSTER') {
                        //sh 'kubectl create deployment nginx --image=nginx'
                        sh 'helm install helmchart ./helmchart'
                    }
                }
            }


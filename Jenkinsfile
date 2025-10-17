    // CREATE & UPDATE BY: GOMGOM SILALAHI
    // 2024-10-03
import groovy.json.JsonSlurperClassic

def deployment( env, jobname, docker_name, docker_network_name, docker_network_subnet, docker_network_gateway, dockerHubUser, dockerHubPass, packageJSONVersion) {

    // Tentukan ke environment mana deployment akan dilakukan
    def split_job_name = jobname.split('-')
    def appType = split_job_name[0]
    def userType = split_job_name[1]
    def hostname = sh(script: " bash ${workdir}/define_host.sh ${appType} ${userType} ${env}", returnStdout: true).trim()

    // Deployment sesuai environment
    withCredentials([string(credentialsId: "$hostname", variable: 'JSON_STRING')]) {
        script {
            def jsonData = new JsonSlurperClassic().parseText(JSON_STRING)

            def remote = [:]
            remote.name = "$hostname"
            remote.host = jsonData.hostname
            remote.user = jsonData.username
            remote.password = jsonData.password
            remote.allowAnyHosts = true

            // Create directory
            //sshCommand remote: remote, command: """ sh(script: "${workdir}/createdir.sh ${jobname}", returnStdout: true).trim() """
                if (hostname.startsWith('sit')) {
                sshCommand remote: remote, command: """
                        if [ ! -d "/data/${jobname}" ]; then
                            mkdir -p /data/${jobname}
                                echo "Folder ${jobname} berhasil dibuat."
                        else
                                echo "Folder ${jobname} sudah ada, tidak perlu membuat ulang."
                        fi
                """
                } else {
                sshCommand remote: remote, command: """
                        if [ ! -d "/data/${jobname}" ]; then
                            mkdir -p /data/${jobname}
                                echo "Folder /data/${jobname} berhasil dibuat."
                        else
                                echo "Folder /data/${jobname} sudah ada, tidak perlu membuat ulang."
                        fi
                """
                }

            echo " LANJUT CHECK/COPY DOCKER-COMPOSE.YAML DAN .ENV"
            sleep(3)

            // Put files
                if (hostname.startsWith('sit')) {
                    // Periksa yaml,env dan hapus file jika ada
                    sshCommand remote: remote, command: """
                    if [ -f ${jobname}/docker-compose.yaml ]; then
                        rm ${jobname}/docker-compose.yaml
                    fi
                    if [ -f ${jobname}/.env_${env} ]; then
                        rm ${jobname}/.env_${env}
                    fi
                    """
                // Salin file, baik file yang dihapus atau file baru
                    echo "salin file ke server $hostname"
                    sshPut remote: remote, from: "docker-compose.yaml", into: "${jobname}/"
                    sshPut remote: remote, from: ".env_${env}", into: "${jobname}/"
                
                }else {
                    sshCommand remote: remote, command: """
                    if [ -f ${jobname}/docker-compose.yaml ]; then
                        rm ${jobname}/docker-compose.yaml
                    fi
                    if [ -f ${jobname}/.env_${env} ]; then
                        rm ${jobname}/.env_${env}
                    fi
                    """
                // Salin file, baik file yang dihapus atau file baru
                    echo "salin file ke server $hostname"
                    sshPut remote: remote, from: "docker-compose.yaml", into: "/data/${jobname}"
                    sshPut remote: remote, from: ".env_${env}", into: "/data/${jobname}"
                }

            echo "next lanjut up pull image dan edit docker-compose file"

            // Docker login 
            sshCommand remote: remote, command: """
            cd /data/${jobname}
            echo "$dockerHubPass" | docker login -u "$dockerHubUser" --password-stdin
            mv .env_${env} .env
            sed -i 's/export //g' .env
            sed -i "s/'//g" .env
            sed -i 's|image: endiazequitylife/eli_application:0.0.1|image: endiazequitylife\\/${jobname}:dev|' docker-compose.yaml
            sed -i 's/^ *build:/    #build:/' docker-compose.yaml
            sed -i 's/^ *context: ./       #context: ./' docker-compose.yaml
            sed -i 's/^ *dockerfile: Dockerfile/       #dockerfile: Dockerfile/' docker-compose.yaml
            sed -i 's/net-eli-app/${docker_network_name}/g' docker-compose.yaml
            sed -i 's|ipv4_address: 0.0.0.2|ipv4_address: ${IPv4_service_1}|' docker-compose.yaml
            sed -i 's|ipv4_address: 0.0.0.3|ipv4_address: ${IPv4_service_2}|' docker-compose.yaml
            sed -i 's|ipv4_address: 0.0.0.4|ipv4_address: ${IPv4_service_3}|' docker-compose.yaml
            sed -i 's|ipv4_address: 0.0.0.5|ipv4_address: ${IPv4_service_4}|' docker-compose.yaml
            sed -i 's|ipv4_address: 0.0.0.6|ipv4_address: ${IPv4_service_5}|' docker-compose.yaml
            sed -i 's|subnet: 0.0.1.0/24|subnet: ${docker_network_subnet}|' docker-compose.yaml
            """

            // KONDISI UNTUK CONFIG GATEWAY BERBEDA ANTARA SIT UAT PRO KARENA PERBEDAAN VERSION DOCKER
            if (hostname.startsWith('sit')) {
                sshCommand remote: remote, command: """
                    echo "Hapus config gateway di docker compose"
                    cd /data/${jobname}
                    sed -i 's|gateway: 0.0.1.1|#gateway: 0.0.1.1|' docker-compose.yaml
                """
            }else {
                sshCommand remote: remote, command: """
                    echo "Ubah gateway di docker compose"
                    cd /data/${jobname}
                    sed -i 's|gateway: 0.0.1.1|#gateway: ${docker_network_gateway}|' docker-compose.yaml 
                """
            }

            echo "next lanjut up container"

        // Execute   
        //  export INFISICAL_VAULT_FILE_PASSPHRASE=${infisical_passphrase}
        //  infisical export --env=${infisical_env} --projectId=65a75e2867587e055c706b95 --format=dotenv-export > .env
        //  sed -i 's|^ *# image: docker-image:tag|    image: ${docker_name}|' docker-compose.yaml
        //  sed -i '/networks:\n  web-eli-cus:/a \ \ \ \ external: true' docker-compose.yaml
        //  sed -i "/image: endiazequitylife\\/api-esubmission:/s/^/#/" docker-compose.yaml 
        //  sed -i 's/- .env/- .env_${env}/g' docker-compose.yaml

            // Execute untuk build and up image 
            if (hostname.startsWith('sit')) {
                sshCommand remote: remote, command: """
                    echo "execute di env sit"
                    cd /data/${jobname}
                    docker-compose down
                    docker image rm endiazequitylife/${jobname}:dev
                    docker-compose -f docker-compose.yaml pull
                    docker-compose up -d                   
            """
            } else if (hostname.startsWith('pro-mw') || hostname.startsWith('pro-in')) {
                sshCommand remote: remote, command: """
                    echo "execute di env pro"
                    cd /data/${jobname}
                    dimage=\$(docker image ls | awk '{print \$1":"\$2}' | grep '^endiazequitylife/${jobname}:')
                    docker compose down
                    echo "\$dimage"
                    docker image rm -f "\$dimage"
                    docker pull endiazequitylife/${jobname}:${packageJSONVersion}
                    sleep 2
                    sed -i 's|image: endiazequitylife/${jobname}:dev|image: endiazequitylife\\/${jobname}:${packageJSONVersion}|' docker-compose.yaml
                    docker compose up -d
                """
            } else if (hostname.startsWith('pro-ex')) {
                sshCommand remote: remote, command: """
                    echo "execute di env pro"
                    cd /data/${jobname}
                    dimage=\$(docker image ls | awk '{print \$1":"\$2}' | grep '^endiazequitylife/${jobname}:')
                    docker-compose down
                    echo "\$dimage"
                    docker image rm -f "\$dimage"
                    docker pull endiazequitylife/${jobname}:${packageJSONVersion}
                    sleep 2
                    sed -i 's|image: endiazequitylife/${jobname}:dev|image: endiazequitylife\\/${jobname}:${packageJSONVersion}|' docker-compose.yaml
                    docker-compose up -d
                """
            } else {
                sshCommand remote: remote, command: """
                    echo "execute di env uat"
                    cd /data/${jobname}
                    docker compose down
                    docker image rm endiazequitylife/${jobname}:dev
                    docker compose -f docker-compose.yaml pull
                    docker compose up -d
                """
            }
        }
    }
}

def check_env(infisicalID, infisicalSE, infisical_env, InfisicalPID) {
    dir("${WORKSPACE}") {
        sh(script: "${workdir}/load_env.sh ${infisicalID} ${infisicalSE} ${infisical_env} ${InfisicalPID}", returnStdout: true).trim()
    }
}

pipeline {
    //agent any
    agent { label 'jenkins-agent' }
    options {
        //PENGATURAN UNTUK BUILD CONCURENT YANG BISA BERJALAN HANYA SATU DI TIAP NODE
        disableConcurrentBuilds(abortPrevious: false)
        throttleJobProperty(
            categories: ['MyThrottleCategory'],
            throttleEnabled: true,
            throttleOption: 'category'
        )
    }
    environment {
        // enable docker buildkit
        DOCKER_BUILDKIT='1'

        // Workdir
        def workdir = "${env.JENKINS_HOME}/script"
        
        // Set kode aplikasi
        //def jobname = "${env.JOB_NAME}".split('/').first()
        def jobname = "${params.applicationID}"

        // set parameter untuk git source code aplikasi 
        def git_repo_url = "${params.gitURL}"
        def commitMessage = "" // <- deklarasi global
        
        // Set docker login, image name, network name, range, gateway, ip container
        def dockerHubPass = credentials('DockerHubPass')
        def dockerHubUser = credentials('DockerHubUser')
        def docker_name = "endiazequitylife/$jobname"
        def docker_network_name = "${params.docker_network_name}"
        def docker_network_subnet = "${params.docker_network_subnet}"
        def docker_network_gateway = "${params.docker_network_gateway}"
        def IPv4_service_1 = "${params.IPv4_service_1}"
        def IPv4_service_2 = "${params.IPv4_service_2}"
        def IPv4_service_3 = "${params.IPv4_service_3}"
        def IPv4_service_4 = "${params.IPv4_service_4}"
        def IPv4_service_5 = "${params.IPv4_service_5}"

        // Set konfigurasi INFISICAL
        def infisicalID = credentials('infisicalID')
        def infisicalSE = credentials('infisicalSE')
        def InfisicalPID = "${params.InfisicalPID}"

        // get version 
        def packageJSONVersion = "0.0.0"

        //deklarasi nilai awal variable untuk approval
        def approval = 'true'
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                echo "Prepare"
                echo "- - - - - - - - - - - - - - - - - - - - - -"
                echo "Cleaned up workspace first for Project"
                echo "JOB_NAME              : ${env.JOB_NAME}"
                cleanWs()
                echo "- - - - - - - - - - - - - - - - - - - - - -"
            }
        }
    
        stage('Checkout SCM (2)') {
            steps {
                checkout([$class: 'GitSCM',
                        branches: [[name: "dev"]],
                        userRemoteConfigs: [[
                            url: "${git_repo_url}",
                            credentialsId: 'tokenID-gomgom-git'
                        ]]
                ])

                // ambil nilai commit message
                script {
                    // Assign nilai ke variabel global
                    commitMessage = sh(
                        script: "git log -1 --pretty=%B",
                        returnStdout: true
                        ).trim()
                        echo "COMMIT_MESSAGE : ${commitMessage}"
                }
            }
        } 

        stage("SCA") {
            steps {
                script {
                    try {
                        def scannerHome = tool 'SonarScanner';
                            withSonarQubeEnv() {
                                sh "${scannerHome}/sonar-scanner-6.1.0.4477/bin/sonar-scanner"
                                //sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar'
                                sleep(10)
                            }
                    } catch (Exception e) {
                        echo "failed SCA"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    try {
                        retry(2) {
                            timeout(time: 5, unit: 'MINUTES') {
                                waitForQualityGate abortPipeline: true
                            }
                        }
                    } catch (Exception e) {
                        echo "failed QG"
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    try {
                        //build image
                        retry(3) {
                            sh(script: "${workdir}/build_image.sh ${jobname} dev", returnStdout: true).trim()
                            echo "BUILD ${jobname}/dev berhasil"
                        }
                        echo " Lanjut proses delete image"
                        retry(2) {
                            sh """
                                if docker image ls --format "{{.Repository}}:{{.Tag}}" | grep -q '${jobname}:dev'; then
                                    docker image rm endiazequitylife/${jobname}:dev
                                else
                                    echo "image tidak ada"
                                fi
                            """
                        }
                    }catch (Exception e){
                        echo "failed stage build"
                        currentBuild.result = 'failure'
                        error("Failed stage build")
                    }
                }
            }
        }

        stage("SIT") {
            steps {
                script {
                    try {
                        //STEP 1 PULL ENV UNTUK SIT
                        echo "pull env untuk env SIT"
                        check_env(
                            "$infisicalID",
                            "$infisicalSE",
                            "sit",
                            "${InfisicalPID}")
                        echo "pull env untuk env SIT berhasil"

                        //STEP 2 DEPLOYMENT KE SERVER TUJUAN
                        echo " deployment ke environment SIT"
                        deployment(
                            "sit",
                            "$jobname",
                            "$docker_name:dev",
                            "$docker_network_name",
                            "$docker_network_subnet",
                            "$docker_network_gateway",
                            "$dockerHubUser",
                            "$dockerHubPass",
                            "$packageJSONVersion")
                        echo " deployment ke server SIT berhasil"

                        //STEP 3 DELETE ENV FILE
                        echo "Delete env_sit file"
                        sh("rm -rf .env_sit")

                        mail to: 'gomgom.silalahi@equity.id,endiaz.arsanto@equity.id,andrean.effendi@equity.id',
                        subject: "Jenkins - success (SIT): Deployment ${env.JOB_NAME} with pipeline number: (${env.BUILD_NUMBER})" ,
                        body: """Status deployment: success \nName: ${env.JOB_NAME} \nID: ${jobname} \nEnvironment: ${env.STAGE_NAME} \nPipeline number: (${env.BUILD_NUMBER}) \nCommit message: ${commitMessage}"""

                    } catch (Exception e) {
                        echo "failed deploy SIT"
                        mail to: 'gomgom.silalahi@equity.id, bona@equity.id, endiaz.arsanto@equity.id, andrean.effendi@equity.id',
                        subject: "Jenkins - failed (SIT): Deployment ${env.JOB_NAME} with pipeline number: (${env.BUILD_NUMBER})" ,
                        body: """Status deployment: failed \nName: ${env.JOB_NAME} \nID: ${jobname} \nEnvironment: ${env.STAGE_NAME} \nPipeline number: (${env.BUILD_NUMBER}) \nCommit message: ${commitMessage}
                                \nPlease go to url: ${BUILD_URL}consoleText for see error detail. 
                                \n
                                \nThank you"""
                        }
                    }
                }
        }

        stage("UAT") {
            steps {
                script {
                    try {
                        //STEP 1 PULL ENV UNTUK SIT
                        echo "pull env untuk env UAT"
                        check_env(
                            "$infisicalID",
                            "$infisicalSE",
                            "uat",
                            "${InfisicalPID}")
                        echo "pull env untuk env SIT berhasil"

                        //STEP 2 DEPLOYMENT KE SERVER TUJUAN
                        echo " deployment ke environment UAT"
                        deployment(
                            "uat",
                            "$jobname",
                            "$docker_name:dev",
                            "$docker_network_name",
                            "$docker_network_subnet",
                            "$docker_network_gateway",
                            "$dockerHubUser",
                            "$dockerHubPass",
                            "$packageJSONVersion")
                        echo " deployment ke server UAT berhasil"

                        //SETEP 3 DELETE ENV FILE
                        echo "Delete env_uat file"
                        sh("rm -rf .env_uat")

                        mail to: 'gomgom.silalahi@equity.id,endiaz.arsanto@equity.id,andrean.effendi@equity.id',
                        subject: "Jenkins - success (UAT): Deployment ${env.JOB_NAME} with pipeline number: (${env.BUILD_NUMBER})" ,
                        body: """Status deployment: success \nName: ${env.JOB_NAME} \nID: ${jobname} \nEnvironment: ${env.STAGE_NAME} \nPipeline number: (${env.BUILD_NUMBER}) \nCommit message: ${commitMessage}"""

                    } catch (Exception e) {
                        echo "failed deploy UAT"
                        mail to: 'gomgom.silalahi@equity.id, bona@equity.id, endiaz.arsanto@equity.id, andrean.effendi@equity.id ',
                        subject: "Jenkins - failed (UAT): Deployment ${env.JOB_NAME} with pipeline number: (${env.BUILD_NUMBER})" ,
                        body: """Status deployment: failed \nName: ${env.JOB_NAME} \nID: ${jobname} \nEnvironment: ${env.STAGE_NAME} \nPipeline number: (${env.BUILD_NUMBER}) \nCommit message: ${commitMessage}
                                \nPlease go to url: ${BUILD_URL}consoleText for see error detail. 
                                \n
                                \nThank you"""
                        }
                    }
                }
        }

        stage("PRO"){
            steps{
                script{
                    stage("PRO-approval"){
                        script {
                            //EMAIL NOTIFIKASI UNTUK APPROVAL DEPLOYMENT PRODUCTION
                            try {
                                mail (to: 'gomgom.silalahi@equity.id, bona@equity.id, endiaz.arsanto@equity.id, andrean.effendi@equity.id',
                                    subject: "Jenkins - Production deployment for ${env.JOB_NAME} with pipeline number (${env.BUILD_NUMBER}) is waiting for approval:",
                                    body: """Deployment '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}) with commit message: ${commitMessage} will deploy to Production. 
                                            \nPlease go to job url: ${env.BUILD_URL} to 'Approved' or 'Aborted'.
                                            \n 
                                            \nStep by step for 'Approved' or 'Aborted': 
                                            \n1. Click url ${env.BUILD_URL} 
                                            \n2. On the left side, select the 'Paused for input' menu
                                            \n3. Select 'Approved' or 'Aborted'
                                            \nnoted: If there is no input for >10 minutes, the job pipeline will automatically aborted. 
                                            \nThank you""");
                                timeout(time:10, activity: false, unit:'MINUTES') {
                                    def userInput = input(id: 'userInput', message: 'Deploy to environment production', ok: 'Yes')
                                        echo "LANJUT DEPLOYMENT KE PRODUCTION"
                                }
                            }catch (Exception e) {
                                echo "Jenkins - Deployment '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}) with commit message: ${commitMessage} was cancelled due to approval timeout or deployment not approved"
                                //EMAIL NOTIFIKASI JIKA DEPLOYMENT PRODUCTION GAGAL ATAU TIDAK DI APPROVE
                                mail (to: 'gomgom.silalahi@equity.id, bona@equity.id, endiaz.arsanto@equity.id, andrean.effendi@equity.id',
                                    subject: "Jenkins - Production deployment for '${env.JOB_NAME}' - ID: '${jobname}' was cancelled",
                                    body: """Deployment '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}) with commit message: ${commitMessage} was cancelled due to approval timeout or deployment not approved.
                                            \nPlease go to job url ${env.BUILD_URL} for re-try the production deployment. 
                                            \n
                                            \nFor the log, you can see it via the following URL: ${BUILD_URL}consoleText
                                            \nThank you""");
                                currentBuild.result = 'aborted'
                                approval = 'false'
                            }
                        }   
                    }
                }

                script{
                    stage("PRO-versioning") {
                        try{
                            script {
                                if(approval=='true'){
                                    // Step untuk menjalankan npm release
                                    echo "Menjalankan npm release..."
                                    // STEP 1 Menjalankan perintah npm release lewat script checkbranch.sh
                                    retry(2){
                                        def branch = 'release'
                                        def checkbranch = sh(script: "${workdir}/checkbranch.sh ${env.WORKSPACE}", returnStdout: true).trim()
                                        echo "${checkbranch}"
                                        if(checkbranch=='1'){
                                            sh """
                                                cd ${env.WORKSPACE}
                                                git branch release      
                                                git checkout release
                                                npm run release
                                                echo "release version selesai"
                                            """
                                        }else {
                                            sh """
                                                echo "branch ${branch} ada"
                                                cd ${env.WORKSPACE}
                                                git branch release
                                                git checkout release
                                                npm run release
                                                echo "release version selesai"
                                            """
                                        }
                                    }       
                                }         
                            }
                        }catch (Exception e) {
                            echo "failed stage pro-versioning"
                            mail (to: 'gomgom.silalahi@equity.id, bona@equity.id, endiaz.arsanto@equity.id, andrean.effendi@equity.id',
                                    subject: "Jenkins - FAILED: Production deployment for '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}",
                                    body: """Deployment '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}) was failed on stage PRO-versioning.
                                            \nPlease go to url: ${BUILD_URL}consoleText for see error detailed or the log. 
                                            \n
                                            \nThank you""");
                            error("Failed stage pro-versioning")
                        }                
                    }

                    stage("PRO-build"){
                        try{
                            script{
                                if(approval=='true'){
                                    // definisikan variable versioning
                                    def packageJSON = readJSON file: 'package.json'
                                    def packageJSONVersion = packageJSON.version
                                    echo "${jobname} version ${packageJSONVersion}"

                                    //STEP 2 BUILD IMAGE USING DOCKER BUILDCLOUD
                                    try {
                                        retry(2) {
                                            sh(script: "${workdir}/build/prod/build_image_prod.sh ${jobname} ${packageJSONVersion}", returnStdout: true).trim()
                                            echo "PRO - BUILD ${jobname}/${packageJSONVersion} berhasil"
                                        } 
                                    }catch (Exception e){
                                        echo "failed build image on production"
                                        error("Failed build image on production")
                                    }
                                }
                            }
                        }catch (Exception e) {
                            echo "failed stage pro-build"
                            mail (to: 'gomgom.silalahi@equity.id, bona@equity.id, endiaz.arsanto@equity.id, andrean.effendi@equity.id',
                                    subject: "Jenkins - FAILED: Production deployment for '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}",
                                    body: """Deployment '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}) was failed on stage PRO-build image.
                                            \nPlease go to url: ${BUILD_URL}consoleText for see error detailed or the log. 
                                            \n
                                            \nThank you""");
                            error("Failed stage pro-build")
                        }
                    }

                    stage("PRO-deployment"){
                        try{
                            script{
                                if(approval=='true'){
                                    def packageJSON = readJSON file: 'package.json'
                                    def packageJSONVersion = packageJSON.version
                                    //STEP 3 PULL ENV UNTUK PRO
                                    echo "pull env untuk env PRO"
                                    retry(2){
                                        check_env(
                                            "$infisicalID",
                                            "$infisicalSE",
                                            "pro",
                                            "${InfisicalPID}")
                                    }
                                    echo "pull env untuk env PRO berhasil"

                                    //STEP 4 DEPLOY KE ENVIRONMENT PRODUCTION
                                    try {
                                        //DEPLOY
                                        echo " deployment ke environment PRO"
                                        retry(2){
                                            deployment(
                                            "pro",
                                            "$jobname",
                                            "$docker_name:$packageJSONVersion",
                                            "$docker_network_name",
                                            "$docker_network_subnet",
                                            "$docker_network_gateway",
                                            "$dockerHubUser",
                                            "$dockerHubPass",
                                            "$packageJSONVersion")
                                        }
                                        echo " deployment ke server PRO berhasil"
                                    } catch (Exception e) {
                                        echo "Failed process deployment on production"
                                        error("Failed process deployment on production")
                                    }
                                    //STEP 5 DELETE ENV FILE
                                    echo "Delete env_pro file"
                                    retry(2){
                                        sh("rm -rf .env_pro")
                                    }
                                    
                                    mail to: 'gomgom.silalahi@equity.id ',
                                    subject: "Jenkins - success (PRO): Deployment '${env.JOB_NAME}' with pipeline number: (${env.BUILD_NUMBER})" ,
                                    body: """Status deployment: success \nName: ${env.JOB_NAME} \nID: ${jobname} \nEnvironment: 'Production' \nPipeline number: (${env.BUILD_NUMBER}) \nCommit message: ${commitMessage}"""  
                                }
                            }
                        }catch (Exception e) {
                            echo "failed stage pro-deployment"
                            mail (to: 'gomgom.silalahi@equity.id, bona@equity.id, endiaz.arsanto@equity.id, andrean.effendi@equity.id',
                                    subject: "Jenkins - failed (PRO): Deployment ${env.JOB_NAME} with pipeline number: (${env.BUILD_NUMBER})" ,
                                    body: """Status deployment: failed \nName: ${env.JOB_NAME} \nID: ${jobname} \nEnvironment: Production \nPipeline number: (${env.BUILD_NUMBER}) \nCommit message: ${commitMessage}
                                            \nPlease go to url: ${BUILD_URL}consoleText for see error detail. 
                                            \n
                                            \nThank you""");
                            error("Failed stage pro-deployment")
                        }
                    } 

                    stage("post-production"){
                        try{
                            script{
                                if(approval=='true'){
                                    def packageJSON = readJSON file: 'package.json'
                                    def packageJSONVersion = packageJSON.version
                                    //STEP 6 LANJUT PUSH SOURCE CODE YG ADA PADA LOKAL JENKINS KE GIT REPO BRANCH release
                                    retry(2){
                                        sh """
                                            echo "push to gitsource untuk ${jobname} version ${packageJSONVersion} branch release"
                                            cd ${env.WORKSPACE}
                                            git push --force origin release
                                        """
                                    }
                                    echo "push ke repo berhasil"

                                    //STEP 7 LAKUKAN MERGE DARI BRANCH RELEASE KE BRANCH MASTER
                                    retry(2){
                                        sh """
                                            cd ${env.WORKSPACE}
                                            sleep 3
                                            echo "merge ${jobname} version ${packageJSONVersion} ke branch release"
                                            glab mr create --source-branch release --target-branch master --title "merge ${jobname} version ${packageJSONVersion} into master" --description "This merge request for ${jobname} version ${packageJSONVersion} "
                                        """
                                    }

                                    //glab auth login --hostname gitsource.myequity.id --stdin < ${tokenGIT}
                                    //sh("git tag -a ${version_number} -m '${split_job_name}:${version_number}'")
                                    //git pull "http://${GIT_USERNAME}:${GIT_PASSWORD}@gitsource.myequity.id/it-delivery/flagship-project/frontend-customer-portal.git" dev

                                    //STEP 8 DELETE IMAGE LAMA YANG ADA DI JENKINS UNTUK PRODUCTION IMAGE
                                    retry(2) {
                                        sh """
                                            if docker image ls --format "{{.Repository}}:{{.Tag}}" | grep -q 'endiazequitylife/${jobname}:${packageJSONVersion}'; then
                                                docker image rm endiazequitylife/${jobname}:${packageJSONVersion}
                                            else
                                                echo "image tidak ada"
                                            fi
                                        """
                                    }
                                }
                            }
                        }catch (Exception e) {
                            echo "failed stage post-production"
                            mail (to: 'gomgom.silalahi@equity.id, bona@equity.id, endiaz.arsanto@equity.id, andrean.effendi@equity.id',
                                    subject: "Jenkins - FAILED: Production deployment for '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}",
                                    body: """Deployment '${env.JOB_NAME}' - ID: '${jobname}' pipeline: (${env.BUILD_NUMBER}) was failed on stage post-production.
                                            \nPlease go to url: ${BUILD_URL}consoleText for see error detailed or the log. 
                                            \n
                                            \nThank you""");
                        }
                    }   
                }
            }
        }    
    }

    //POST PIPELINE
    post {
        //valid conditions are [always, changed, fixed, regression, aborted, success, unsuccessful, unstable, failure, notBuilt, cleanup]
        success {
            mail (to: 'gomgom.silalahi@equity.id,endiaz.arsanto@equity.id,andrean.effendi@equity.id',
                subject: "Jenkins - success-pipeline: '${env.JOB_NAME}' pipeline number: (${env.BUILD_NUMBER})",
                body: """Status deployment: success \nName: ${env.JOB_NAME} \nID: ${jobname} \nPipeline number: (${env.BUILD_NUMBER}) \nCommit message: ${commitMessage}
                        \nFor details pipeline job, please see the following link: ${env.BUILD_URL}.
                        \n
                        \nThank you"""
            )
        }
        failure {
            mail (to: 'gomgom.silalahi@equity.id,endiaz.arsanto@equity.id,andrean.effendi@equity.id',
                subject: "Jenkins - failed-pipeline: '${env.JOB_NAME}' - ID: '${jobname}' - pipeline: (${env.BUILD_NUMBER})",
                body: """Deployment '${env.JOB_NAME}' - ID: '${jobname}' - pipeline: (${env.BUILD_NUMBER}) was failed. 
                        \nFor details pipeline job, please see the following link: ${env.BUILD_URL}.
                        \n
                        \nThank you"""
            )
        }
        aborted {
            echo "Deployment '${env.JOB_NAME}' - ID: '${jobname}' - pipeline: (${env.BUILD_NUMBER}) was aborted."
        }
    }
}

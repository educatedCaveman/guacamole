pipeline {
    agent any 

    environment {
        ANSIBLE_REPO = '/var/lib/jenkins/workspace/ansible_master'
        WEBHOOK = credentials('JENKINS_DISCORD')
        PORTAINER_DEV_WEBHOOK = credentials('PORTAINER_WEBHOOK_DEV_GUAC')
        PORTAINER_PRD_WEBHOOK = credentials('PORTAINER_WEBHOOK_PRD_GUAC')
    }

    //triggering periodically so the code is always present
    // run every friday at 3AM
    triggers { cron('0 3 * * 5') }

    stages {
        // deploy code to lv-426.lab, when the branch is 'dev_test'
        stage('deploy code (DEV)') {
            when { branch 'dev_test' }
            steps {
                // deploy configs to DEV
                echo 'deploy docker config files (DEV)'
                sh 'ansible-playbook ${ANSIBLE_REPO}/deploy/docker/deploy_docker_compose_dev.yml --extra-vars repo="guacamole"'
            }
        }
        // trigger portainer redeploy
        // separated out so this only gets run if the ansible playbook doesn't fail
        stage('redeploy portainer stack (DEV)') {
            when { branch 'dev_test' }
            steps {
                // deploy configs to DEV
                echo 'Redeploy DEV stack'
                sh 'http post ${PORTAINER_DEV_WEBHOOK}'
            }
        }

        // deploy code to sevastopol, when the branch is 'master'
        stage('deploy code (PRD)') {
            when { branch 'master' }
            steps {
                // deploy configs to PRD
                echo 'deploy docker config files (PRD)'
                sh 'ansible-playbook ${ANSIBLE_REPO}/deploy/docker/deploy_docker_compose_prd.yml --extra-vars repo="guacamole"'
            }
        }
        // trigger portainer redeploy
        // separated out so this only gets run if the ansible playbook doesn't fail
        stage('redeploy portainer stack (PRD)') {
            when { branch 'master' }
            steps {
                // deploy configs to DEV
                echo 'Redeploy PRD stack'
                sh 'http post ${PORTAINER_PRD_WEBHOOK}'
            }
        }

    }
    post {
        always {
            discordSend \
                description: "${JOB_NAME} - build #${BUILD_NUMBER}", \
                // footer: "Footer Text", \
                // link: env.BUILD_URL, \
                result: currentBuild.currentResult, \
                // title: JOB_NAME, \
                webhookURL: "${WEBHOOK}"
        }
    }
}


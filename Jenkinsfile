pipeline {
    agent any 

    environment {
        ANSIBLE_REPO = '/var/lib/jenkins/workspace/ansible_master'
        SCRIPTS_REPO = '/var/lib/jenkins/workspace/scripts_master'
        PORTAINER_DEV_PASS = credentials('PORTAINER_DEV_PASS')
        PORTAINER_PRD_PASS = credentials('PORTAINER_PRD_PASS')
        LOCAL_REPO_DEV = '/var/lib/jenkins/workspace/docker_dev_test'
        LOCAL_REPO_PRD = '/var/lib/jenkins/workspace/docker_master'
        WEBHOOK = credentials('JENKINS_DISCORD')
    }

    //triggering periodically so the code is always present
    // run every friday at 3AM
    triggers { cron('0 3 * * 5') }

    stages {
        // deploy code to lv-426.lab, when the branch is 'dev_test'
        stage('deploy dev code') {
            when { 
                expression { env.BRANCH_NAME == 'dev_test' } 
            }
            steps {
                // deploy configs to DEV
                echo 'deploy docker config files (DEV)'
                sh 'ansible-playbook ${ANSIBLE_REPO}/deploy/docker/deploy_docker_compose_dev.yml --extra-vars repo="homer"'
            }
        }

        // deploy code to sevastopol, when the branch is 'master'
        stage('deploy prd code') {
            when { branch 'master' }
            steps {
                // deploy configs to PRD
                echo 'deploy docker config files (PRD)'
                sh 'ansible-playbook ${ANSIBLE_REPO}/deploy/docker/deploy_docker_compose_prd.yml --extra-vars repo="homer"'
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


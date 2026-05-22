def failureMessages = []

pipeline {
    agent { label 'linux-amd64' }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    environment {
        ANSIBLE_GALAXY_TOKEN = credentials('ansible-roles-galaxy-token')
        GITHUB_TOKEN = credentials('inviqa-ansible-roles-releases')
        SLACK_NOTIFICATION_CHANNEL = 'ops-integrations'
        SLACK_NOTIFICATIONS_ENABLED = 'true'
        SLACK_TOKEN_CREDENTIAL_ID = 'inviqa-slack-integration-token'
    }

    parameters {
        string(
            name: 'RELEASE_VERSION',
            defaultValue: '',
            description: 'Optional release version to publish. Leave blank to use the latest concrete CHANGELOG.md release section.'
        )
        booleanParam(
            name: 'PUBLISH_GITHUB_RELEASE',
            defaultValue: true,
            description: 'On main only, publish the GitHub release after validation succeeds.'
        )
        booleanParam(
            name: 'PUBLISH_ANSIBLE_GALAXY_RELEASE',
            defaultValue: true,
            description: 'On main only, import the validated release into Ansible Galaxy after validation succeeds.'
        )
    }

    stages {
        stage('Build') {
            steps {
                sh 'ws enable'
                milestone(10)
            }
            post {
                failure {
                    script { failureMessages << 'Workspace environment failed to enable' }
                }
            }
        }

        stage('Linting') {
            steps {
                sh 'ws ansible lint'
            }
            post {
                failure {
                    script { failureMessages << 'Ansible linting failed' }
                }
            }
        }

        stage('Syntax checks') {
            steps {
                sh 'ws ansible syntax'
            }
            post {
                failure {
                    script { failureMessages << 'Ansible playbook syntax checks failed' }
                }
            }
        }

        stage('Container tests') {
            steps {
                sh 'ws test-docker'
            }
            post {
                failure {
                    script { failureMessages << 'Container-backed Fail2Ban role tests failed' }
                }
            }
        }

        stage('Release preflight') {
            steps {
                withEnv(["RELEASE_VERSION=${params.RELEASE_VERSION ?: ''}"]) {
                    sh '''
                        set +e
                        ws github release check
                        github_release_status="$?"
                        set -e

                        [ "${github_release_status}" = 0 ] || [ "${github_release_status}" = 2 ] || exit "${github_release_status}"

                        ws ansible galaxy check-token
                    '''
                }
            }
            post {
                failure {
                    script { failureMessages << 'Release preflight checks failed' }
                }
            }
        }

        stage('Publish GitHub release') {
            when {
                allOf {
                    branch 'main'
                    expression { return params.PUBLISH_GITHUB_RELEASE }
                }
            }
            steps {
                withEnv(["RELEASE_VERSION=${params.RELEASE_VERSION ?: ''}"]) {
                    sh 'ws github release publish'
                }
            }
            post {
                failure {
                    script { failureMessages << 'GitHub release publication failed' }
                }
            }
        }

        stage('Publish Ansible Galaxy release') {
            when {
                allOf {
                    branch 'main'
                    expression { return params.PUBLISH_ANSIBLE_GALAXY_RELEASE }
                }
            }
            steps {
                withEnv(["RELEASE_VERSION=${params.RELEASE_VERSION ?: ''}"]) {
                    sh 'ws ansible galaxy publish'
                }
            }
            post {
                failure {
                    script { failureMessages << 'Ansible Galaxy release publication failed' }
                }
            }
        }
    }

    post {
        failure {
            script {
                def message = "ansible-fail2ban: ${env.JOB_BASE_NAME} #${env.BUILD_NUMBER} - Failure after ${currentBuild.durationString.minus(' and counting')} (<${env.RUN_DISPLAY_URL}|View Build>)"
                def fallbackMessages = [ message ]
                def fields = []

                def failureMessage = failureMessages.join("\n")
                if (failureMessage) {
                    fields << [
                        title: 'Reason(s)',
                        value: failureMessage,
                        short: false
                    ]
                    fallbackMessages << failureMessage
                }
                def attachments = [
                    [
                        text: message,
                        fallback: fallbackMessages.join("\n"),
                        color: 'danger',
                        fields: fields
                    ]
                ]
                if (env.SLACK_NOTIFICATIONS_ENABLED == 'true') {
                    slackSend(channel: env.SLACK_NOTIFICATION_CHANNEL, color: 'danger', attachments: attachments, tokenCredentialId: env.SLACK_TOKEN_CREDENTIAL_ID)
                } else {
                    echo "Slack ${currentBuild.currentResult} notification skipped; SLACK_NOTIFICATIONS_ENABLED is false."
                }
            }
        }
        always {
            sh 'ws destroy'
            cleanWs()
        }
    }
}

pipeline {
    agent {
        label 'AGENT-1' 
    }
    options{
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds() 
    }
    parameters{ 
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'uat', 'prod', 'pre-prod'], description: 'select your environment')  
        string(name: 'VERSION', description: 'Enter Your App Version') 
        string(name: 'jira-id', description: 'Enter Your jira-id') 
    } 
    environment{
        appVersion = ''  // we can use this env variables across the pipeline 
        region = 'us-east-1' 
        account_id = ''
        project = 'expense'
        environment = ''
        component = 'backend'
    }
    stages {
        stage('Setup Environment') {
            steps{
                script{
                    environment = params.ENVIRONMENT
                    appVersion = params.VERSION 
                    account_id = pipelineGlobals.getAccountID(environment) 
                }
            }
        }
        stage('Integration tests') {
            when {
                expression {params.ENVIRONMENT == 'qa'}
            }
            steps{
                script{
                    sh """
                        echo "run integration tests"
                    """
                }
            }
        }
        stage('check JIRA') {
            when {
                expression {params.ENVIRONMENT == 'prod'}
            }
            steps {
                script {
                    sh """
                        echo "check jira status"
                        echo "check jira deployment window"
                        echo "fail pipeline if above two are not true"
                    """
                }
            }
        }
        stage('Deploy') { 
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-credentials') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${environment}

                        cd helm

                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml

                        helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml .
                    """
                } 
            }
        }    
    } 
    post{
        always{
            echo "this section runs always" 
            deleteDir() 
        }
        success{
            echo "this section runs when pipeline is success"
        }
        failure{
            echo "this section runs when pipeline is failure" 
        }
    }
} 
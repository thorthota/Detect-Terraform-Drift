def mailRecipients = "john.doe@vmail.com"
def jobName = currentBuild.fullDisplayName

pipeline {
  agent {
    label 'terraform-hashicorp'
  }

  stages {
    stage('Checkout scm') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'my_gitHub_Token', url: 'https://github.com/repo-name.git']]])
      }
    }
    stage('Terraform Init') {
      steps {
        container('terraform') {
          withCredentials([
            [$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'Account-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
            script {

              sh "terraform init -input=false"

            }
          }
        }
      }
    }
    stage('Terraform Plan') {
      steps {
        container('terraform') {
          withCredentials([
            [$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'Account-creds', secretKeyVariable:'AWS_SECRET_ACCESS_KEY']
          ]) {
            script {
              sh '''
					terraform plan -out=tfplan -input=false >> terraform-plan.log
	                cat terraform-plan.log
	                if (grep -r "Terraform will perform the following actions" terraform-plan.log); then touch stringFound; fi
	                     
                 '''

              stage('Send email') {

                if (fileExists('stringFound'))   {
                  emailext body: '''Terraform Drift has been detected. Take action''',
                  mimeType: 'text/html',
                    attachLog: true,
                    subject: "[Jenkins] ${jobName}",
                    to: "${mailRecipients}",
                    replyTo: "${mailRecipients}",
                    recipientProviders: [[$class: 'CulpritsRecipientProvider']]
                }
                 else {
                           echo "No changes detected. Infrastructure is up-to-date"
                       }

              }
            }
          }

        }
      }
    }
  }

}

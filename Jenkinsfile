import groovy.transform.Field
import groovy.json.JsonSlurper 

String fileContents = new File('/home/hp/JenkinsSecondInstance/input.json').text
String username, passwd, key, value

pipeline {
    agent any
   
    stages
    {   
        
      stage('ParsingJson') 
      {
        steps 
        {
            script{
                def jsonSlurper = new JsonSlurper()
                def object = jsonSlurper.parseText(fileContents)
                username = object.username
                passwd = object.password
                print username
                print passwd
            }
            
        }
        
      }
      
      stage ('GettingCredfromSECONDinstance')
        {
            steps
            {
                script
                {
                    sh "java -jar '/home/hp/JenkinsSecondInstance/jenkins-cli.jar' -auth ${username}:${passwd} -s http://127.0.0.1:8081 who-am-i"
                    sh "java -jar '/home/hp/JenkinsSecondInstance/jenkins-cli.jar' -auth ${username}:${passwd} -s http://127.0.0.1:8081 list-credentials-as-xml system::system::jenkins > cred.xml"
                    
                    sh "java -jar '/home/hp/JenkinsSecondInstance/jenkins-cli.jar' -s http://127.0.0.1:8080 import-credentials-as-xml system::system::jenkins <  cred.xml"
                }
            }
        }
        
        stage ('DecryptingAndRetrievingData')
        {
            steps
            {
                script
                {
                 ansibleVault(action: 'decrypt', input: './secr.json', vaultCredentialsId: 'ansiblefilek')
                      print "Decrypted"
                      //String avfile = sh ( script: "cat secr.json" , returnStdout: true)
                      def somefile = new File ("/var/lib/jenkins/workspace/getting from second instance/secr.json")
                      String avfile = somefile.text
                      def jsonSlurper = new JsonSlurper()
                      def object = jsonSlurper.parseText(avfile)
                         key = "thekey"
                         value = object.key
                         print key
                         print value 
                }            
                    
            }        
                
        }
        stage('Encrypting')
        {
            steps
            {
                script
                {
                    ansibleVault(action: 'encrypt', input: './secr.json', vaultCredentialsId: 'ansiblefilek')
                    print "Ecrypted"
                }
            }
        }
        stage('Putting into Vault')
        {
             environment 
                {
                  UNP_CRED = credentials('firstcred')
                  VAULT_ADDR='http://127.0.0.1:8200'
                }
             steps
                {
                  script
                    {
                     def configuration = [vaultUrl: 'http://127.0.0.1:8200',  vaultCredentialId: 'my-role-id', engineVersion: 2]
                     def secrets = [  [path: 'secret/adding', engineVersion: 2, secretValues: [ ]],]
                     withVault([configuration: configuration, vaultSecrets: secrets])
                        {
                        sh "vault kv put secret/adding ${UNP_CRED_USR}=${UNP_CRED_PSW} "
                        sh "vault kv get secret/adding"
                        sh "vault kv put secret/adding ${key}=${value} "
                        sh "vault kv get secret/adding"
                        } 
                    }
                }       
          
        }
      
    }
}

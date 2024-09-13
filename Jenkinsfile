pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }
    stage('Login into Org') {
      steps{
          script
        {
                   try {
                            sh 'sfdx force:auth:jwt:grant --instance-url ${SF_PROD_URL} --client-id ${MFIPROD_CONSUMER_KEY} --username ${MFIPROD_USR_NAME} --jwt-key-file ${SF_PROD_KEY} -s'
                        } 
                          catch (Exception e) {
                            echo 'Exception occurred: ' + e.toString()
                          }
        }
      }
    }
            stage('validate-code'){
            when {     
                expression {
                    env.BRANCH_NAME =~ 'feature/master/*' || env.BRANCH_NAME =~ 'PR-*';
                }                
            } 
            steps{
                echo'======== Validating Changes ============'
                validate()
            }
                stage('Deploy to Org'){
            when {     
                expression {
                    return env.BRANCH_NAME == 'master'
                }                
            }
            steps{
                echo'======== Deploying Changes ============'
                deploy()
            }
         def validate(){
    sh "rm -fr changes"
    sh "mkdir changes"
        sh "git diff --name-only origin/${env.BRANCH_NAME}..origin/mfitest1"
        echo "_____  creating package.xml and destructive changes---------"                            
        sh "sfdx sgd:source:delta --to 'origin/${env.BRANCH_NAME}' --from 'origin/mfitest1'  --output changes/"
  
        
    echo "***  package xml with added modified metadata ****"
    sh "cat changes/package/package.xml"

    echo ""
    echo "--- destructiveChanges.xml generated with deleted metadata ---"
    sh "cat changes/destructiveChanges/destructiveChanges.xml"
    echo  "" 

    def addSize = getXMLoutput('package/package.xml')
    echo "Additons ===== ${addSize}"
    if (addSize != 0) {
            echo "***** There are changes in package.xml and validating now ******"
            sh "sfdx project deploy start --manifest changes/package/package.xml --dry-run -l NoTestRun --verbose -w 50"
    }
    else {
        echo "No change to Apply"
    }

    def removeSize = getXMLoutput('destructiveChanges/destructiveChanges.xml')
    echo "${removeSize}"
    if (removeSize != 0) {
            echo "***** There are changes in destructiveChanges.xml and validating now ******"
            sh "sfdx project deploy start --manifest changes/destructiveChanges/package.xml --post-destructive-changes changes/destructiveChanges/destructiveChanges.xml --dry-run --verbose -w 5"
    }
    else {
        echo "No change to Delete"
    }
}
def deploy(){
     ///Assume deploy only happens from proper branch , hence compare head   
    sh "rm -fr changes"
    sh "mkdir changes"
    echo "**** ALL changed files in ****"
    sh "git diff --name-only HEAD^ HEAD"
    echo "_____  creating package.xml and destructive changes---------"                            
    sh "sfdx sgd:source:delta --to 'HEAD' --from 'HEAD^'   --output changes/"
    echo "***  package xml with added modified metadata ****"
    sh "cat changes/package/package.xml"

    echo ""
    echo "--- destructiveChanges.xml generated with deleted metadata ---"
    sh "cat changes/destructiveChanges/destructiveChanges.xml"
    echo  "" 

    def addSize = getXMLoutput('package/package.xml')
    echo "Additons ===== ${addSize}"
    if (addSize != 0) {
            echo "***** There are changes in package.xml and Deploying now ******"
            sh "sfdx project deploy start --manifest changes/package/package.xml -l NoTestRun --verbose -w 50"
    }
    else {
        echo "No change to Apply"
    }

    def removeSize = getXMLoutput('destructiveChanges/destructiveChanges.xml')
    echo "${removeSize}"
    if (removeSize != 0) {
            echo "***** There are changes in destructiveChanges.xml and Deploying now ******"
            sh "sfdx project deploy start --manifest changes/destructiveChanges/package.xml --post-destructive-changes changes/destructiveChanges/destructiveChanges.xml --verbose -w 5"
    }
    else {
        echo "No change to Delete"
    }
}
/**
 * Function to parse xml and gives output
 * getXMLoutput --
**/
def getXMLoutput(pParam) {
    def File = readFile "${env.WORKSPACE}/changes/${pParam}"
    echo "${File}"
    def types = 0
    def xml = new XmlSlurper().parseText(File)
    types = xml.types.size()
    return types;
}
}

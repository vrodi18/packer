node("packer"){
    stage('Pull Repo') {
        println(env.BRANCH_NAME)
    }
}
def aws_region_var = ''
def aws_environment = ''
if(env.BRANCH_NAME ==~ "dev.*"){
    println("Creating image at Dev")
    aws_region_var = "us-east-1"
    aws_environment = 'Dev'
}
else if(env.BRANCH_NAME ==~ "qa.*") {
    println("Creating image at QA")
    aws_region_var = "us-east-2"
    aws_environment = 'QA'
}
else if(env.BRANCH_NAME ==~ "master") {
    println("Creating image at Prod")
    aws_region_var = "us-west-2"
    aws_environment = 'Prod'
}
def random_name = "${env.BRANCH_NAME}-${ UUID.randomUUID().toString()}"
node("packer"){
    stage('Pull Repo') {
        git url: 'https://github.com/ikambarov/packer.git'
    }
    withCredentials([usernamePassword(credentialsId: 'jenkins_aws_keys', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        withEnv(["AWS_REGION=${aws_region_var}", "PACKER_AMI_NAME=${random_name}"]) {
            stage('Packer Validate') {
                sh 'packer validate apache.json'
            }
            stage('Packer Build') {
                sh """
                    packer build apache.json | tee output.log
                """
            }
            stage('Trigger Deploy Instance'){
                build wait: false, job: 'terraform-ec2-by-name', parameters: [
                    string(name: 'ACTION', value: 'Apply'),
                    string(name: 'ENVIRONMENT', value: "${aws_environment}"),
                    string(name: 'AMI_NAME', value: "${random_name}")
                ]
            }
        }
    }
}

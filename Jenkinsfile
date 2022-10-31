pipeline {
    agent {label "slave"}  // cont-slave / ec2-slave
    stages {
        stage('build and push') {
            steps {
                script {
                    if ( env.BRANCH_NAME == "release" ) {                   
                        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USR', passwordVariable: 'PASS')]) { 
                        sh """
                            
                            docker build .  -t yossefsameh/bakehouse-app:$BUILD_NUMBER
                            docker login -u ${USR} -p ${PASS}
                            docker push yossefsameh/bakehouse-app:$BUILD_NUMBER
                            echo ${BUILD_NUMBER} > ../bakehouse-build-num.txt
                            
                        """   
                        }
                    }
              }     
            }
        }
        stage('deploy') {
            steps {

                script {    

                    if ( env.BRANCH_NAME == "dev" || env.BRANCH_NAME == "master" || env.BRANCH_NAME == "prod" ) {   
                   
                        withCredentials([file(credentialsId: 'kubectl', variable: 'kubectl-config')]){

                            sh """
                                    export BUILD_NUMBER=\$(cat ../bakehouse-build-num.txt)
                                    mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                    cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                    rm -f Deployment/deploy.yaml.tmp
                                    kubectl apply --kubeconfig=${kubectl-config} -f Deployment
                            """   
                        }
                    }    
                }
            }        
        }                
    }
}

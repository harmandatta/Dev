pipeline {
    agent any
    stages {
        stage("build"){
            steps {
                echo 'building...'
                sh '''
                    echo "PWD"
                    pwd
                    echo "List all"
                    ls -l
                    echo "Getting the files changed in last commit"
                    git diff --name-only 
                    git status
                '''
            }
        }

        stage("test"){
            steps {
                echo 'testing...'
            }
        }

        stage("deploy"){
            steps {
                echo 'deploying...'
            }
        }
    }
}

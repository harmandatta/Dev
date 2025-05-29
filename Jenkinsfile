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

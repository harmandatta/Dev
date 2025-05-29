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
                    git diff-tree --no-commit-id --name-only -r $github_event_pull_request_merge_commit_sha
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

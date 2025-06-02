pipeline {
    agent any
    stages {
        stage('Checkout Core Code') {
            steps {
                dir('other-repo'){
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[
                            url: "https://github.com/harmandatta/libraries.git"
                        ]]
                    ])   
                }
            }
        }
        
        stage("build"){
            steps {
                echo 'building...'
                sh '''
                    echo "PWD"
                    pwd
                    echo "List all"
                    ls -l

                    ls -l ../
                    
                    echo "Getting the files changed in last commit"
                    # get the 2nd last merged sha
                    second_last_merge_commit_sha=$(git log --merges --format=%H -n 2 | tail -n 1)

                    # Get list of files added or modified in the commit
                    added_or_modified=$(git diff-tree --no-commit-id --name-status -r $second_last_merge_commit_sha $github_event_pull_request_merge_commit_sha | awk '$1 == "A" || $1 == "M" { print $2 }')
                    
                    # Get list of files deleted in the commit
                    deleted=$(git diff-tree --no-commit-id --name-status -r $second_last_merge_commit_sha $github_event_pull_request_merge_commit_sha | awk '$1 == "D" { print $2 }')
                    
                    # Filter out deleted files from added_or_modified
                    changed_list=()
                    
                    for file in $added_or_modified; do
                      if ! echo "$deleted" | grep -qx "$file"; then
                        changed_list+=("$file")
                      else
                        deleted=$(echo "$deleted" | sed "s/${file}//g")
                      fi
                    done

                    for file in "${changed_list[@]}"; do
                      echo "$file"
                    done
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

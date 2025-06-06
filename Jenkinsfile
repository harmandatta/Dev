 pipeline {
    agent any

    environment {
        
        tfVar_location = "primary_region/dev primary_region/sit primary_region/uat primary_region/prod secondary_region/dev secondary_region/sit secondary_region/uat secondary_region/prod"

        //pipeline configuration
        //github endpoints
        commits='/repos/harmandatta/Dev/commits'
        
        
        //regions
        dr_region = "ap-south-1"

        //approval 
        need_approval_primary_region_dev = false
        need_approval_primary_region_sit = false
        need_approval_primary_region_uat = false
        need_approval_primary_region_prod = true
        need_approval_secondary_region_prod = true
    }
    
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

        stage('Pre-requisite'){
            steps{
                script {
                    def TEST_ENV_VAR = sh(script: '''
                    echo "{}" | cat > json
                    key=`jq -r '.action' <<< "$gh_event"`
                    value=`jq -r '.number' <<< "$gh_event"`
                    jq --arg k "$key" --arg v "$value" '. + {($k): $v}' json > tmp && mv tmp json
                    jq --arg v "$value" '. + {"key1": $v}' json > tmp && mv tmp json
                    cat json
                    ''', returnStdout: true).trim()

                    echo "$TEST_ENV_VAR"
                    def json = new groovy.json.JsonSlurper().parseText(TEST_ENV_VAR)
                    env.TEST_ENV_VAR = json.key1 
                }
                
            }
            
        }

        stage('PR open') {
            environment{
                GH_TOKEN = credentials('gh_pat')
            }
            when {
                allOf {
                    expression { env.current_status == 'opened' }
                    expression { env.merged == 'false' }
                }
            }
            steps {
                echo 'step: PR open'
                echo "Checking for *.tfvars files in all the changed file..."
                script {
                    def checkResult = sh(script:'''
                        export GH_PR_NUMBER=`jq -r '.number' <<< "$gh_event"`
                        targets=$tfVar_location
                        files=`gh pr view $GH_PR_NUMBER --json files --jq '.files[].path'`
                        echo "{}" | cat > json
                        for item in $files; do
                            for pattern in $targets; do
                                if echo "$item" | grep -q "$pattern" && echo "$item" | grep -qE "\\.txt$"; then
                                    # Transform pattern (replace '/' with '_') to make key
                                    key=$(echo "$pattern" | sed 's#/#_#g')
                                    jq --arg k "$key" --arg v "$value" '. + {($k): $v}' json > tmp && mv tmp json
                                fi
                            done
                        done
                        cat json
                    ''', returnStdout: true).trim()
                    echo "$checkResult"
                    
                    def json = new groovy.json.JsonSlurper().parseText(checkResult)
                    env.primary_region_dev = json.containsKey(primary_region_dev) ? json.primary_region_dev : ''
                    env.primary_region_sit = json.containsKey(primary_region_sit) ? json.primary_region_sit : ''
                    env.primary_region_uat = json.containsKey(primary_region_uat) ? json.primary_region_uat : ''
                    env.primary_region_prod = json.containsKey(primary_region_prod) ? json.primary_region_prod : ''
                }
            }
        }
        
        stage("build"){
            steps {
                echo 'building...'
                sh '''
                    printenv
                    echo "PWD"
                    pwd
                    echo "List all"
                    ls -l
                    ls -l ./other-repo
                    ls -l ../
                    
                    # echo "Getting the files changed in last commit"
                    # get the 2nd last merged sha
                    # second_last_merge_commit_sha=$(git log --merges --format=%H -n 2 | tail -n 1)

                    # Get list of files added or modified in the commit
                    # added_or_modified=$(git diff-tree --no-commit-id --name-status -r $second_last_merge_commit_sha $github_event_pull_request_merge_commit_sha | awk '$1 == "A" || $1 == "M" { print $2 }')
                    
                    # Get list of files deleted in the commit
                    # deleted=$(git diff-tree --no-commit-id --name-status -r $second_last_merge_commit_sha $github_event_pull_request_merge_commit_sha | awk '$1 == "D" { print $2 }')
                    
                    # Filter out deleted files from added_or_modified
                    # changed_list=()
                    
                    # for file in $added_or_modified; do
                    #   if ! echo "$deleted" | grep -qx "$file"; then
                    #     changed_list+=("$file")
                    #   else
                    #     deleted=$(echo "$deleted" | sed "s/${file}//g")
                    #   fi
                    # done

                    # for file in "${changed_list[@]}"; do
                    #   echo "$file"
                    # done
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

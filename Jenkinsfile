 pipeline {
    agent any

    environment {
        
        tfVar_location = "primary_region/dev primary_region/sit primary_region/uat primary_region/prod"

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
                dir('core_modules'){
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

        // stage('Pre-requisite'){
        //     steps{
        //         script {
        //             def TEST_ENV_VAR = sh(script: '''
        //             echo "{}" | cat > json
        //             key=`jq -r '.action' <<< "$gh_event"`
        //             value=`jq -r '.number' <<< "$gh_event"`
        //             jq --arg k "$key" --arg v "$value" '. + {($k): $v}' json > tmp && mv tmp json
        //             jq --arg v "$value" '. + {"key1": $v}' json > tmp && mv tmp json
        //             cat json
        //             ''', returnStdout: true).trim()

        //             echo "$TEST_ENV_VAR"
        //             def json = new groovy.json.JsonSlurper().parseText(TEST_ENV_VAR)
        //             env.TEST_ENV_VAR = json.key1 
        //         }
                
        //     }
            
        // }

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
                                    jq --arg k "$key" --arg v "$item" '. + {($k): $v}' json > tmp && mv tmp json
                                fi
                            done
                        done
                        cat json
                    ''', returnStdout: true).trim()
                    echo "$checkResult"
                    
                    def json = new groovy.json.JsonSlurper().parseText(checkResult)
                    env.primary_region_dev = json.containsKey('primary_region_dev') ? json.primary_region_dev : ''
                    env.primary_region_sit = json.containsKey('primary_region_sit') ? json.primary_region_sit : ''
                    env.primary_region_uat = json.containsKey('primary_region_uat') ? json.primary_region_uat : ''
                    env.primary_region_prod = json.containsKey('primary_region_prod') ? json.primary_region_prod : ''
                }
            }
        }

        stage('PR closed and merged') {
            environment{
                GH_TOKEN = credentials('gh_pat')
            }
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                }
            }
            steps {
                echo 'step: PR closed and merged'
                echo "Checking for *.tfvars files in all the changed file..."
                
                script {
                    def checkResult = sh(script:'''
                        export GH_PR_NUMBER=`jq -r '.number' <<< "$gh_event"`
                        targets=$tfVar_location
                        files=`gh api /repo/$github_event_repository_full_name/commits/$github_event_pull_request_merge_commit_sha --jq '.files[].filename'`
                        echo "{}" | cat > json
                        for item in $files; do
                            for pattern in $targets; do
                                if echo "$item" | grep -q "$pattern" && echo "$item" | grep -qE "\\.txt$"; then
                                    # Transform pattern (replace '/' with '_') to make key
                                    key=$(echo "$pattern" | sed 's#/#_#g')
                                    jq --arg k "$key" --arg v "$item" '. + {($k): $v}' json > tmp && mv tmp json
                                fi
                            done
                        done
                        cat json
                    ''', returnStdout: true).trim()
                    echo "$checkResult"

                    def json = new groovy.json.JsonSlurper().parseText(checkResult)
                    env.primary_region_dev = json.containsKey('primary_region_dev') ? json.primary_region_dev : ''
                    env.primary_region_sit = json.containsKey('primary_region_sit') ? json.primary_region_sit : ''
                    env.primary_region_uat = json.containsKey('primary_region_uat') ? json.primary_region_uat : ''
                    env.primary_region_prod = json.containsKey('primary_region_prod') ? json.primary_region_prod : ''
                }
            }
        }


        //Terraform Init
        stage('Terraform INIT'){
            when{
                anyOf{
                    expression { env.primary_region_dev != '' }
                    expression { env.primary_region_sit != '' }
                    expression { env.primary_region_uat != '' }
                    expression { env.primary_region_prod != '' }
                }
            }
            steps{
                sh '''
                    cd <enter folder name to enter main>
                    echo 'TF_INIT: executing terraform init'
                    terraform init
                '''
            }
        }
        
        // Plan 
        // Primary Region
        // Dev
        stage('Plan_PrimaryRegion_Dev') {
            when {
                allOf {
                    expression { env.primary_region_dev != '' }
                }
            }
            steps {
                sh '''
                    varFile=$primary_region_dev
                    echo 'TF PLAN: executing terrform plan for $varFile'
                    terraform plan -var-file="$varFile" | tee ${varFile}.plan
                '''
            }
        }

        // Sit
        stage('Plan_PrimaryRegion_Sit') {
            when {
                allOf {
                    expression { env.primary_region_sit != '' }
                }
            }
            steps {
                sh '''
                    varFile=$primary_region_sit
                    echo 'TF PLAN: executing terrform plan for $varFile'
                    terraform plan -var-file="$varFile" | tee ${varFile}.plan
                '''
            }
        }

        // Uat
        stage('Plan_PrimaryRegion_Uat') {
            when {
                allOf {
                    expression { env.primary_region_uat != '' }
                }
            }
            steps {
                sh '''
                    varFile=$primary_region_uat
                    echo 'TF PLAN: executing terrform plan for $varFile'
                    terraform plan -var-file="$varFile" | tee ${varFile}.plan
                '''
            }
        }

        // Prod
        stage('Plan_PrimaryRegion_Prod') {
            when {
                allOf {
                    expression { env.primary_region_prod != '' }
                }
            }
            steps {
                sh '''
                    varFile=$primary_region_prod
                    echo 'TF PLAN: executing terrform plan for $varFile'
                    terraform plan -var-file="$varFile" | tee ${varFile}.plan

                    drFile=${varFile/.tfvar/_dr.tfvar}
                    cp $varFile $drFile
                    echo 'TF PLAN: executing terrform plan for $drFile'
                    terraform plan -var-file="$drFile" -var="region=${dr_region}" | tee ${drFile}.plan

                '''
            }
        }

        // Approval
        // Primary Region
        // Dev
        stage('Approval_PrimaryRegion_Dev') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.primary_region_dev != '' }
                    expression { env.need_approval_primary_region_dev == true }
                }
            }
            steps {
                script {
                    def userInput = input message: "Execute 'terraform apply' with file ${env.primary_region_dev} ?", ok: 'Yes, proceed'
                    env.APPROVAL_RESULT = userInput.toString()
                    echo "Approval result: ${env.approval_primary_region_dev}"
                }
            }
        }

        // Sit
        stage('Approval_PrimaryRegion_Sit') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.primary_region_sit != '' }
                    expression { env.need_approval_primary_region_sit == true }
                }
            }
            steps {
                script {
                    def userInput = input message: "Execute 'terraform apply' with file ${env.primary_region_sit} ?", ok: 'Yes, proceed'
                    env.approval_primary_region_sit = userInput.toString()
                    echo "Approval result: ${env.approval_primary_region_sit}"
                }
            }
        }

        // Uat
        stage('Approval_PrimaryRegion_Uat') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.primary_region_uat != '' }
                    expression { env.need_approval_primary_region_uat == true }
                }
            }
            steps {
                script {
                    def userInput = input message: "Execute 'terraform apply' with file ${env.primary_region_uat} ?", ok: 'Yes, proceed'
                    env.approval_primary_region_uat = userInput.toString()
                    echo "Approval result: ${env.approval_primary_region_uat}"
                }
            }
        }

        // Prod
        stage('Approval_PrimaryRegion_Prod') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.primary_region_prod != '' }
                    expression { env.need_approval_primary_region_prod == true }
                }
            }
            steps {
                script {
                    def userInput = input message: "Execute 'terraform apply' with file ${env.primary_region_prod} ?", ok: 'Yes, proceed'
                    env.approval_primary_region_prod = userInput.toString()
                    echo "Approval result: ${env.approval_primary_region_prod}"
                }
            }
        }

        // Secondary Region
        // Prod
        stage('Approval_SecondaryRegion_Prod') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.secondary_region_prod != '' }
                    expression { env.need_approval_secondary_region_prod == true }
                }
            }
            steps {
                script {
                    def userInput = input message: "Execute 'terraform apply' with file ${env.secondary_region_prod} ?", ok: 'Yes, proceed'
                    env.approval_secondary_region_prod = userInput.toString()
                    echo "Approval result: ${env.approval_secondary_region_prod}"
                }
            }
        }

        // Apply 
        // Primary Region
        // Dev
        stage('Apply_PrimaryRegion_Dev') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.primary_region_dev != '' }
                    expression { env.approval_primary_region_dev == true }
                }
            }
            steps {
                sh '''
                    varFile=$primary_region_dev
                    echo 'TF APPLY: executing terrform apply for $varFile'
                    terraform apply -var-file="$varFile" | tee ${varFile}.apply
                '''
            }
        }

        // Sit
        stage('Apply_PrimaryRegion_Sit') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.primary_region_sit != '' }
                    expression { env.approval_primary_region_sit == true }
                }
            }
            steps {
                sh '''
                    varFile=$primary_region_sit
                    echo 'TF APPLY: executing terrform apply for $varFile'
                    terraform apply -var-file="$varFile" | tee ${varFile}.apply
                '''
            }
        }

        // Uat
        stage('Apply_PrimaryRegion_Uat') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.primary_region_uat != '' }
                    expression { env.approval_primary_region_uat == true }
                }
            }
            steps {
                sh '''
                    varFile=$primary_region_uat
                    echo 'TF APPLY: executing terrform apply for $varFile'
                    terraform apply -var-file="$varFile" | tee ${varFile}.apply
                '''
            }
        }

        // Prod
        stage('Apply_PrimaryRegion_Prod') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.primary_region_prod != '' }
                    expression { env.approval_primary_region_prod == true }
                }
            }
            steps {
                sh '''
                    varFile=$primary_region_prod
                    echo 'TF APPLY: executing terrform apply for $varFile'
                    terraform apply -var-file="$varFile" | tee ${varFile}.apply
                '''
            }
        }

        // Secondary Region
        // Prod
        stage('Apply_SecondaryRegion_Prod') {
            when {
                allOf {
                    expression { env.current_status == 'closed' }
                    expression { env.merged == 'true' }
                    expression { env.secondary_region_prod != '' }
                    expression { env.approval_secondary_region_prod == true }
                }
            }
            steps {
                sh '''
                    varFile=$secondary_region_prod
                    echo 'TF APPLY: executing terrform apply for $varFile'
                    terraform apply -var-file="$varFile" | tee ${varFile}.apply
                '''
            }
        }
        // Stages --- end
    }
}

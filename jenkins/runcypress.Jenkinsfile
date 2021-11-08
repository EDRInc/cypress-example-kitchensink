//To debug Jenkinsfile, make sure you can access: http://jenkinsdev.edrnet.com/cli/
//wget http://jenkinsdev.edrnet.com/jnlpJars/jenkins-cli.jar
//java -jar jenkins-cli.jar -s http://jenkinsdev.edrnet.com:8080/ -auth YOUR_LOGIN:YOUR_PASSWORD declarative-linter < Jenkinsfile

// Some functions definitions
def github_pull_request_send_comment(message, repo_name, pr_num) {
    dir ("${env.WORKSPACE}/resources") {
        sh "set +x; python3 -u github_pull_request_send_comment.py ${repo_name} ${pr_num} '${message}' "
    }
}

def github_status(repo_name, pr_num, state, xcontext, target_url = "", description = "") {
    dir ("${env.WORKSPACE}/resources") {
        sh "set +x; python3 -u github_pull_request_upsert_status.py ${repo_name} ${pr_num} ${state} ${xcontext} '${description}' ${target_url}"
    }
}

// Gets the total from the status file which will determine if the checks were successful or not.
def get_result_total(status_file) {
    dir ("${env.WORKSPACE}") {
        return sh (
            returnStdout: true,
            script: "set +x ./api.sh 'ci.file_total' $status_file"
        ).trim()
    }
}

// Returns path to where we would clone our target repos (C360, AccountsAPI, etc).
def get_target_repo_path() {
    return "${env.WORKSPACE}".minus("${env.JOB_NAME}") + "cache/ci-checks/" + "${env.JOB_NAME}"
}

node ("tf-ubu-jen-slave-00") {
    try {
        def slackmessage, the_user_id, user_message, state1 = "failure";

        wrap([$class: 'BuildUser']) { 
            try {
                // https://wiki.jenkins-ci.org/display/JENKINS/Build+User+Vars+Plugin variables available inside this block
                the_user_id = "${BUILD_USER_ID}"
            } catch (MissingPropertyException err) {
                // Build was started by a another build, which was started by a GitHub hook.
                // So we get the user as a parameter // since this is a one option choice list, only Jenkinsfile can bypass the original value
                the_user_id = params.github_user;
            }
        }
        properties([
            parameters([
                choice(
                    name: 'project',
                    description: 'Name of the project to run cypress against',
                    defaultValue: 'C360\n',
                ),
                string(
                    name: 'branch',
                    defaultValue: 'master',
                    description: ''
                ),
                // string(
                //     name: 'target_branch',
                //     defaultValue: 'housni/DEV-1639',
                //     description: 'The repo to be checked (C360 Accounts API, etc)'
                // ),
                // choice(
                //     name: 'github_user',
                //     choices: '[autodetected]',
                //     description: "Don't worry. We will detect your github user automatically :)",
                // ),
                // string(
                //     name: 'pr_num',
                //     defaultValue: '18239',
                //     description: 'The PR number (in order to send output)',
                // ),
                // string(
                //     name: 'extra_build_info',
                //     defaultValue: '',
                // ),
                
            ])
        ])

        stage('Checkout') {    
            print("-----------------------------------")
            print("Build ID: ${currentBuild.id}")
            print("git_branch_tag_or_commit=${branch}")
            //Set other variables
            sh 'date'
            sh 'pwd'
            sh "echo BUILD_USER_ID=${the_user_id}"
            currentBuild.displayName="#${currentBuild.id}/${the_user_id}/${branch}"
            checkout scm
            sh 'git log -n 1'
            // def target_repo_path = get_target_repo_path()

            // checkout([
            //     $class: 'GitSCM',
            //     branches: [[name: "${target_branch}"]],
            //     doGenerateSubmoduleConfigurations: false,
            //     extensions: [
            //         [$class: 'CloneOption', shallow: true],
            //         [$class: 'RelativeTargetDirectory', relativeTargetDir: "${target_repo_path}"]
            //     ],
            //     submoduleCfg: [],
            //     userRemoteConfigs: [[
            //         credentialsId: 'df4bbaf2-b416-451e-ae34-5f0ca4188215',
            //         url: "https://github.com/EDRInc/${repo_name}"
            //     ]]
            // ])
            sh 'date'
            sh "git log -n 1"
        }

        // Dependencies
        stage('Dependencies') {
            //def target_repo_path = get_target_repo_path()
            sh "pwd"
            sh "npm init"
            
        }

        // stage('Build') {
        //     //def target_repo_path = get_target_repo_path()
        //     sh "npm run build"
            
        // }

        //  stage('Unit Test') {
        //     //def target_repo_path = get_target_repo_path()
        //     sh "npm run test"
            
        // }

         stage('e2e Tests') {
            //def target_repo_path = get_target_repo_path()
            sh "npx cypress run"
            
        }

        stage ("publish reports"){
            def target_repo_path = "a"
            //sh "cp ${target_repo_path}-analysis/coverage.clover.xml ${currentBuild.id}-reports/"
            //sh "cp ${target_repo_path}-analysis/checkstyle.xml ${currentBuild.id}-reports/"
            //sh "cp -R ${target_repo_path}-analysis/coverage ${currentBuild.id}-reports/"
            // def exists = fileExists "${target_repo_path}/${env.BUILD_NUMBER}-analysis/coverage"
            // if (exists) {
            //     sh "cp -R ${target_repo_path}/${env.BUILD_NUMBER}-analysis/coverage ${currentBuild.id}-reports/"
            // } 
            // sh "ls -ltr ${currentBuild.id}-reports"
            // // Replacing all instances of '/mount/' with '/' so that the paths in the checkstyle file match
            // // the path on the host machine instead of the container which is mounted in '/mount/'.
            // sh """#!/usr/bin/env bash
            //         set -xe
            //         sed -i "s%/mount${target_repo_path}%${env.WORKSPACE}/${currentBuild.id}%g" ${currentBuild.id}-reports/checkstyle.xml
            // """
            
            // recordIssues(tools: [checkStyle(pattern: "${currentBuild.id}-reports/checkstyle.xml")])
            // junit testResults: "${currentBuild.id}-reports/*junit.xml"
            // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "${currentBuild.id}-reports/coverage", reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
            // step([$class: 'CloverPublisher', cloverReportDir: "${currentBuild.id}-reports/", cloverReportFileName: 'coverage.clover.xml'])
        }

    } catch(error) {
        throw error

    } finally {
        // Remove all the dirs of the target repo.
        def target_repo_path = "b"
        // dir ("${env.WORKSPACE}/${env.BUILD_ID}") {
        //     deleteDir()
        // }
        // dir ("${target_repo_path}") {
        //     // Find and delete files/dirs that are not the analysis dir.
        //     sh "find * -maxdepth 0 -name '*-analysis' -prune -o -exec rm -rf '{}' ';'"
        // }
        // dir ("${target_repo_path}@tmp") {
        //     deleteDir()
        // }
        // dir ("${target_repo_path}@script") {
        //     deleteDir()
        // }
    }
}

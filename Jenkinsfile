pipeline  {
    agent any
    environment {
        // Jenkins home dir
        HOME_ROOT="/home/lane/work/for-jenkins"

        // About module
        functionName="CUR-JENKINS"
        functionNameU="CUR-JENKINS"
        functionNameJ="CURのJENKINS"
        TEST_HOST="DB_BK"
    }
    stages {
        // Setup
        stage("Setup") {
            steps {
                //各種スクリプトの保存
                sh "echo Setup"
                archiveArtifacts "Jenkinsfile"
            }
        }
        // Code inspection
//         stage("Code Inspection") {
//             steps {
//                 //静的コード解析
// //                 sh "cppcheck --enable=all src/${functionNameJ}/${functionName}/src/ > jenkins/${functionNameJ}/cur_code_inspection.log 2>&1"
// //                 archiveArtifacts "jenkins/${functionNameJ}/cur_code_inspection.log"

//                 //ステップカウント
//                 stepcounter outputFile: "jenkins/${functionNameJ}/stepcount.xls", outputFormat: 'excel', settings: [
//                     [key:'Shell', filePattern: "src/${functionNameJ}/**/*.sh"],
//                     [key:'C', filePattern: "src/${functionNameJ}/**/*.c"],
//                     [key:'C++', filePattern: "src/${functionNameJ}/**/*.cpp"],
//                     [key:'Perl', filePattern: "src/${functionNameJ}/**/*.pl"],
//                     [key:'Java', filePattern: "src/${functionNameJ}/**/*.java"],
//                     [key:'SQL', filePattern: "src/${functionNameJ}/**/*.sql"],
//                     [key:'HTML', filePattern: "src/${functionNameJ}/**/*.html"],
//                     [key:'JS', filePattern: "src/${functionNameJ}/**/*.js"],
//                     [key:'CSS', filePattern: "src/${functionNameJ}/**/*.css"]
//                 ]
//                 archiveArtifacts "jenkins/${functionNameJ}/stepcount.xls"

//                 //事前Checksum
//                 sh "echo zhanglin"
//                 archiveArtifacts "jenkins/${functionNameJ}/checksum_before.log"
//             }
//         }
        // Build
        stage("Build") {
            when {
                expression { params.BUILD }
            }
            steps{
                sh "echo \"Building...\""

                // make
//                 sh "g++ -o src/${functionNameJ}/${functionName}/bin/HelloWorld src/${functionNameJ}/${functionName}/src/HelloWorld.cpp"
            }
        }
        // Deploy
        stage("Deploy") {
            when {
                expression { params.DEPLOY }
            }
            steps{
                sh "echo \"Deploying...\""

                sh "mkdir -p /home/lane/CUR-jenkins"

                // // Deploy
                // sh "rm -rf ansible/roles/deploy${functionNameU}/files"
                // sh "mkdir -p ansible/roles/deploy${functionNameU}/files"
                // sh "cp -pr src/${functionNameJ}/${functionName} ansible/roles/deploy${functionNameU}/files/"
                // sh "ansible-playbook -v ansible/deploy${functionNameU}.yml -i ansible/staging_VM2 -l ${TEST_HOST}"

                // // ansibleツリーへの配信(商用配信のため)
                // sh "cp -prf ansible/* /ansible/"
            }
        }
    }
    post {
        //常に実施
        always {
            sh "date"
            // //事後Checksum
            // sh "find src/${functionNameJ} -type f | xargs cksum > jenkins/${functionNameJ}/checksum_after.log"
            // archiveArtifacts "jenkins/${functionNameJ}/checksum_after.log"
            // sh "diff jenkins/${functionNameJ}/checksum_before.log jenkins/${functionNameJ}/checksum_after.log > jenkins/${functionNameJ}/checksum_diff.log  || true"
            // archiveArtifacts "jenkins/${functionNameJ}/checksum_diff.log"

            //メール送信
            // sendMail(currentBuild.currentResult)
        }
        success { // 成功した場合
            build(
                job: "【新CUR】ansibleアーカイブ",
                wait: false
            )
        }
    }
}

// メールを送信関数
def sendMail(result) {
    def mailRecipients = "jpn_sb_cur_jenkins@hpe.com"
    def jobName = currentBuild.fullDisplayName

    emailext body: '''${SCRIPT, template="groovy-html.template"}''',
        mimeType: 'text/html',
        subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} [${result}]",
        to: "${mailRecipients}",
        replyTo: "${mailRecipients}",
        recipientProviders: [[$class: 'CulpritsRecipientProvider']]
}
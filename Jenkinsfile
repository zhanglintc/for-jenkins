pipeline  {
    agent any
    environment {
        // Jenkins home dir
        HOME_ROOT="/opt2/tmp/jenkins"

        // About module
        functionName="entry_count"
        functionNameU="entry_count"
        functionNameJ="DBエントリ数集計機能"
        TEST_HOST="DB_BK"
    }
    stages {
        // Setup
        stage("Setup") {
            steps {
                //各種スクリプトの保存
                archiveArtifacts "jenkins/${functionNameJ}/Jenkinsfile"
            }
        }
        // Code inspection
        stage("Code Inspection") {
            steps {
                //静的コード解析
//                 sh "cppcheck --enable=all src/${functionNameJ}/${functionName}/src/ > jenkins/${functionNameJ}/cur_code_inspection.log 2>&1"
//                 archiveArtifacts "jenkins/${functionNameJ}/cur_code_inspection.log"

                //ステップカウント
                stepcounter outputFile: "jenkins/${functionNameJ}/stepcount.xls", outputFormat: 'excel', settings: [
                    [key:'Shell', filePattern: "src/${functionNameJ}/**/*.sh"],
                    [key:'C', filePattern: "src/${functionNameJ}/**/*.c"],
                    [key:'C++', filePattern: "src/${functionNameJ}/**/*.cpp"],
                    [key:'Perl', filePattern: "src/${functionNameJ}/**/*.pl"],
                    [key:'Java', filePattern: "src/${functionNameJ}/**/*.java"],
                    [key:'SQL', filePattern: "src/${functionNameJ}/**/*.sql"],
                    [key:'HTML', filePattern: "src/${functionNameJ}/**/*.html"],
                    [key:'JS', filePattern: "src/${functionNameJ}/**/*.js"],
                    [key:'CSS', filePattern: "src/${functionNameJ}/**/*.css"]
                ]
                archiveArtifacts "jenkins/${functionNameJ}/stepcount.xls"

                //事前Checksum
                sh "find src/${functionNameJ} -type f | xargs cksum > jenkins/${functionNameJ}/checksum_before.log"
                archiveArtifacts "jenkins/${functionNameJ}/checksum_before.log"
            }
        }
        // Build
        stage("Build") {
            when {
                expression { params.BUILD_Build }
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
                expression { params.BUILD_Deploy }
            }
            steps{
                sh "echo \"Deploying...\""

                // Deploy
                sh "rm -rf ansible/roles/deploy${functionNameU}/files"
                sh "mkdir -p ansible/roles/deploy${functionNameU}/files"
                sh "cp -pr src/${functionNameJ}/${functionName} ansible/roles/deploy${functionNameU}/files/"
                sh "ansible-playbook -v ansible/deploy${functionNameU}.yml -i ansible/staging_VM2 -l ${TEST_HOST}"

                // ansibleツリーへの配信(商用配信のため)
                sh "cp -prf ansible/* /ansible/"
            }
        }
        // Test UT
        stage("Test UT") {
            environment {
                testType="UT"
            }
            when {
                expression { params.BUILD_UT }
            }
            steps{
                sh "echo \"${testType} Testing...\""

                // 共通セットアップ配信
                sh "ssh ${TEST_HOST} \"mkdir -p ${HOME_ROOT}/common\""
                sh "rsync -r -l -c -v --delete jenkins/common/ ${TEST_HOST}:${HOME_ROOT}/common/"
                sh "ssh ${TEST_HOST} \"chmod 755 ${HOME_ROOT}/common/*.sh\""

                // 試験スクリプト配信・実行
                sh "ssh ${TEST_HOST} \"mkdir -p ${HOME_ROOT}/${functionName}/${testType}/\""
                sh "ssh ${TEST_HOST} \"rm -rf ${HOME_ROOT}/${functionName}/${testType}/*\""
                sh "rsync -r -l -c -v --delete jenkins/${functionNameJ}/${testType}/ ${TEST_HOST}:${HOME_ROOT}/${functionName}/${testType}/"
                sh "ssh ${TEST_HOST} \"chmod 755 ${HOME_ROOT}/${functionName}/${testType}/*.sh\""
                sh "ssh ${TEST_HOST} \"cd ${HOME_ROOT}/${functionName}/${testType} ; sh -x ${HOME_ROOT}/${functionName}/${testType}/${testType}.sh ${testType} > ${HOME_ROOT}/${functionName}/${testType}/${testType}.log 2>&1\""

                // 試験結果取得
                sh "rsync -r -l -c -v --delete ${TEST_HOST}:${HOME_ROOT}/${functionName}/${testType}/ jenkins/${functionNameJ}/${testType}/"
                //sh "grep '^Result:' jenkins/${functionNameJ}/${testType}/${testType}.log > jenkins/${functionNameJ}/${testType}/${testType}_Result.log"
                sh "tar -zcvf jenkins/${functionNameJ}/${testType}.tgz jenkins/${functionNameJ}/${testType}/"
                //archiveArtifacts "jenkins/${functionNameJ}/${testType}/${testType}_Result.log"
                archiveArtifacts "jenkins/${functionNameJ}/${testType}.tgz"
                sh "rm -rf jenkins/${functionNameJ}/${testType}*"
            }
        }

        // Test IT
        stage("Test IT") {
            environment {
                testType="IT"
            }
            when {
                expression { params.BUILD_IT }
            }
            steps{
                sh "echo \"${testType} Testing...\""

                // 共通セットアップ配信
                sh "ssh ${TEST_HOST} \"mkdir -p ${HOME_ROOT}/common\""
                sh "rsync -r -l -c -v --delete jenkins/common/ ${TEST_HOST}:${HOME_ROOT}/common/"
                sh "ssh ${TEST_HOST} \"chmod 755 ${HOME_ROOT}/common/*.sh\""

                // 試験スクリプト配信・実行
                sh "ssh ${TEST_HOST} \"mkdir -p ${HOME_ROOT}/${functionName}/${testType}/\""
                sh "ssh ${TEST_HOST} \"rm -rf ${HOME_ROOT}/${functionName}/${testType}/*\""
                sh "rsync -r -l -c -v --delete jenkins/${functionNameJ}/${testType}/ ${TEST_HOST}:${HOME_ROOT}/${functionName}/${testType}/"
                sh "ssh ${TEST_HOST} \"chmod 755 ${HOME_ROOT}/${functionName}/${testType}/*.sh\""
                sh "ssh ${TEST_HOST} \"cd ${HOME_ROOT}/${functionName}/${testType} ; sh -x ${HOME_ROOT}/${functionName}/${testType}/${testType}.sh ${testType} > ${HOME_ROOT}/${functionName}/${testType}/${testType}.log 2>&1\""

                // 試験結果取得
                sh "rsync -r -l -c -v --delete ${TEST_HOST}:${HOME_ROOT}/${functionName}/${testType}/ jenkins/${functionNameJ}/${testType}/"
                sh "grep '^Result:' jenkins/${functionNameJ}/${testType}/${testType}.log > jenkins/${functionNameJ}/${testType}/${testType}_Result.log"
                sh "tar -zcvf jenkins/${functionNameJ}/${testType}.tgz jenkins/${functionNameJ}/${testType}/"
                archiveArtifacts "jenkins/${functionNameJ}/${testType}/${testType}_Result.log"
                archiveArtifacts "jenkins/${functionNameJ}/${testType}.tgz"
                sh "rm -rf jenkins/${functionNameJ}/${testType}*"
            }
        }

        // Test ST
        stage("Test ST") {
            environment {
                testType="ST"
            }
            when {
                expression { params.BUILD_ST }
            }
            steps{
                sh "echo \"${testType} Testing...\""

                // 共通セットアップ配信
                sh "ssh ${TEST_HOST} \"mkdir -p ${HOME_ROOT}/common\""
                sh "rsync -r -l -c -v --delete jenkins/common/ ${TEST_HOST}:${HOME_ROOT}/common/"
                sh "ssh ${TEST_HOST} \"chmod 755 ${HOME_ROOT}/common/*.sh\""

                // 試験スクリプト配信・実行
                sh "ssh ${TEST_HOST} \"mkdir -p ${HOME_ROOT}/${functionName}/${testType}/\""
                sh "ssh ${TEST_HOST} \"rm -rf ${HOME_ROOT}/${functionName}/${testType}/*\""
                sh "rsync -r -l -c -v --delete jenkins/${functionNameJ}/${testType}/ ${TEST_HOST}:${HOME_ROOT}/${functionName}/${testType}/"
                sh "ssh ${TEST_HOST} \"chmod 755 ${HOME_ROOT}/${functionName}/${testType}/*.sh\""
                sh "ssh ${TEST_HOST} \"cd ${HOME_ROOT}/${functionName}/${testType} ; sh -x ${HOME_ROOT}/${functionName}/${testType}/${testType}.sh ${testType} > ${HOME_ROOT}/${functionName}/${testType}/${testType}.log 2>&1\""

                // 試験結果取得
                sh "rsync -r -l -c -v --delete ${TEST_HOST}:${HOME_ROOT}/${functionName}/${testType}/ jenkins/${functionNameJ}/${testType}/"
                sh "grep '^Result:' jenkins/${functionNameJ}/${testType}/${testType}.log > jenkins/${functionNameJ}/${testType}/${testType}_Result.log"
                sh "tar -zcvf jenkins/${functionNameJ}/${testType}.tgz jenkins/${functionNameJ}/${testType}/"
                archiveArtifacts "jenkins/${functionNameJ}/${testType}/${testType}_Result.log"
                archiveArtifacts "jenkins/${functionNameJ}/${testType}.tgz"
                sh "rm -rf jenkins/${functionNameJ}/${testType}*"
            }
        }
    }
    post {
        //常に実施
        always {
            //事後Checksum
            sh "find src/${functionNameJ} -type f | xargs cksum > jenkins/${functionNameJ}/checksum_after.log"
            archiveArtifacts "jenkins/${functionNameJ}/checksum_after.log"
            sh "diff jenkins/${functionNameJ}/checksum_before.log jenkins/${functionNameJ}/checksum_after.log > jenkins/${functionNameJ}/checksum_diff.log  || true"
            archiveArtifacts "jenkins/${functionNameJ}/checksum_diff.log"

            //メール送信
            sendMail(currentBuild.currentResult)
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
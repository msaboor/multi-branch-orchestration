pipeline {
    agent any

    stages {
        stage('Scan All Branches') {
            steps {
                sh 'git ls-remote --heads origin | cut -f2 | cut -d/ -f3-'
            }
        }

        stage('Scan All Folders in Root') {
            steps {
                sh 'ls -d */'
            }
        }

        stage('Create New Branches') {
            steps {
                script {
                    def branches = sh(returnStdout: true, script: 'git ls-remote --heads origin | cut -f2 | cut -d/ -f3-').trim().split("\\r?\\n")
					def branches = branches.replaceAll("/", "")
                    def folders = sh(returnStdout: true, script: 'ls -d */').trim().split("\\r?\\n")
					def folders =  folders.replaceAll("/", "")
                    for (folder in folders) {
                        if (!branches.contains("origin/${folder}")) {
                            sh "git checkout -b ${folder} develop"
                            sh 'git push -u origin ${folder}'
                        }
                    }
                }
            }
        }

        stage('Merge Code to All Branches') {
            steps {
                script {
                    def branches = sh(returnStdout: true, script: 'git ls-remote --heads origin | cut -f2 | cut -d/ -f3-').trim().split("\\r?\\n")

                    for (branch in branches) {
                        sh "git checkout ${branch}"
                        sh 'git merge develop'
                    }
                }
            }
        }

        stage('Copy Jenkinsfile to Corresponding Branches') {
            steps {
                script {
                    def rootFolder = '.'
                    def branches = sh(returnStdout: true, script: 'git ls-remote --heads origin | cut -f2 | cut -d/ -f3-').trim().split("\\r?\\n")

                    for (branch in branches) {
                        def folderName = branch.substring(branch.lastIndexOf('/') + 1)

                        if (folderName) {
                            def jenkinsfile = readFile("${rootFolder}/${folderName}/Jenkinsfile")

                            sh "git checkout ${branch}"
                            writeFile file: 'Jenkinsfile', text: jenkinsfile

                            sh 'git add Jenkinsfile'
                            sh 'git commit -m "Copy Jenkinsfile"'
                            sh 'git push origin ${branch}'
                        }
                    }
                }
            }
        }
    }
}



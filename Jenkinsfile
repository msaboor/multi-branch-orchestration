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
                   sh '''				   
				   #!/bin/sh
				   echo "Who I'm $SHELL"
				   branches=`git ls-remote --heads origin | cut -f2 | cut -d/ -f3-`
				   echo $branches
				   folders=`ls -d */`
				   echo $folders
				   for folder in "${folders}"; do
					   branch_name="${folder}"
					   trimmedFolder=$(echo "$folder" | sed 's|/||')
					   echo $trimmedFolder
					   branches_space_delimited=$(echo -n "$branches" |  tr '\n' ' ')
					   echo $branches_space_delimited

					   if [ "$branches_space_delimited" != *"$trimmedFolder"* ]; then
							echo "The grep command had output, but it was not equal to $trimmedFolder."
							echo "Output: $grep_result"
							git checkout -b "$trimmedFolder"
							git push -u origin "$trimmedFolder"
							echo "Created and pushed branch: $trimmedFolder"
					   else
							echo "The grep command had no output or found $trimmedFolder."
					  fi
					done
				'''
                }
            }
        }

        stage('Merge Code to All Branches') {
            steps {
                script {
                    def branches = sh(returnStdout: true, script: 'git ls-remote --heads origin | cut -f2 | cut -d/ -f3-').trim().split("\\r?\\n")

                    for (branch in branches) {
                        sh "git checkout ${branch}"
                        sh 'git merge main'
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

// pipeline {
//     agent any

//     stages {
//         stage('Check for CFG Changes') {
//             steps {
//                 script {
//                     withChecks('CFG Change Detection') {
//                         try {
//                             echo "Starting CFG file change detection..."

//                             // Get list of files changed in the last commit
//                             def changedFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim()
//                             def cfgFilesChanged = false

//                             if (changedFiles) {
//                                 cfgFilesChanged = changedFiles.split('\n').any { it.endsWith('.cfg') }
//                             }

//                             if (cfgFilesChanged) {
//                                 echo "Changes detected in .cfg files. Comparison stage will run."
//                                 env.CFG_CHANGES_DETECTED = 'true'
//                             } else {
//                                 echo "No changes detected in .cfg files. Comparison stage will be skipped."
//                                 env.CFG_CHANGES_DETECTED = 'false'
//                             }
//                         } catch (Exception e) {
//                             echo "Error during CFG change detection: ${e.message}"
//                             error "CFG change detection failed: ${e.message}"
//                         }
//                     }
//                 }
//             }
//         }

//         stage('Compare CFG Values') {
//             when {
//                 environment name: 'CFG_CHANGES_DETECTED', value: 'true'
//             }
//             steps {
//                 script {
//                     withChecks('CFG Value Comparison') {
//                         try {
//                             echo "CFG changes detected. Proceeding to compare values..."

//                             // Ensure main branch is available for comparison
//                             sh 'git fetch origin main:refs/remotes/origin/main'
//                             // sh 'git fetch origin main'

//                             def cfgFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim().split('\n')
//                             def mismatchedFiles = []

//                             for (file in cfgFiles) {
//                                 if (file.endsWith('.cfg')) {
//                                     echo "Comparing ${file} with main branch version..."

//                                     def currentContent = readFile(file).trim()
//                                     def expectedContent = sh(script: "git show origin/main:${file}", returnStdout: true).trim()

//                                     if (currentContent != expectedContent) {
//                                         mismatchedFiles << file
//                                     }
//                                 }
//                             }

//                             if (mismatchedFiles) {
//                                 error "CFG value comparison failed. Mismatched files: ${mismatchedFiles.join(', ')}"
//                             } else {
//                                 echo "All .cfg files matched the expected content from 'main' branch."
//                             }

//                         } catch (Exception e) {
//                             echo "Error during CFG value comparison: ${e.message}"
//                             error "CFG value comparison failed: ${e.message}"
//                         }
//                     }
//                 }
//             }
//         }
//     }
// }



pipeline {
    agent any

    environment {
        BRANCH_NAME = "${env.BRANCH_NAME}"
        IS_PR = false
    }

    stages {
        stage('Determine Event Type') {
            steps {
                script {
                    if (env.CHANGE_BRANCH) {
                        IS_PR = true
                        BRANCH_NAME = env.CHANGE_BRANCH
                        echo "📦 Pull Request from branch: ${BRANCH_NAME}"
                    } else if (env.BRANCH_NAME) {
                        echo "🔁 Branch build: ${env.BRANCH_NAME}"
                    } else {
                        error("Unsupported event: Could not detect branch from environment")
                    }

                    def allowedBranches = ['main', 'automation1', 'automation2']
                    if (!allowedBranches.contains(BRANCH_NAME)) {
                        echo "⚠️ Skipping branch ${BRANCH_NAME} as it is not in the allowed list."
                        currentBuild.result = 'SUCCESS'
                        error("Branch ${BRANCH_NAME} is not allowed, skipping the build.")
                    }
                }
            }
        }

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Detect Changed .cfg Files') {
            steps {
                script {
                    withChecks(name: 'CFG Change Detection') {
                        try {
                            echo "📄 Detecting changed .cfg files..."
                            sh "git fetch origin main"

                            // def changedCfgFiles = sh(script: "git diff --name-only origin/main...HEAD | grep '\\.cfg\\$' || true", returnStdout: true).trim()
                            def changedCfgFiles = sh(script: 'git diff --name-only origin/main...HEAD | grep \'\.cfg$\' || true', returnStdout: true).trim()

                            if (!changedCfgFiles) {
                                echo "✅ No .cfg files changed. Skipping comparison stage."
                                currentBuild.result = 'SUCCESS'
                                skipRemainingStages = true
                            } else {
                                env.CFG_CHANGED_FILES = changedCfgFiles.replaceAll('\\n', ',')
                                echo "🔍 Changed CFG files: ${env.CFG_CHANGED_FILES}"
                            }
                        } catch (err) {
                            echo "⚠️ Failed during CFG change detection"
                            error("Change detection failed: ${err}")
                        }
                    }
                }
            }
        }

        stage('Run CFG Comparison') {
            when {
                expression { return !env.skipRemainingStages }
            }
            steps {
                script {
                    withChecks(name: 'CFG Value Comparison') {
                        try {
                            sh '''
                                set -e
                                python3 -m venv venv
                                . venv/bin/activate
                                pip install --upgrade pip
                                pip install -r requirements.txt || true
                                python scripts/compare_cfg.py \
                                    --input_files "${CFG_CHANGED_FILES}" \
                                    --default_file scripts/default.cfg
                            '''
                        } catch (err) {
                            echo "❌ CFG validation failed"
                            error("CFG value comparison failed: ${err}")
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully for branch: ${BRANCH_NAME}"
        }
        failure {
            echo "❌ Pipeline failed for branch: ${BRANCH_NAME}"
        }
    }
}

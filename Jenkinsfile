

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
//                             sh 'git fetch origin main'

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

    stages {
        stage('Check for CFG Changes') {
            steps {
                script {
                    withChecks('CFG Change Detection') {
                        try {
                            echo "Starting CFG file change detection..."

                            // Get list of files changed in the last commit
                            def changedFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim()
                            def cfgFilesChanged = false

                            if (changedFiles) {
                                cfgFilesChanged = changedFiles.split('\n').any { it.endsWith('.cfg') }
                            }

                            if (cfgFilesChanged) {
                                echo "Changes detected in .cfg files. Comparison stage will run."
                                env.CFG_CHANGES_DETECTED = 'true'
                            } else {
                                echo "No changes detected in .cfg files. Comparison stage will be skipped."
                                env.CFG_CHANGES_DETECTED = 'false'
                            }
                        } catch (Exception e) {
                            echo "Error during CFG change detection: ${e.message}"
                            error "CFG change detection failed: ${e.message}"
                        }
                    }
                }
            }
        }

        stage('Compare CFG Values') {
            when {
                environment name: 'CFG_CHANGES_DETECTED', value: 'true'
            }
            steps {
                script {
                    withChecks('CFG Value Comparison') {
                        try {
                            echo "CFG changes detected. Proceeding to compare values..."

                            // Ensure main branch is available for comparison
                            sh 'git fetch origin main:refs/remotes/origin/main'
                            // sh 'git fetch origin main'

                            def cfgFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim().split('\n')
                            def mismatchedFiles = []

                            for (file in cfgFiles) {
                                if (file.endsWith('.cfg')) {
                                    echo "Comparing ${file} with main branch version..."

                                    def currentContent = readFile(file).trim()
                                    def expectedContent = sh(script: "git show origin/main:${file}", returnStdout: true).trim()

                                    if (currentContent != expectedContent) {
                                        mismatchedFiles << file
                                    }
                                }
                            }

                            if (mismatchedFiles) {
                                error "CFG value comparison failed. Mismatched files: ${mismatchedFiles.join(', ')}"
                            } else {
                                echo "All .cfg files matched the expected content from 'main' branch."
                            }

                        } catch (Exception e) {
                            echo "Error during CFG value comparison: ${e.message}"
                            error "CFG value comparison failed: ${e.message}"
                        }
                    }
                }
            }
        }
    }
}
// fixed with origin/main
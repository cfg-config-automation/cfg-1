pipeline {
    agent any

    stages {
        stage('Check for CFG Changes') {
            steps {
                script {
                    // Start GitHub Check for this stage using withChecks
                    withChecks('CFG Change Detection') {
                        try {
                            echo "Starting CFG file change detection..."
                            // Get files changed in the last commit
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
                            error "CFG change detection failed: ${e.message}" // Fails the pipeline and GitHub Check
                        }
                    }
                }
            }
        }

        stage('Compare CFG Values') {
            // This stage only runs if CFG_CHANGES_DETECTED is true
            when {
                environment name: 'CFG_CHANGES_DETECTED', value: 'true'
            }
            steps {
                script {
                    // Start GitHub Check for this stage using withChecks
                    withChecks('CFG Value Comparison') {
                        try {
                            echo "CFG changes detected. Proceeding to compare values."
                            // --- YOUR CUSTOM CFG COMPARISON LOGIC GOES HERE ---
                            //
                            // Example:
                            // def comparisonResult = true // Set to 'false' if a mismatch is found
                            // def comparisonDetails = []
                            //
                            // // Read and compare your .cfg files
                            // if (file('path/to/your.cfg').exists()) {
                            //    def currentContent = readFile('path/to/your.cfg')
                            //    def expectedContent = sh(script: 'git show main:path/to/your.cfg', returnStdout: true).trim()
                            //    if (currentContent != expectedContent) {
                            //        comparisonResult = false
                            //        comparisonDetails << "Mismatch in 'path/to/your.cfg'"
                            //    }
                            // }
                            //
                            // if (!comparisonResult) {
                            //    error "CFG value comparison failed: ${comparisonDetails.join(', ')}" // Fails pipeline and GitHub Check
                            // }
                            echo "--- Placeholder for actual comparison logic ---"
                            echo "All .cfg files matched expected configuration (or no comparison logic defined yet)."

                        } catch (Exception e) {
                            echo "Error during CFG value comparison: ${e.message}"
                            error "CFG value comparison failed: ${e.message}" // Fails the pipeline and GitHub Check
                        }
                    }
                }
            }
        }
    }
}

pipeline {
    agent any
    options { 
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5')
        disableConcurrentBuilds()
        parallelsAlwaysFailFast()
    }
    stages {
        stage("Build info") {
            steps {
                script {
                    def buildCause = currentBuild.getBuildCauses()
                    echo "buildCause: ${buildCause}"

                    C_TEXT = """
                        Job: ${JOB_NAME}
                        Build cause: ${buildCause.shortDescription[0]}
                    """

                    C_PROJECT = GIT_URL.substring(19,GIT_URL.length()-4)
                    echo C_PROJECT
                    echo C_TEXT
                    currentBuild.description = C_TEXT
                }
            }
        }
        stage('Run tests') {
            steps {
                echo "Job: ${JOB_NAME}"
                script {
                    def params = [
                        // [
                        //     $class: 'StringParameterValue',
                        //     name: 'dockerimage_compilers',
                        //     value: "tonlabs/compilers:latest"
                        // ],
                        // [
                        //     $class: 'StringParameterValue',
                        //     name: 'dockerimage_local_node',
                        //     value: "tonlabs/local-node:latest"
                        // ],
                        [
                            $class: 'StringParameterValue',
                            name: 'tonos_cli_branch',
                            value: "${GIT_BRANCH}"
                        ],
                        [
                            $class: 'StringParameterValue',
                            name: 'tonos_cli_commit',
                            value: "${GIT_COMMIT}"
                        ],
                        [
                            $class: 'BooleanParameterValue',
                            name: 'RUN_TESTS_ALL',
                            value: false
                        ],
                        [
                            $class: 'BooleanParameterValue',
                            name: 'RUN_TESTS_TONOS_CLI',
                            value: true
                        ],
                    ] 

                    build job: "Integration/integration-tests/add-tonos-cli", parameters: params
                    // build job: "Integration/integration-tests/master", parameters: params
                }
            }
        }
    }
}

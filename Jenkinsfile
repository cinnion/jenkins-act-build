pipeline {
    agent any
    environment {
        BUILD_DATE_TIME = "${sh(script: 'date +%Y-%m-%d\\ %H:%M:%S\\ %Z', returnStdout: true).trim()}"
        PKG_INIT_VERSION = "${sh(script: 'git describe --tags --always | sed -e s/^v//',  returnStdout: true).trim()}"
    }

    stages {
        stage('Checkout source code.') {
            steps {
                // This forces Jenkins to pull the exact branch and commit
                // from the original 'act' repository that triggered the job.
                checkout scmGit(
                    branches: [[name: "${env.BRANCH_NAME}"]],
                    userRemoteConfigs: [[url: 'https://github.com/nektos/act.git']],
                    extensions: [[ $class: 'LocalBranch', localBranch: "**"]]
                )
            }
        }
        stage('Set build environment variables') {
            steps {
                script {
                    env.PKG_VERSION = sh(script: 'git describe --tags --always | sed -e s/^v//',  returnStdout: true).trim()
                }
            }
        }
        stage ('Checkout remote files.') {
            steps {
                sh "mkdir -p .jenkins"
                dir('.jenkins') {
                    checkout scmGit(
                        branches: [[name: "${env.BRANCH_NAME}"]],
                        userRemoteConfigs: [[ url: "${env.RJPP_SCM_URL}" ]],
                        extensions: [[ $class: 'LocalBranch', localBranch: "**"]]
                    )
                    sh """
                        env | sort
                        set
                        ls -la
                        sed -e "s/{{ PKG_VERSION }}/${env.PKG_VERSION}/" act.spec.template > act.spec
                    """
                }
                sh "touch .xyzzy"
            }
        }

        stage('Build custom container') {
            steps {
                sh '''
                   pwd
                   env | sort
                   set
                '''
                script {
                    customImage = docker.build(
                        "jenkins-act-builder:${env.BUILD_NUMBER}",
                        "-f .jenkins/Dockerfile.rhel9 .jenkins"
                    )
                }
            }
        }

        stage('Build act for the target OSes in parallel.') {
            parallel {
                stage('Build on RHEL 9') {
                    steps {
                        script {
                            customImage.inside {
                                sh '''
                                    pwd
                                    rpm --showrc
                                    make
                                    find . -type f -newer .xyzzy -print
                                '''
                            }
                        }
                    }
                }
            }
        }

        stage('Run tests') {
            steps {
                sh '''
				    ls -laR
                    find . -type f -newer .xyzzy -print
                '''
            }
        }
    }
}
// Local Variables:
// eval: (auto-fill-mode -1)
// End:

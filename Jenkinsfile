pipeline {
    agent any

    stages {
        stage('Checkout source code.') {
            steps {
                // This forces Jenkins to pull the exact branch and commit
                // from the original 'act' repository that triggered the job.
                checkout scmGit(
                    branches: [[name: '**']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/nektos/act.git']]
                )
            }
        }

        stage('Build act for the target OSes in parallel.') {
            parallel {
                stage('Build on RHEL 9') {
                    agent {
                        dockerfile {
                            filename 'Dockerfile.rhel9'
                            label 'act-rocky9'
                        }
                    }
                    steps {
                        sh 'ls -laR'
                        sh 'make test'
                        sh 'make install'
                    }
                }
            }
        }
    }
}
// Local Variables:
// eval: (auto-fill-mode -1)
// End:

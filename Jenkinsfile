@Library('my-shared-library') _

pipeline {

    agent { label 'Node' }

    environment {
        SONAR_HOME = tool "Sonar"
    }

    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag for frontend image')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag for backend image')
    }

    stages {

        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }

        stage("Workspace Cleanup") {
            steps {
                cleanWs()
            }
        }

        stage('Git: Code Checkout') {
            steps {
                git_checkout(
                    "https://github.com/yvardhan8563/Mega_project.git",
                    "main"
                )
            }
        }

        stage("Trivy: Filesystem Scan") {
            steps {
                trivy_scan()
            }
        }

        stage("OWASP: Dependency Check") {
            steps {
                owasp_dependency()
            }
        }

        stage("SonarQube: Code Analysis") {
            steps {
                sonarqube_analysis(
                    "Sonar",
                    "wanderlust",
                    "wanderlust"
                )
            }
        }

        stage("SonarQube: Code Quality Gates") {
            steps {
                sonarqube_code_quality()
            }
        }

        stage('Exporting environment variables') {

            parallel {

                stage("Backend env setup") {
                    steps {
                        dir("Automations") {
                            sh "bash updatebackendnew.sh"
                        }
                    }
                }

                stage("Frontend env setup") {
                    steps {
                        dir("Automations") {
                            sh "bash updatefrontendnew.sh"
                        }
                    }
                }

            }

        }

        stage("Docker: Build Images") {

            steps {

                dir('backend') {
                    docker_build(
                        "wanderlust-backend-beta",
                        "${params.BACKEND_DOCKER_TAG}"
                    )
                }

                dir('frontend') {
                    docker_build(
                        "wanderlust-frontend-beta",
                        "${params.FRONTEND_DOCKER_TAG}"
                    )
                }

            }

        }

        stage("Docker: Push to DockerHub") {

            steps {

                docker_push(
                    "wanderlust-backend-beta",
                    "${params.BACKEND_DOCKER_TAG}",
                    "yvardhan8563"
                )

                docker_push(
                    "wanderlust-frontend-beta",
                    "${params.FRONTEND_DOCKER_TAG}",
                    "yvardhan8563"
                )

            }

        }

    }

    post {

        success {

            archiveArtifacts artifacts: '*.xml', followSymlinks: false

            build job: "Wanderlust-CD", parameters: [

                string(
                    name: 'FRONTEND_DOCKER_TAG',
                    value: "${params.FRONTEND_DOCKER_TAG}"
                ),

                string(
                    name: 'BACKEND_DOCKER_TAG',
                    value: "${params.BACKEND_DOCKER_TAG}"
                )

            ]

        }

    }

}

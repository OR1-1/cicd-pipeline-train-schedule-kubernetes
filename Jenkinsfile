pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "or11/train-schedule"
    }
    // pipelineTriggers {
    //     triggers {
    //         githubPush()
    //     }
    // }
    stages {
        stage('Build') {
            steps {
                input id: 'Iddd', message: 'Deploy to PROD?', ok: 'OK', parameters: [password(defaultValue: 'L0v3n!23T', description: 'Deployment approval password ', name: 'Deployment password')]

                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        // stage('Trigger when tag is created') {
        //     when {
        //         tag "v*"
        //     }
        //     steps {
        //         sh 'oh yeah tags!'
        //     }
        // }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
pipeline {
    agent any

    environment {
        postman_api_key = credentials('postman-api-key')
        collection_id = '10825352-5fcf2dac-164c-4891-b738-126babc795ad'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t learner-api .'
            }
        }

        stage('Test') {
            options {
                timeout(time: 10, unit: 'MINUTES')
            }

            stages {
                stage('Fetch Postman collection') {
                    steps {
                        sh '''curl \\
                            -H "X-API-Key: ${postman_api_key}" \\
                            https://api.getpostman.com/collections/${collection_id} \\
                            > collection.json'''
                    }
                }
                stage('Create Docker network') {
                    steps {
                        sh 'docker network create learner-api || true'
                    }
                }
                stage('Run API server') {
                    steps {
                        sh '''docker run \\
                            --rm \\
                            -p 3000:3000 \\
                            --name learner-api-server \\
                            --network learner-api \\
                            --detach \\
                            learner-api'''          
                    }
                }
                stage('Run Postman tests') {
                    steps {

                        sh '''docker run \\
                            -v $WORKSPACE:/etc/newman \\
                            --rm \\
                            --network learner-api \\
                            postman/newman \\
                            run collection.json \\
                            --env-var url=http://learner-api-server:3000 \\
                            --reporters cli,junit \\
                            --reporter-junit-export newman/report.xml'''
                    }
                }
            }

            post {
                always {
                    sh 'docker kill learner-api-server || true'
                    sh 'docker network rm learner-api || true'
                }
            }
        }
    }
    
    post {
        always {
            junit 'newman/report.xml'
        }
    }
}
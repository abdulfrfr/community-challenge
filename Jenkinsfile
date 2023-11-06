pipeline {
    agent any

    environment {
        def buildNumber = env.BUILD_NUMBER.toInteger()
        def version = "1.0.${buildNumber}"
        echo "Setting application version to: ${version}"
        // Set the version as an environment variable for your application
        env.APP_VERSION = version
    }

    environment {
        // Define global environment variables
        ZONE_ID = credentials('ZONE_ID')
        CF_API_KEY = credentials('CF_API_KEY')
        CF_API_EMAIL = credentials('CF_API_EMAIL')
        VUE_APP_PROXY_URL = credentials('VUE_APP_PROXY_URL')

        //DEfine this environmental variable to verison to my application's docker image
        def buildNumber = env.BUILD_NUMBER.toInteger()
        def version = "1.0.${buildNumber}"
        echo "Setting application version to: ${version}"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    
                        git branch: 'main', url: 'https://github.com/abdulfrfr/community-challenge.git'
                    
                }
            }
        }

        stage('Build and run Docker Images locally') {
            steps {
                script {
                // here i will build my docker images and run them locally for testing...
                sh """

                        export backend_endpoint=http://localhost:5000

                        docker build -t vue-app .

                        cd backend
                        docker build -t python .

                        cd ..

                        docker compose -f docker-compose.yaml up 

                    """
                }
            }
        }

        stage('Testing frontend application locally') {
            steps {
                script {

                    //here i will test and send requests to my running fontend docker container of my 
                    //applciation to test if they are configured properly and if they are up and running
                    sh """

                        # Make API requests using cURL
                        curl -X GET http://localhost:8080


                        # Check the HTTP status code (response code)
                        if [ $? -eq 0 ]; then
                            # If the exit status is 0, the cURL request was successful, and the HTTP status code is accessible in the response headers.
                            HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "http://localhost:8080")

                            if [ "$HTTP_STATUS" -eq 200 ]; then
                                echo "HTTP Status Code: $HTTP_STATUS (OK)"
                            else
                                echo "HTTP Status Code: $HTTP_STATUS (Not OK)"
                                exit 1
                            fi
                        else
                            echo "HTTP Request Failed"
                            exit 1
                        fi

                    """

                    
                }
            }
        }

        
        stage('Testing python-proxy application locally') {
            steps {
                script {

                    //here i will test and send requests to my running backend docker container of my 
                    //applciation to test if they are configured properly and if they are up and running
                    sh """

                        # Make API requests using cURL
                        curl -X GET http://localhost:5000


                        # Check the HTTP status code (response code)
                        if [ $? -eq 0 ]; then
                            # If the exit status is 0, the cURL request was successful, and the HTTP status code is accessible in the response headers.
                            HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "http://localhost:5000")

                            if [ "$HTTP_STATUS" -eq 200 ]; then
                                echo "HTTP Status Code: $HTTP_STATUS (OK)"
                            else
                                echo "HTTP Status Code: $HTTP_STATUS (Not OK)"
                                exit 1
                            fi
                        else
                            echo "HTTP Request Failed"
                            exit 1
                        fi

                    """

                    
                }
            }
        }
    }


        stage('Build Docker Images for deployment') {
            steps {
                script {
                // here i will build my docker images and run them locally for testing...
                sh """

                        export backend_endpoint=http://aacf724f1ada5446cb70e439e677aa09-761205241.us-east-1.elb.amazonaws.com/proxy/

                        docker build -t vue-app:$version .

                        cd backend
                        docker build -t python:$version .

                        cd ..

                    """
                }
            }
        }

        stage('Deploy to Amazon ECR...') {
            steps {
                script {

                    //after building my docker images, i will then tag and push them to my Amazon ECR repro


                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_ACCESS']]) {
                    sh """
                            aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/l1z2o5a3
                    """
                }

                    sh """

                            docker tag python:$version public.ecr.aws/l1z2o5a3/python-proxy:$version
                            docker push public.ecr.aws/l1z2o5a3/python-proxy:$version


                            docker tag vue-app:$version public.ecr.aws/l1z2o5a3/vue-app:$version
                            docker push public.ecr.aws/l1z2o5a3/vue-app:$version

                    """
                }
            }
        }


    post {
        success {
            script {

                //if this piplein is a success i want to then deploy my docker images to my Amazon EKS cluster which i have already
                //configured and running...

                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_ACCESS']]) {
                    sh """
                            aws eks update-kubeconfig --name my-eks-cluster --region us-east-1

                            helm upgrade --install ingress-nginx ingress-nginx   --repo https://kubernetes.github.io/ingress-nginx

                    """
                }

                    sh """

                            export version=$version

                            kubectl apply -f /k8s/eks_deployments.yaml

                            kubectl apply -f ingress-service.yaml

                    """

                    sh """

                        # Make API requests using cURL
                        curl -X GET http://aacf724f1ada5446cb70e439e677aa09-761205241.us-east-1.elb.amazonaws.com/


                        # Check the HTTP status code (response code)
                        if [ $? -eq 0 ]; then
                            # If the exit status is 0, the cURL request was successful, and the HTTP status code is accessible in the response headers.
                            HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "http://aacf724f1ada5446cb70e439e677aa09-761205241.us-east-1.elb.amazonaws.com/")

                            if [ "$HTTP_STATUS" -eq 200 ]; then
                                echo "HTTP Status Code: $HTTP_STATUS (OK)"
                            else
                                echo "HTTP Status Code: $HTTP_STATUS (Not OK)"
                                exit 1
                            fi
                        else
                            echo "HTTP Request Failed"
                            exit 1
                        fi

                    """

                    sh """

                        # Make API requests using cURL
                        curl -X GET http://aacf724f1ada5446cb70e439e677aa09-761205241.us-east-1.elb.amazonaws.com/proxy/


                        # Check the HTTP status code (response code)
                        if [ $? -eq 0 ]; then
                            # If the exit status is 0, the cURL request was successful, and the HTTP status code is accessible in the response headers.
                            HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "http://aacf724f1ada5446cb70e439e677aa09-761205241.us-east-1.elb.amazonaws.com/")

                            if [ "$HTTP_STATUS" -eq 200 ]; then
                                echo "HTTP Status Code: $HTTP_STATUS (OK)"
                            else
                                echo "HTTP Status Code: $HTTP_STATUS (Not OK)"
                                exit 1
                            fi
                        else
                            echo "HTTP Request Failed"
                            exit 1
                        fi

                    """
                }
            }
        }
    }
        }
        failure {
            // Actions to take when the pipeline fails.

            sh "echo 'Pipeline Failed"
        }
        always {
            // Actions to take regardless of the pipeline result.
            def buildNumber = env.BUILD_NUMBER.toInteger()
            echo "This is pipeline run number: ${buildNumber}"
        }
    }
}

@Library("Shared") _
pipeline{
    
    agent { label "vinod"}
    stages{
        stage("hello"){
            steps{
                script{
                    hello()
                }
            }
        }
        stage("code"){
            steps{
                script{
                    echo "this is cloning the code enjoy"
                    clone("https://github.com/harshit805/django-notes-app.git","main")
                    echo "code cloning successful"
                }
            }
        }
        stage("build"){
            steps{
                  echo "this is building the code"
        sh "whoami"
        script{
            docker_build("notes-app", "latest", "harrycreations")
        }
            }
        }
        stage("test"){
            steps{
                echo "this is testing the code"
            }
        }
          stage("Push") {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'dockerhubcreds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                // Perform docker login manually
                sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                // Tag and push
                sh "docker image tag notes-app:latest harrycreations/notes-app:latest"
                sh "docker push harrycreations/notes-app:latest"
            }
        }
    }
}
        stage("deploy") {
    steps {
        echo "this is deploying the code"

        // Start MySQL container
        sh '''
        docker run -d --name test_db --network my_network \
            -e MYSQL_ROOT_PASSWORD=rootpass \
            -e MYSQL_DATABASE=mydb \
            -e MYSQL_USER=myuser \
            -e MYSQL_PASSWORD=mypassword \
            mysql:8
        '''

        // Wait for MySQL to be ready (polling every 2s for up to 60s)
        sh '''
        echo "Waiting for MySQL to be ready..."
        for i in {1..30}; do
            if docker exec test_db mysqladmin ping -hmysql --silent; then
                echo "MySQL is ready!"
                break
            fi
            echo "Still waiting for MySQL ($i)..."
            sleep 2
        done
        '''

        // Now start Django container
        sh '''
        docker run -d --name django_app --network my_network -p 8000:8000 \
            -e DB_HOST=test_db \
            -e DB_NAME=mydb \
            -e DB_USER=myuser \
            -e DB_PASSWORD=mypassword \
            notes-app:latest
        '''
            }
        }
    }
}

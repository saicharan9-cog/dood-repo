pipeline{
    agent any
    environment {
        IMAGE = 'saicharan209/dood-flask'   // change to your namespace/repo
        TAG   = "${env.BUILD_NUMBER}"
    }
    stages{
        stage ('Checkout') {
            steps{
                git branch: 'main', credentialsId: 'github-pat', 
                url: 'https://github.com/saicharan9-cog/dood-repo.git'
                echo 'checking out from private repo'
            }
        }
        stage ('Build') {
            steps{
                sh 'docker build -t "$IMAGE:$TAG" -t "$IMAGE:latest" .'
                echo 'This is BUild step'
            }
        }
        stage ('Push') {
            steps{
                echo 'This is Push step'
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PWD')]) {
                sh 'echo "$DOCKERHUB_PWD" | docker login -u "$DOCKERHUB_USER" --password-stdin'
                sh 'docker push "$IMAGE:$TAG"'
                sh 'docker push "$IMAGE:latest"'
                }
            }
        }
        stage ('Deloy') {
            steps{
                echo 'This is Deploy step'
                sh 'docker pull "$IMAGE:$TAG"'
                sh 'docker rm -f dood-flask || true'
                sh 'docker run -d --name dood-flask -p 5000:5000 "$IMAGE:$TAG"'
                sh '''
                      cat > deploy-info-$BUILD_NUMBER.txt <<EOF
                build: $BUILD_NUMBER
                image: $IMAGE:$TAG
                commit: ${GIT_COMMIT}
                branch: $GIT_BRANCH
                time: $(date -u +"%Y-%m-%dT%H:%M:%SZ")
                url: $BUILD_URL
                EOF
                    '''

               archiveArtifacts artifacts: "deploy-info-${BUILD_NUMBER}.txt", fingerprint: true
            }
        }
        stage ('Test') {
            steps{
                sh 'sleep 2; curl -s http://172.198.78.185:5000 || true'
            }
        }
        stage('cleanup') {
            steps {
              cleanWs()
            }
        }
        
    }
    post {
      success { echo "Build ${env.BUILD_NUMBER} succeeded" }
      failure { echo "Build ${env.BUILD_NUMBER} failed" }
      always  { echo "Build ${env.BUILD_NUMBER} finished" }
   }
}

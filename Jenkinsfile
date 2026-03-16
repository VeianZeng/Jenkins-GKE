pipeline {
    agent any
    
    // 监听 GitHub Push 事件
    triggers {
        githubPush()
    }
    
    environment {
        // --- 请修改为你的 GCP 项目信息 ---
        PROJECT_ID = 'wdtest-001'
        REGION     = 'asia-east2-a'
        REPO_NAME  = 'vija-images' 
        
        // --- 保持默认 ---
        IMAGE_NAME = 'hello-app'
        AR_URL     = "${REGION}-docker.pkg.dev"
        FULL_IMAGE_PATH = "${AR_URL}/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
    }

    stages {
        stage('1. 拉取代码') {
            steps {
                // 将 URL 换成你刚才创建的 GitHub 仓库地址
                git credentialsId: 'github-auth', url: 'https://github.com/VeianZeng/Jenkins-GKE.git', branch: 'main'
            }
        }

        stage('2. 构建并推送') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GCP_KEY')]) {
                    sh '''
                        set +x
                        gcloud auth activate-service-account --key-file="$GCP_KEY" --quiet
                        gcloud auth configure-docker ${AR_URL} --quiet
                        docker build -t ${FULL_IMAGE_PATH} .
                        docker push ${FULL_IMAGE_PATH}
                    '''
                }
            }
        }

        stage('3. 部署到 GKE') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GCP_KEY')]) {
                    sh '''
                        set +x
                        gcloud container clusters get-credentials vija-hk-cluster --zone ${REGION} --project ${PROJECT_ID} --quiet
                        # 核心逻辑：用刚生成的镜像地址替换 YAML 里的占位符
                        sed -i "s|IMAGE_PLACEHOLDER|${FULL_IMAGE_PATH}|g" k8s-deploy.yaml
                        kubectl apply -f k8s-deploy.yaml
                    '''
                }
            }
        }
    }
}

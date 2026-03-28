pipeline {
    agent {
        label 'test' // รันบน Mac ของส้ม
    }

    environment {
        APP_NAME    = 'my-nginx-web'
        IMAGE_TAG   = "${BUILD_NUMBER}"
        NAMESPACE   = 'jenkins' // เพิ่มตัวแปร namespace ไว้เลย
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // สร้าง Image ปกติ
                    sh "docker build -t ${APP_NAME}:${IMAGE_TAG} -t ${APP_NAME}:latest ."
                    
                    // สั่งให้ KIND (ถ้าส้มใช้) รู้จัก Image นี้
                    sh "kind load docker-image ${APP_NAME}:${IMAGE_TAG}" 
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // ใส่ -n ${NAMESPACE} ทุกบรรทัด
                    sh "kubectl apply -f k8s/deployment.yaml -n ${NAMESPACE}"
                    sh "kubectl apply -f k8s/service.yaml -n ${NAMESPACE}"
                    sh "kubectl apply -f k8s/ingress.yaml -n ${NAMESPACE}"
                    
                    // อัปเดต Image ให้เป็นเวอร์ชันใหม่ล่าสุด (BUILD_NUMBER)
                    // **เช็คชื่อ deployment/nginx-deployment ให้ตรงกับในไฟล์ yaml ด้วยนะ**
                    sh "kubectl set image deployment/nginx-deployment nginx-container=${APP_NAME}:${IMAGE_TAG} -n ${NAMESPACE}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // ตรวจสอบสถานะการรัน
                    sh "kubectl rollout status deployment/nginx-deployment -n ${NAMESPACE} --timeout=120s"
                    sh "kubectl get pods -n ${NAMESPACE} -l app=my-nginx"
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful! Access at http://localhost:8081"
        }
    }
}
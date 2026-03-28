pipeline {
    agent {
        label 'test' // มั่นใจว่า Node ใน Jenkins ชื่อ 'test'
    }

    environment {
        APP_NAME    = 'my-nginx-web'
        IMAGE_TAG   = "${BUILD_NUMBER}" // ใช้เลข Build ป้องกัน Image ซ้ำ
        NAMESPACE   = 'jenkins'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Load Image') {
            steps {
                script {
                    // 1. Build Image ในเครื่อง Mac
                    sh "docker build -t ${APP_NAME}:${IMAGE_TAG} ."
                    
                    // 2. โหลด Image เข้า KIND Cluster (ถ้าใช้ Docker Desktop ปกติ ให้คอมเมนต์บรรทัดนี้ทิ้ง)
                    sh "kind load docker-image ${APP_NAME}:${IMAGE_TAG} --name my-cluster"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // สั่ง Apply ไฟล์ทั้งหมดใน Namespace 'jenkins'
                    sh "kubectl apply -f k8s/deployment.yaml -n ${NAMESPACE}"
                    sh "kubectl apply -f k8s/service.yaml -n ${NAMESPACE}"
                    sh "kubectl apply -f k8s/ingress.yaml -n ${NAMESPACE}"
                    
                    // อัปเดต Deployment ให้ใช้ Image ตัวล่าสุด
                    // **ส้มเช็คชื่อ nginx-deployment กับ nginx-container ในไฟล์ yaml ให้ตรงกันนะ**
                    sh "kubectl set image deployment/nginx-deployment nginx-container=${APP_NAME}:${IMAGE_TAG} -n ${NAMESPACE}"
                }
            }
        }

        stage('Verify') {
            steps {
                script {
                    // รอจนกว่าจะรันสำเร็จ
                    sh "kubectl rollout status deployment/nginx-deployment -n ${NAMESPACE} --timeout=60s"
                    // แสดงรายชื่อ Service เพื่อให้ส้มเอาไปทำ Port-forward ต่อได้ง่ายๆ
                    sh "kubectl get svc -n ${NAMESPACE}"
                }
            }
        }
    }

    post {
        success {
            echo "สำเร็จแล้ว! ลองรันคำสั่ง port-forward เพื่อเข้าชมเว็บได้เลย"
        }
    }
}
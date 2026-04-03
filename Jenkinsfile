pipeline {
    agent any
    
    environment {
        // ชื่อของ Docker Image ที่คุณใช้
        DOCKER_IMAGE = 'kittiphong57412003/react-app'
        
        // เราจะใช้ Build Number ของ Jenkins มารันเป็นแท็กเวอร์ชัน (ได้เป็น v1, v2, v3...)
        // หรือถ้าอยากฟิกซ์เลขเอง ก็แก้เป็น DOCKER_TAG = 'v0.0.1' แบบเดิมได้ครับ
        DOCKER_TAG = "v${env.BUILD_NUMBER}" 
        
        // รหัส ID ของ Credential ใน Jenkins ที่ใช้บอก Username/Password สำหรับลอกอิน Docker Hub
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' 
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                // ดึงโค้ดล่าสุดลงมาจาก Repository ของคุณโดยตรง
                git branch: 'main', url: 'https://github.com/kittiphong-dev/react-app-devops.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "กำลัง Build Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    // สั่ง Build ตัว image ตาม Dockerfile
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    
                    // แท็กเวอร์ชันนี้ให้เป็น latest ด้วย 
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    // ขั้นตอนนี้จะดึง Username/Password จาก Jenkins Credentials มาเพื่อล็อกอิน
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // สั่งล็อกอิน Docker
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        
                        echo "กำลัง Push Image ขึ้น Docker Hub..."
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
    }
    
    // สิ่งที่ต้องทำหลังจากผ่านทุก Stage เสร็จ (หรือตอนที่พัง)
    post {
        always {
            script {
                // ล้าง Image ที่เพิ่งสร้างทิ้งบนเครื่องเซิร์ฟเวอร์ Jenkins เพื่อป้องกันดิสก์เต็ม
                echo "Cleaning up local docker images..."
                sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
                sh "docker rmi ${DOCKER_IMAGE}:latest || true"
                // เพื่อความปลอดภัย: ออกจากระบบ Docker ด้วย
                sh "docker logout"
            }
        }
        success {
            echo "✅ Pipeline ผ่านฉลุย! Push Image เรียบร้อยแล้ว"
        }
        failure {
            echo "❌ Pipeline ล้มเหลว โปรดเช็คตาราง Log สำหรับข้อบกพร่องเพิ่มเติม"
        }
    }
}

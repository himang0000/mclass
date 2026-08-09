pipeline {
    agent any // 어떤 에이전트(실행서버)에서든 실행 가능

    tools { //tools : jenkins tools에 등록된 도구 사용
        maven 'maven 3.9.12' //jenkins tools에 등록한 이름과 정확히 일치해야 함.
    }

    enviroment {
        // 배포에 필요한 변수 설정               
        DOCKER_IMAGE = "demo-app" // 도커 이미지
        CONTAINER_NAME = "springboot-container" //도커 컨테이너
        JAR_FILE_NAME = "app.jar" // 복사할 jar 파일 이름
        PORT = "8081" // 컨테이너와 연결할 포트

        REMOTE_USER ="ec2-user"  //원격(spring) 서버 사용자 이름
        REMOTE_HOST ="3.106.18.142" //원격(spring) 서버 public ip

        REMOTE_DIR ="/home/ec2-user/deploy" //원격 서버 배포
        SSH_CREDENTIALS_ID = "edc13376-66f0-46ec-a283-12e11b59184c" // RSA Credentials
    }

    stages { // stages : 실제 자동 빌드를 수행하는 단계 정의
        stage ('Git Checkout') { // 수행 단계 구분
            steps { // 실제 수행할 명령어 정의
                // jenkins가 연결된 Git 저장소에서 최신 코드 체크 아웃
                checkout scm
            }
        } 

        stage('Maven Build') {
            steps {
                // 테스트는 건너뛰고 maven 빌드
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Prepare Jar') {
            steps {
                // 빌드 결과물을 app.jar 라는 고정이름으로 복사
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }

        stage('Copy to Remote Server') {
            steps {
                //원격 명령 실행
                sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
                    // 배포 디렉토리 생성 (없으면)
                    sh "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} \"mkdir -p ${REMOTE_DIR}\""
                    // JAR과 Dockerfile을 원격 서버로 복사
                    sh "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${JAR_FILE_NAME} Dockerfile ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
                }
            }
        }
    }
}
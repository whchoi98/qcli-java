# Q CLI를 활용한 Graviton 전환 프로젝트

이 저장소는 Amazon Q CLI를 활용하여 Java 애플리케이션을 x86에서 AWS Graviton (ARM64) 아키텍처로 전환하는 과정을 보여주는 데모 프로젝트입니다.

## 📋 프로젝트 개요

이 프로젝트는 다음과 같은 구성요소를 포함합니다:
- **Java 챗봇 애플리케이션**: JNA를 사용하여 네이티브 C++ 라이브러리와 상호작용하는 웹 서버
- **C++ 네이티브 라이브러리**: 챗봇 기능을 제공하는 공유 라이브러리
- **AWS 배포 설정**: 다양한 AWS 서비스에 배포하기 위한 설정 파일들

## 🏗️ 프로젝트 구조

```
qcli-java/
├── README.md
├── java-demo.zip                    # 원본 압축 파일
└── java-demo/
    ├── gdd-chatbot-main/           # C++ 챗봇 소스 코드
    │   ├── chatbot.cpp
    │   ├── CMakeLists.txt
    │   ├── README.md
    │   └── commands.txt
    └── java/                       # Java 애플리케이션
        ├── code/                   # Java 소스 코드
        │   ├── pom.xml            # Maven 설정
        │   └── src/main/java/chatbot/Main.java
        ├── chatbot-libs/          # 네이티브 라이브러리
        │   └── libchatbot-amd64.so
        ├── manifests/             # Kubernetes 배포 매니페스트
        │   └── deploy.yaml
        ├── Dockerfile             # 컨테이너 이미지 빌드
        ├── buildspec.yaml         # CodeBuild 기본 빌드 스펙
        ├── buildspec.ami.yaml     # AMI 빌드 스펙
        ├── buildspec.asg.yaml     # Auto Scaling Group 빌드 스펙
        ├── buildspec.eks.yaml     # EKS 빌드 스펙
        ├── buildspec.manifest.yaml # 매니페스트 빌드 스펙
        └── .gitignore
```

## 🚀 애플리케이션 특징

### Java 챗봇 웹 서버
- **포트**: 8081
- **프레임워크**: Java 21 + Maven
- **주요 의존성**:
  - JNA (Java Native Access) 5.14.0
  - JSON 처리 라이브러리 20240303
- **엔드포인트**:
  - `GET /`: 웰컴 메시지
  - `POST /generateResponse`: 챗봇 응답 생성

### 네이티브 라이브러리 통합
- C++ 라이브러리와 JNA를 통한 상호작용
- 현재 AMD64 아키텍처용 라이브러리 포함 (`libchatbot-amd64.so`)

## 🔄 Graviton 전환 과정

### 1. 현재 상태 분석
- **아키텍처**: x86_64 (AMD64)
- **네이티브 라이브러리**: `libchatbot-amd64.so`
- **베이스 이미지**: `public.ecr.aws/lts/ubuntu:22.04_stable`

### 2. Graviton 전환을 위한 주요 변경사항

#### 네이티브 라이브러리 재컴파일
```bash
# C++ 라이브러리를 ARM64용으로 재컴파일 필요
cd java-demo/gdd-chatbot-main/
cmake -DCMAKE_BUILD_TYPE=Release .
make
```

#### Docker 이미지 멀티 아키텍처 빌드
```dockerfile
# Dockerfile에서 ARM64 지원 추가
FROM --platform=$BUILDPLATFORM public.ecr.aws/lts/ubuntu:22.04_stable
```

#### 라이브러리 경로 수정
```java
// Main.java에서 아키텍처별 라이브러리 로딩
String arch = System.getProperty("os.arch");
String libName = arch.equals("aarch64") ? "libchatbot-arm64.so" : "libchatbot-amd64.so";
chatbotlib = (ChatbotLib) Native.load(libName, ChatbotLib.class);
```

## 🛠️ Q CLI 활용 방법

### 1. 코드 분석 및 최적화
```bash
# Q CLI를 사용하여 코드 분석
q chat "이 Java 애플리케이션을 Graviton으로 전환하는 방법을 알려줘"
```

### 2. 빌드 스크립트 생성
```bash
# 멀티 아키텍처 빌드 스크립트 생성 요청
q chat "Docker 멀티 아키텍처 빌드를 위한 buildspec.yaml을 작성해줘"
```

### 3. 성능 최적화 가이드
```bash
# Graviton 최적화 방법 문의
q chat "Java 애플리케이션의 Graviton 성능 최적화 방법을 알려줘"
```

## 📦 배포 옵션

이 프로젝트는 다양한 AWS 서비스에 배포할 수 있도록 설정되어 있습니다:

### 1. Amazon EKS
```bash
kubectl apply -f java-demo/java/manifests/deploy.yaml
```

### 2. AWS CodeBuild
- `buildspec.yaml`: 기본 빌드
- `buildspec.eks.yaml`: EKS 배포
- `buildspec.ami.yaml`: AMI 생성
- `buildspec.asg.yaml`: Auto Scaling Group 배포

### 3. 컨테이너 배포
```bash
docker build -t chatbot-java .
docker run -p 8081:8081 chatbot-java
```

## 🎯 Graviton 전환의 이점

1. **비용 절감**: 최대 20% 가격 성능 향상
2. **성능 향상**: ARM64 아키텍처의 효율성
3. **에너지 효율성**: 더 낮은 전력 소비
4. **AWS 네이티브**: AWS에서 설계된 프로세서

## 🔧 개발 환경 설정

### 필수 요구사항
- Java 21 (Amazon Corretto 권장)
- Maven 3.6+
- Docker
- AWS CLI
- Q CLI

### 로컬 실행
```bash
cd java-demo/java/code
mvn clean package
mvn exec:java -Dexec.mainClass=chatbot.Main
```

### 테스트
```bash
# 웰컴 메시지 확인
curl http://localhost:8081/

# 챗봇 응답 테스트
curl -X POST http://localhost:8081/generateResponse \
  -H "Content-Type: application/json" \
  -d '{"Prompt": "Hello", "Tokens": 100}'
```

## 📚 참고 자료

- [AWS Graviton 프로세서](https://aws.amazon.com/ec2/graviton/)
- [Java on Graviton 최적화 가이드](https://github.com/aws/aws-graviton-getting-started/blob/main/java.md)
- [Amazon Q CLI 문서](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/cli.html)
- [JNA (Java Native Access)](https://github.com/java-native-access/jna)

## 🤝 기여하기

1. 이 저장소를 포크합니다
2. 기능 브랜치를 생성합니다 (`git checkout -b feature/amazing-feature`)
3. 변경사항을 커밋합니다 (`git commit -m 'Add some amazing feature'`)
4. 브랜치에 푸시합니다 (`git push origin feature/amazing-feature`)
5. Pull Request를 생성합니다

## 📄 라이선스

이 프로젝트는 MIT 라이선스 하에 배포됩니다. 자세한 내용은 `LICENSE` 파일을 참조하세요.

---

**Amazon Q CLI를 활용하여 더 효율적인 Graviton 전환을 경험해보세요!** 🚀

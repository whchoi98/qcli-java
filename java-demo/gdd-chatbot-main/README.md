# GDD 챗봇 - C++ 네이티브 라이브러리

이 프로젝트는 Graviton Developer Day를 위한 C++ 챗봇 네이티브 라이브러리입니다. Mistral-7B 모델을 사용하여 챗봇 기능을 제공하며, Java 애플리케이션에서 JNA(Java Native Access)를 통해 호출할 수 있는 공유 라이브러리를 생성합니다.

## 📋 프로젝트 개요

이 C++ 라이브러리는 다음과 같은 기능을 제공합니다:
- **Mistral-7B 모델 기반 텍스트 생성**: Hugging Face에서 호스팅되는 GGUF 형식의 모델 사용
- **멀티스레딩 지원**: 동시 요청 처리를 위한 뮤텍스 기반 스레드 안전성
- **JNA 호환 인터페이스**: Java 애플리케이션에서 직접 호출 가능한 C 스타일 API
- **멀티 아키텍처 지원**: AMD64 및 ARM64(Graviton) 아키텍처 모두 지원

## 🏗️ 프로젝트 구조

```
gdd-chatbot-main/
├── README.md           # 이 파일
├── chatbot.cpp         # 메인 챗봇 구현
├── CMakeLists.txt      # CMake 빌드 설정
└── commands.txt        # 빌드 명령어 참조
```

## 🚀 주요 기능

### 1. 모델 초기화 (`init()`)
- Mistral-7B Instruct 모델을 Hugging Face에서 자동 다운로드
- LLAMA.cpp 백엔드 초기화
- 컨텍스트 및 배치 크기 설정 (512 토큰)
- 멀티스레딩 설정 (8 스레드)

### 2. 텍스트 생성 (`predict()`)
- 사용자 프롬프트를 받아 AI 응답 생성
- 추론 시간 측정
- 생성된 토큰 수 계산
- 스레드 안전한 처리

### 3. 응답 구조체
```cpp
struct chatbot_response {
    char* output;           // 생성된 텍스트
    float inference_time;   // 추론 시간 (밀리초)
    int num_tokens;         // 생성된 토큰 수
};
```

## 🔧 빌드 요구사항

### 필수 패키지
```bash
sudo apt install cmake build-essential
```

### 의존성
- **LLAMA.cpp**: AI 모델 추론 엔진
- **CMake 3.22+**: 빌드 시스템
- **C++11 표준**: 컴파일러 요구사항
- **CURL**: 모델 다운로드용
- **pthread**: 멀티스레딩 지원

## 🛠️ 빌드 과정

### 1. LLAMA.cpp 클론
```bash
git clone https://github.com/ggerganov/llama.cpp -b b3145
```

### 2. AMD64 아키텍처용 빌드
```bash
# CMakeLists.txt에서 프로젝트명을 chatbot-amd64로 설정
cmake -B build
cmake --build build --config Release
sudo cp build/libchatbot-amd64.so /usr/lib
```

### 3. ARM64 (Graviton) 아키텍처용 빌드
```bash
# CMakeLists.txt에서 프로젝트명을 chatbot-arm64로 설정
cmake -B build
cmake --build build --config Release
sudo cp build/libchatbot-arm64.so /usr/lib
```

## 🔄 Graviton 최적화 설정

CMakeLists.txt에는 ARM64 최적화를 위한 다음 설정이 포함되어 있습니다:

```cmake
set(LLAMA_CURL ON)          # CURL 지원 활성화
set(LLAMA_SVE ON)           # ARM SVE(Scalable Vector Extension) 활성화
set(CMAKE_POSITION_INDEPENDENT_CODE ON)  # 위치 독립적 코드 생성
```

### ARM SVE 최적화
- **SVE (Scalable Vector Extension)**: ARM64의 고급 벡터 처리 기능
- **Graviton3/4 프로세서**: SVE를 통한 AI 워크로드 가속화
- **성능 향상**: 벡터 연산 최적화로 추론 속도 개선

## 📊 성능 특징

### 모델 설정
- **모델**: Mistral-7B Instruct v0.1 (Q4_K_M 양자화)
- **컨텍스트 크기**: 512 토큰
- **배치 크기**: 512
- **스레드 수**: 8개 (CPU 코어에 따라 조정 가능)

### 메모리 사용량
- **모델 크기**: 약 4.4GB (Q4_K_M 양자화)
- **런타임 메모리**: 컨텍스트 크기에 따라 가변

## 🔌 Java 통합

이 라이브러리는 Java 애플리케이션에서 다음과 같이 사용됩니다:

```java
public interface ChatbotLib extends Library {
    int init();
    int predict(String prompt, int tokens, ChatbotResponse response);
}

// 아키텍처별 라이브러리 로딩
String arch = System.getProperty("os.arch");
String libName = arch.equals("aarch64") ? "libchatbot-arm64.so" : "libchatbot-amd64.so";
ChatbotLib chatbotlib = (ChatbotLib) Native.load(libName, ChatbotLib.class);
```

## 🐛 문제 해결

### 일반적인 문제들

1. **모델 다운로드 실패**
   - 인터넷 연결 확인
   - `/tmp` 디렉토리 권한 확인

2. **라이브러리 로딩 실패**
   - 라이브러리가 `/usr/lib`에 올바르게 설치되었는지 확인
   - `ldd` 명령어로 의존성 확인

3. **메모리 부족 오류**
   - 시스템 메모리가 최소 8GB 이상인지 확인
   - 스왑 메모리 설정 고려

### 디버깅 팁
```bash
# 라이브러리 의존성 확인
ldd /usr/lib/libchatbot-amd64.so

# 시스템 정보 확인
cat /proc/cpuinfo | grep -E "(model name|flags)"

# 메모리 사용량 모니터링
htop
```

## 🎯 성능 최적화

### Graviton 프로세서에서의 최적화
1. **SVE 활용**: ARM SVE 명령어 세트 사용
2. **스레드 최적화**: vCPU 수에 맞춘 스레드 설정
3. **메모리 대역폭**: Graviton의 높은 메모리 대역폭 활용

### 일반적인 최적화
1. **배치 크기 조정**: 메모리와 성능의 균형점 찾기
2. **컨텍스트 크기**: 용도에 맞는 적절한 크기 설정
3. **양자화 레벨**: 정확도와 속도의 트레이드오프 고려

## 📚 참고 자료

- [LLAMA.cpp 프로젝트](https://github.com/ggerganov/llama.cpp)
- [Mistral-7B 모델](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.1-GGUF)
- [AWS Graviton 최적화 가이드](https://github.com/aws/aws-graviton-getting-started)
- [ARM SVE 프로그래밍 가이드](https://developer.arm.com/documentation/102476/latest/)

## 🤝 기여하기

1. 이슈 리포트: 버그나 개선사항을 GitHub Issues에 등록
2. 코드 기여: Pull Request를 통한 코드 개선
3. 문서 개선: README나 코드 주석 개선

## 📄 라이선스

이 프로젝트는 MIT 라이선스 하에 배포됩니다.

---

**AWS Graviton에서 최적화된 AI 챗봇을 경험해보세요!** 🚀

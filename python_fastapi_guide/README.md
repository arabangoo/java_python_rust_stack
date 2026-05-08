# Python FastAPI를 활용한 AI 서비스 구축 가이드

## 목차
1. [개요](#개요)
2. [AI 서비스 개발 기초 이해하기](#ai-서비스-개발-기초-이해하기)
3. [Python/FastAPI vs Java/Spring AI 비교](#pythonfastapi-vs-javaspring-ai-비교)
4. [FastAPI 아키텍처 완전 이해](#fastapi-아키텍처-완전-이해)
5. [프로젝트 설정 단계별 가이드](#프로젝트-설정-단계별-가이드)
6. [핵심 구현 상세 설명](#핵심-구현-상세-설명)
7. [장기 메모리 관리 완전 가이드](#장기-메모리-관리-완전-가이드)
8. [성능 최적화 전략](#성능-최적화-전략)
9. [프로덕션 배포 실전 가이드](#프로덕션-배포-실전-가이드)
10. [모니터링 및 운영](#모니터링-및-운영)
11. [FAQ](#faq)

---
 
## 개요

본 가이드는 **Python과 FastAPI**를 활용하여 AI 서비스를 빠르고 효율적으로 구축하는 방법을 다룹니다. Python의 풍부한 AI 생태계와 FastAPI의 현대적인 웹 프레임워크를 결합하여, 프로토타입부터 프로덕션까지 커버합니다.

### 왜 Python/FastAPI인가?

- **빠른 개발 속도**: 간결한 문법으로 아이디어를 빠르게 구현
- **풍부한 AI 생태계**: LangChain, Transformers, OpenAI SDK 등 최신 라이브러리 즉시 활용
- **비동기 지원**: async/await로 고성능 비동기 처리
- **자동 문서화**: OpenAPI(Swagger) 자동 생성
- **타입 힌트**: Pydantic으로 런타임 검증 및 자동 완성 지원
- **낮은 진입 장벽**: 초보자도 쉽게 시작 가능

### 이 가이드가 적합한 경우

✅ MVP/프로토타입을 빠르게 만들어야 할 때
✅ 최신 AI 기술을 즉시 적용하고 싶을 때
✅ 소규모~중규모 팀 (1~10명)
✅ 빠른 이터레이션이 중요한 스타트업
✅ 데이터 사이언스 팀과 협업이 많을 때
✅ Python 개발자가 많은 조직

---

## AI 서비스 개발 기초 이해하기

### AI 챗봇 서비스란 무엇인가?

AI 챗봇 서비스는 마치 **스마트 어시스턴트**와 대화하는 것과 같습니다:

```
사용자(User) → 질문(Question) → AI 모델(LLM) → 답변(Answer) → 사용자(User)
```

이 과정에서 시스템은:
- 사용자의 질문을 이해해야 함 (자연어 처리)
- 이전 대화를 기억해야 함 (메모리 관리)
- 관련 정보를 찾아야 함 (검색 및 RAG)
- 빠르게 응답해야 함 (성능 최적화)

### Python으로 AI 서비스를 만드는 이유

#### 1. **AI 라이브러리 생태계가 가장 풍부**

Python은 AI/ML의 **사실상 표준 언어**입니다:

```python
# 주요 AI 라이브러리들 - 모두 Python 우선 지원
from langchain import LangChain              # LLM 오케스트레이션
from transformers import AutoModel           # Hugging Face 모델
from openai import OpenAI                    # OpenAI API
from anthropic import Anthropic              # Claude API
import tiktoken                              # 토큰 계산
import chromadb                              # 벡터 DB
```

**새로운 AI 기술이 나오면**:
1. Python 라이브러리가 **가장 먼저** 출시됨
2. 튜토리얼과 예제가 **가장 많음**
3. 커뮤니티 지원이 **가장 활발**

#### 2. **코드가 간결하고 읽기 쉬움**

**같은 기능을 구현할 때**:

Python (5줄):
```python
from openai import OpenAI

client = OpenAI(api_key="sk-...")
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

Java (20줄):
```java
import com.openai.client.OpenAIClient;
import com.openai.models.*;

public class ChatExample {
    public static void main(String[] args) {
        OpenAIClient client = OpenAIClient.builder()
            .apiKey("sk-...")
            .build();

        ChatCompletionRequest request = ChatCompletionRequest.builder()
            .model("gpt-4")
            .messages(Arrays.asList(
                ChatMessage.builder()
                    .role("user")
                    .content("Hello!")
                    .build()
            ))
            .build();

        ChatCompletion response = client.createChatCompletion(request);
        System.out.println(response.getChoices().get(0).getMessage().getContent());
    }
}
```

#### 3. **개발 속도가 빠름**

```
아이디어 → 프로토타입 → 테스트 → 배포

Python/FastAPI:
- 프로토타입: 1~2일
- 기본 기능 완성: 1~2주
- 프로덕션 준비: 2~4주

Java/Spring:
- 프로토타입: 3~5일 (프로젝트 설정 복잡)
- 기본 기능 완성: 3~4주
- 프로덕션 준비: 4~8주
```

#### 4. **동적 타입의 유연성**

**런타임에 타입 변경 가능**:
```python
# Python - 유연함
data = {"name": "John", "age": 30}
data["new_field"] = "추가 필드"  # 자유롭게 추가 가능

# API 응답도 쉽게 확장
response = {
    "message": "답변",
    "metadata": {...}  # 필요에 따라 추가
}
```

**하지만 타입 안정성이 필요하면?**
```python
# Pydantic으로 타입 안정성 확보
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str | None = None

user = User(name="John", age="30")  # ❌ ValidationError 발생
user = User(name="John", age=30)    # ✅ OK
```

### FastAPI가 다른 Python 프레임워크보다 나은 이유

#### FastAPI vs Flask

| 기능 | FastAPI | Flask |
|------|---------|-------|
| **성능** | ⭐⭐⭐⭐⭐ (Starlette 기반, 비동기) | ⭐⭐⭐ (동기 처리) |
| **타입 안정성** | ⭐⭐⭐⭐⭐ (Pydantic 내장) | ⭐ (수동 검증 필요) |
| **자동 문서화** | ⭐⭐⭐⭐⭐ (Swagger 자동 생성) | ⭐ (수동 작성) |
| **비동기 지원** | ⭐⭐⭐⭐⭐ (네이티브 async/await) | ⭐⭐ (제한적) |
| **학습 곡선** | ⭐⭐⭐⭐ (현대적이지만 쉬움) | ⭐⭐⭐⭐⭐ (매우 쉬움) |
| **생태계** | ⭐⭐⭐⭐ (성장 중) | ⭐⭐⭐⭐⭐ (성숙함) |

**코드 비교**:

```python
# Flask - 수동 검증 필요
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/chat", methods=["POST"])
def chat():
    data = request.get_json()

    # 수동 검증
    if "message" not in data:
        return jsonify({"error": "message required"}), 400
    if not isinstance(data["message"], str):
        return jsonify({"error": "message must be string"}), 400

    message = data["message"]
    response = f"You said: {message}"
    return jsonify({"response": response})


# FastAPI - 자동 검증
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class ChatRequest(BaseModel):
    message: str

class ChatResponse(BaseModel):
    response: str

@app.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    # 자동으로 검증됨! message가 없거나 타입이 틀리면 422 에러
    response = f"You said: {request.message}"
    return ChatResponse(response=response)
```

**FastAPI의 장점**:
1. 타입 힌트 → 자동 검증 → 자동 문서화 (한 번에!)
2. 비동기 처리로 높은 동시성
3. 최신 Python 기능 적극 활용 (3.8+)

#### FastAPI 성능

```
동시 요청 1000개 처리 (벤치마크):

Flask (동기):
- 요청/초: 1,200 req/s
- 평균 응답시간: 830ms

FastAPI (비동기):
- 요청/초: 3,500 req/s
- 평균 응답시간: 285ms

→ FastAPI가 약 3배 빠름!
```

### Python의 한계와 극복 방법

#### 한계 1: 타입 안정성

**문제**:
```python
def process(data):  # data가 뭐죠?
    return data.upper()  # data가 int면? 💥 런타임 에러
```

**해결: Type Hints + Pydantic**:
```python
from pydantic import BaseModel

class Data(BaseModel):
    text: str

def process(data: Data) -> str:
    return data.text.upper()

# IDE가 자동완성 제공
# 런타임에 검증됨
```

#### 한계 2: GIL (Global Interpreter Lock)

**문제**: Python은 한 번에 하나의 스레드만 실행 (CPU 병렬 처리 제한)

**해결 방법**:
1. **비동기 처리** (I/O 작업에 효과적)
   ```python
   import asyncio

   async def call_ai():
       # I/O 대기 중에 다른 작업 처리 가능
       response = await openai.chat.completions.create(...)
       return response
   ```

2. **멀티프로세스** (CPU 작업에 효과적)
   ```python
   from multiprocessing import Pool

   with Pool(4) as pool:
       results = pool.map(process_data, data_list)
   ```

3. **컨테이너 스케일링** (가장 일반적)
   ```bash
   # Docker로 여러 인스턴스 실행
   docker-compose up --scale api=4
   ```

#### 한계 3: 메모리 사용량

**Python vs Java 메모리 비교**:
```
빈 애플리케이션:
- Python: 30MB
- Java: 150MB (JVM 오버헤드)

→ Python이 가벼움 ✅

대용량 데이터 처리 (1GB 데이터):
- Python: 3GB (메모리 오버헤드 높음)
- Java: 1.5GB (효율적인 메모리 관리)

→ Java가 유리 ✅
```

**Python 메모리 최적화**:
```python
# ❌ 비효율적
data = [process(item) for item in huge_list]  # 전체 리스트를 메모리에 로드

# ✅ 효율적
data = (process(item) for item in huge_list)  # Generator - 필요할 때만 로드
```

### 언제 Python/FastAPI를 선택해야 할까?

#### 의사결정 플로우차트

```
[시작]
    ↓
최신 AI 기술을 빠르게 적용해야 하나요?
    ↓ YES                          ↓ NO
Python/FastAPI 강력 추천      팀에 Python 개발자가 있나요?
                                ↓ YES        ↓ NO
                              Python 추천   Java 고려

프로토타입 단계인가요?
    ↓ YES              ↓ NO
Python으로 빠르게   사용자 수가 많나요? (10만+)
    시작               ↓ YES            ↓ NO
                    확장성 고려 필요   Python으로 충분
                    (Python도 가능)
```

#### Python/FastAPI를 선택해야 하는 경우

✅ **스타트업/MVP**
- 빠른 제품 출시가 중요
- 시장 검증이 우선
- 유연한 요구사항 변경

✅ **AI/ML 중심 서비스**
- LLM, RAG, 에이전트 시스템
- 최신 AI 기술 즉시 적용
- 데이터 사이언티스트와 협업

✅ **중소규모 프로젝트**
- 사용자: ~100만
- 동시 접속: ~1만
- 팀 규모: 1~10명

✅ **프로토타입 → 프로덕션**
- 빠른 검증 후 점진적 개선
- 마이크로서비스 아키텍처로 확장

#### Java/Spring을 선택해야 하는 경우

✅ **엔터프라이즈 환경**
- 기존 Java 시스템과 통합
- 대규모 조직 (수백 명)
- 엄격한 거버넌스

✅ **대규모 트래픽**
- 사용자: 1000만+
- 동시 접속: 10만+
- 복잡한 트랜잭션

✅ **장기 운영**
- 5년+ 유지보수 계획
- 대규모 리팩토링 빈번
- 타입 안정성 최우선

### 하이브리드 아키텍처

**최선의 선택**은 두 가지 장점을 결합하는 것:

```
[클라이언트]
    ↓
[API Gateway - FastAPI]
  ↓                ↓
[AI Service]   [Business Logic]
(Python)       (Java/Spring)
  ↓                ↓
[Vector DB]    [PostgreSQL]
```

**역할 분담**:
- **Python/FastAPI**: AI 처리, 벡터 검색, 실험적 기능
- **Java/Spring**: 핵심 비즈니스 로직, 트랜잭션, 레거시 통합

**장점**:
- Python의 빠른 개발 + Java의 안정성
- 각 언어의 강점 활용
- 팀의 전문성 활용

---

## Python/FastAPI vs Java/Spring AI 비교

### 개발 경험 비교

#### 프로젝트 시작하기

**Python/FastAPI**:
```bash
# 1. 가상환경 생성 (5초)
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 2. 라이브러리 설치 (30초)
pip install fastapi uvicorn openai langchain

# 3. 코드 작성 (5분)
# main.py
from fastapi import FastAPI
from openai import OpenAI

app = FastAPI()
client = OpenAI()

@app.post("/chat")
async def chat(message: str):
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": message}]
    )
    return {"response": response.choices[0].message.content}

# 4. 실행 (즉시)
uvicorn main:app --reload

# 총 시간: 약 10분
```

**Java/Spring AI**:
```bash
# 1. Spring Initializr에서 프로젝트 생성 (2분)
# 2. Maven 빌드 (1~3분)
mvn clean install

# 3. 코드 작성 (15분)
# ChatController.java, ChatService.java, application.yml 등

# 4. 실행 (30초~1분)
mvn spring-boot:run

# 총 시간: 약 20~30분
```

#### 코드 비교: 대화 메모리 관리

**Python/FastAPI - 간결함**:
```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

# 메모리 초기화 (2줄)
memory = ConversationBufferMemory()
llm = ChatOpenAI(model="gpt-4")

# 체인 생성 (1줄)
conversation = ConversationChain(llm=llm, memory=memory)

# 대화 (1줄)
response = conversation.predict(input="안녕하세요!")
print(response)

# 총 코드: 약 10줄
```

**Java/Spring AI - 명시적**:
```java
@Service
public class ChatService {
    private final ChatClient chatClient;
    private final ConversationRepository repository;

    public ChatService(ChatClient.Builder builder,
                      ConversationRepository repository) {
        this.chatClient = builder.build();
        this.repository = repository;
    }

    @Transactional
    public ChatResponse chat(ChatRequest request) {
        // 1. 대화 히스토리 로드
        List<Message> history = repository
            .findByUserId(request.getUserId());

        // 2. 메시지 변환
        List<Message> messages = new ArrayList<>();
        for (Conversation conv : history) {
            messages.add(new UserMessage(conv.getUserMessage()));
            messages.add(new AssistantMessage(conv.getAssistantMessage()));
        }

        // 3. AI 호출
        ChatResponse response = chatClient.prompt()
            .messages(messages)
            .user(request.getMessage())
            .call()
            .chatResponse();

        // 4. 저장
        Conversation conversation = new Conversation();
        conversation.setUserId(request.getUserId());
        conversation.setUserMessage(request.getMessage());
        conversation.setAssistantMessage(response.getResult().getOutput().getContent());
        repository.save(conversation);

        return response;
    }
}

// 총 코드: 약 40~50줄
```

### 성능 비교

#### 벤치마크 시나리오: 동시 요청 1000개

**테스트 환경**:
- CPU: 8 Core
- RAM: 16GB
- 시나리오: 간단한 챗봇 응답 (OpenAI API 호출)

**결과**:

```
Python/FastAPI (uvicorn workers=4):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
평균 응답시간: 245ms
Throughput: 3,200 req/s
메모리 사용: 512MB (안정화)
CPU 사용률: 60~70%
에러율: 0.1%

장점: 빠른 시작, 낮은 메모리
단점: CPU 바운드 작업에 제한적


Java/Spring AI (Tomcat):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
평균 응답시간: 180ms
Throughput: 4,800 req/s
메모리 사용: 768MB (초기) → 512MB (안정화)
CPU 사용률: 50~60%
에러율: 0.05%

장점: 높은 처리량, 안정적
단점: 느린 시작, 초기 메모리 높음
```

**결론**:
- **I/O 중심 작업**: FastAPI 충분히 빠름 (비동기 효과)
- **CPU 중심 작업**: Spring AI 유리 (멀티스레딩)
- **대규모 동시 접속**: 둘 다 가능 (적절한 스케일링으로)

### 생태계 비교

#### AI 라이브러리 지원

**Python**:
```
LangChain: ⭐⭐⭐⭐⭐ (Python 최우선 지원)
- 릴리즈: Python 먼저 → Java 수개월 후
- 기능: Python 풍부 > Java 기본 기능만

Hugging Face Transformers: ⭐⭐⭐⭐⭐ (Python 전용)
- 최신 모델 즉시 사용 가능
- Java 지원 없음

OpenAI Python SDK: ⭐⭐⭐⭐⭐
- 공식 라이브러리
- 모든 기능 즉시 지원

Vector Databases:
- Pinecone: Python SDK 완성도 높음
- Weaviate: 둘 다 좋음
- ChromaDB: Python 우선
```

**Java**:
```
Spring AI: ⭐⭐⭐⭐ (성장 중)
- Spring 생태계와 완벽 통합
- 기본 기능 탄탄

LangChain4j: ⭐⭐⭐⭐ (Java 전용 구현)
- LangChain의 개념을 Java로 재구현
- 기능은 제한적이지만 타입 안전

DJL (Deep Java Library): ⭐⭐⭐
- 로컬 모델 실행
- 성능 좋음

Vector Databases:
- pgvector: 둘 다 좋음
- Elasticsearch: 둘 다 좋음
```

### 개발자 경험 (DX)

#### 디버깅

**Python**:
```python
# 즉시 테스트 가능 (REPL)
>>> from openai import OpenAI
>>> client = OpenAI()
>>> response = client.chat.completions.create(...)
>>> print(response)  # 바로 확인!

# 간단한 로깅
import logging
logging.debug(f"User message: {message}")

# IPython 디버거
from IPython import embed
embed()  # 이 시점에서 대화형 셸 시작
```

**Java**:
```java
// 컴파일 필요
// 수정 → 컴파일 → 실행 → 확인

// 로깅 설정 필요
private static final Logger log =
    LoggerFactory.getLogger(ChatService.class);
log.debug("User message: {}", message);

// 디버거
// IDE에서 breakpoint 설정 → 디버그 모드 실행
```

#### 배포 속도

**Python**:
```bash
# Dockerfile
FROM python:3.11-slim
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]

# 빌드 시간: 30초~1분
# 이미지 크기: 200~300MB
```

**Java**:
```dockerfile
# Dockerfile
FROM maven:3.9-eclipse-temurin-21 AS builder
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src src
RUN mvn package

FROM eclipse-temurin:21-jre-alpine
COPY --from=builder target/*.jar app.jar
CMD ["java", "-jar", "app.jar"]

# 빌드 시간: 3~5분
# 이미지 크기: 150~200MB
```

### 비용 비교

#### 개발 비용

```
프로젝트 기간: 3개월 (MVP)
팀 구성: 개발자 2명

Python/FastAPI:
- 개발 시간: 2개월 (빠른 프로토타이핑)
- 버그 수정: 2주 (타입 에러 등)
- 문서 작성: 2주
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
총 개발 비용: 2.5개월 × 2명 = 5 Man-Month

Java/Spring AI:
- 초기 설정: 1주
- 개발 시간: 2.5개월 (보일러플레이트 많음)
- 버그 수정: 1주 (컴파일 타임 체크)
- 문서 작성: 1주
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
총 개발 비용: 3개월 × 2명 = 6 Man-Month

절감: 약 17% 시간 단축 (Python)
```

#### 운영 비용

```
트래픽: 일 100만 요청
서버: AWS EC2

Python/FastAPI:
- 인스턴스: t3.medium × 4대
- 비용: $150/월 × 4 = $600/월

Java/Spring AI:
- 인스턴스: t3.medium × 3대 (더 효율적)
- 비용: $150/월 × 3 = $450/월

절감: 약 25% (Java가 저렴)

하지만:
- Python: 개발 속도 빠름 → 더 빠른 수익 창출
- Java: 운영 비용 낮음 → 장기적으로 유리

결론:
- 초기 단계: Python 유리 (빠른 시장 진입)
- 성숙 단계: Java 유리 (운영 비용 절감)
```

### 최종 비교표

| 측면 | Python/FastAPI | Java/Spring AI | 승자 |
|------|---------------|----------------|------|
| **개발 속도** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 🐍 Python |
| **AI 생태계** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 🐍 Python |
| **타입 안정성** | ⭐⭐⭐ (Pydantic) | ⭐⭐⭐⭐⭐ | ☕ Java |
| **성능** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ☕ Java |
| **메모리 효율** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ☕ Java |
| **학습 곡선** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 🐍 Python |
| **배포 편의성** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 🐍 Python |
| **장기 유지보수** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ☕ Java |
| **대규모 팀 협업** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ☕ Java |
| **엔터프라이즈 통합** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ☕ Java |

---

## FastAPI 아키텍처 완전 이해

### 아키텍처 설계 철학

FastAPI는 **속도와 간결함**을 동시에 추구합니다:

```
식당 구조              →    FastAPI 구조
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
메뉴판 (주문서)        →    API 엔드포인트 (@app.post)
웨이터 (주문 검증)     →    Pydantic (자동 검증)
주방장 (요리)          →    비즈니스 로직 (함수)
재료 창고 (DB)         →    데이터베이스 / 벡터 DB
```

### FastAPI의 핵심 개념

#### 1. **ASGI (Asynchronous Server Gateway Interface)**

**WSGI (Flask) vs ASGI (FastAPI)**:

```python
# WSGI (Flask) - 동기 처리
@app.route("/slow")
def slow_endpoint():
    time.sleep(2)  # 2초 대기 (다른 요청 블로킹!)
    return "Done"

# 요청 1 → 2초 대기 → 완료
# 요청 2 → 요청 1 완료 대기 → 2초 대기 → 완료
# 총 4초


# ASGI (FastAPI) - 비동기 처리
@app.get("/slow")
async def slow_endpoint():
    await asyncio.sleep(2)  # 2초 대기 (다른 요청 처리 가능!)
    return "Done"

# 요청 1 → 2초 대기 (비동기) ┐
# 요청 2 → 2초 대기 (비동기) ┴→ 동시 처리
# 총 2초
```

**실전 예시: AI API 호출**:

```python
@app.post("/chat")
async def chat(request: ChatRequest):
    # OpenAI API 호출 (I/O 대기 중 다른 요청 처리 가능)
    response = await openai_client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": request.message}]
    )
    return {"response": response.choices[0].message.content}

# 100개 요청이 동시에 들어와도:
# - 순차 처리: 100초 (각 1초씩)
# - 비동기 처리: 약 1~2초 (동시 처리)
```

#### 2. **Pydantic Models - 자동 검증**

**역할**: 데이터의 **청사진**이자 **검증기**

```python
from pydantic import BaseModel, Field, validator
from typing import Optional

class ChatRequest(BaseModel):
    """채팅 요청 모델

    사용자의 메시지를 받아 검증합니다.
    """
    message: str = Field(
        ...,  # 필수 필드
        min_length=1,
        max_length=5000,
        description="사용자 메시지 (1~5000자)"
    )
    user_id: str = Field(..., min_length=1)
    session_id: Optional[str] = None
    temperature: float = Field(0.7, ge=0.0, le=2.0)  # 0.0~2.0 범위

    @validator("message")
    def message_must_not_be_empty(cls, v):
        """빈 문자열 검증"""
        if not v.strip():
            raise ValueError("메시지가 비어있습니다")
        return v.strip()

    class Config:
        json_schema_extra = {
            "example": {
                "message": "안녕하세요!",
                "user_id": "user123",
                "temperature": 0.7
            }
        }

# FastAPI에서 사용
@app.post("/chat")
async def chat(request: ChatRequest):
    # request.message는 이미 검증됨!
    # - 타입 체크 완료
    # - 길이 체크 완료
    # - 빈 문자열 체크 완료
    return {"received": request.message}
```

**자동으로 얻는 것들**:
1. ✅ 타입 검증 (런타임)
2. ✅ OpenAPI 스키마 생성
3. ✅ 자동 문서화 (Swagger UI)
4. ✅ IDE 자동완성
5. ✅ 에러 메시지 (422 Unprocessable Entity)

#### 3. **Dependency Injection (의존성 주입)**

**일반 설명**: 필요한 것을 **자동으로** 제공받기

```python
from fastapi import Depends
from typing import Annotated

# 데이터베이스 세션 제공
async def get_db_session():
    """DB 세션을 생성하고 자동으로 닫아줌"""
    session = create_session()
    try:
        yield session  # 함수에 전달
    finally:
        session.close()  # 자동으로 정리

# 현재 사용자 가져오기
async def get_current_user(token: str = Header(...)):
    """토큰으로 사용자 인증"""
    user = verify_token(token)
    if not user:
        raise HTTPException(401, "Unauthorized")
    return user

# 엔드포인트에서 사용
@app.post("/chat")
async def chat(
    request: ChatRequest,
    db: Annotated[Session, Depends(get_db_session)],  # DB 자동 주입
    user: Annotated[User, Depends(get_current_user)]  # 사용자 자동 주입
):
    # db와 user가 자동으로 준비됨!
    conversation = Conversation(
        user_id=user.id,
        message=request.message
    )
    db.add(conversation)
    db.commit()
    return {"status": "saved"}
```

**장점**:
- 재사용 가능한 로직 (DRY)
- 자동 정리 (try/finally)
- 테스트 쉬움 (의존성 교체 가능)

### FastAPI 레이어 구조

```
[API Layer - 엔드포인트]
  역할: HTTP 요청/응답 처리
  파일: routers/*.py
  예시: @app.post("/chat")
            ↓
[Validation Layer - Pydantic]
  역할: 자동 검증 및 직렬화
  파일: schemas/*.py
  예시: ChatRequest, ChatResponse
            ↓
[Business Logic Layer - Services]
  역할: 핵심 비즈니스 로직
  파일: services/*.py
  예시: ChatService, MemoryService
            ↓
[Data Access Layer - Repositories]
  역할: 데이터베이스 CRUD
  파일: repositories/*.py, models/*.py
  예시: SQLAlchemy ORM
            ↓
[Infrastructure Layer]
  역할: 외부 서비스 연동
  파일: integrations/*.py
  예시: OpenAI API, Vector DB
```

### 데이터 흐름 상세 예시

사용자가 "오늘 날씨 어때?"라고 질문하면:

```python
# ========================================
# 1단계: API Layer - 요청 받기
# ========================================
@app.post("/api/v1/chat", response_model=ChatResponse)
async def chat(
    request: ChatRequest,
    db: Session = Depends(get_db),
    chat_service: ChatService = Depends(get_chat_service)
):
    """
    채팅 엔드포인트

    1. Pydantic이 자동으로 request 검증
    2. DB 세션 자동 주입
    3. ChatService 자동 주입
    """
    response = await chat_service.chat(
        user_id=request.user_id,
        message=request.message,
        session=db
    )
    return response


# ========================================
# 2단계: Validation - 자동 검증
# ========================================
# ChatRequest 모델이 자동으로:
# - message가 str인지 확인
# - user_id가 있는지 확인
# - temperature가 0~2 범위인지 확인
# 실패 시 422 에러 자동 반환


# ========================================
# 3단계: Service - 비즈니스 로직
# ========================================
class ChatService:
    def __init__(self, openai_client, vector_store):
        self.openai_client = openai_client
        self.vector_store = vector_store

    async def chat(
        self,
        user_id: str,
        message: str,
        session: Session
    ) -> ChatResponse:
        # 3-1. 대화 히스토리 로드
        history = await self.get_conversation_history(
            user_id,
            session,
            limit=10
        )

        # 3-2. 벡터 메모리 검색
        relevant_memories = await self.vector_store.search(
            query=message,
            user_id=user_id,
            limit=3
        )

        # 3-3. 프롬프트 구성
        system_prompt = self.build_system_prompt(relevant_memories)
        messages = self.format_messages(history, message)

        # 3-4. OpenAI API 호출
        response = await self.openai_client.chat.completions.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": system_prompt},
                *messages,
                {"role": "user", "content": message}
            ]
        )

        assistant_message = response.choices[0].message.content

        # 3-5. 대화 저장
        await self.save_conversation(
            user_id=user_id,
            user_message=message,
            assistant_message=assistant_message,
            session=session
        )

        # 3-6. 벡터 임베딩 (비동기)
        asyncio.create_task(
            self.vector_store.add_conversation(
                user_id=user_id,
                content=f"User: {message}\nAssistant: {assistant_message}"
            )
        )

        return ChatResponse(
            message=assistant_message,
            user_id=user_id
        )


# ========================================
# 4단계: Repository - 데이터 저장/조회
# ========================================
class ConversationRepository:
    async def get_recent(
        self,
        user_id: str,
        session: Session,
        limit: int = 10
    ) -> List[Conversation]:
        """최근 대화 조회"""
        return session.query(Conversation)\
            .filter(Conversation.user_id == user_id)\
            .order_by(Conversation.created_at.desc())\
            .limit(limit)\
            .all()

    async def save(
        self,
        conversation: Conversation,
        session: Session
    ):
        """대화 저장"""
        session.add(conversation)
        await session.commit()


# ========================================
# 5단계: Infrastructure - 외부 연동
# ========================================
class VectorStoreService:
    def __init__(self, chroma_client):
        self.client = chroma_client
        self.collection = self.client.get_or_create_collection("conversations")

    async def search(
        self,
        query: str,
        user_id: str,
        limit: int = 3
    ) -> List[str]:
        """의미 기반 검색"""
        results = self.collection.query(
            query_texts=[query],
            where={"user_id": user_id},
            n_results=limit
        )
        return results["documents"][0]

    async def add_conversation(
        self,
        user_id: str,
        content: str
    ):
        """대화를 벡터로 저장"""
        self.collection.add(
            documents=[content],
            metadatas=[{"user_id": user_id}],
            ids=[f"{user_id}_{uuid.uuid4()}"]
        )
```

### 프로젝트 구조 예시

```
fastapi_ai_service/
├── main.py                      # 애플리케이션 진입점
├── config.py                    # 설정 (Pydantic Settings)
├── requirements.txt             # 의존성
│
├── routers/                     # API 엔드포인트
│   ├── __init__.py
│   ├── chat.py                  # 채팅 라우터
│   └── health.py                # 헬스체크
│
├── schemas/                     # Pydantic 모델
│   ├── __init__.py
│   ├── chat.py                  # ChatRequest, ChatResponse
│   └── common.py                # 공통 스키마
│
├── services/                    # 비즈니스 로직
│   ├── __init__.py
│   ├── chat_service.py          # 채팅 서비스
│   ├── memory_service.py        # 메모리 관리
│   └── vector_service.py        # 벡터 검색
│
├── repositories/                # 데이터 접근
│   ├── __init__.py
│   ├── conversation.py          # 대화 CRUD
│   └── user.py                  # 사용자 CRUD
│
├── models/                      # SQLAlchemy 모델
│   ├── __init__.py
│   ├── conversation.py          # Conversation 테이블
│   └── user.py                  # User 테이블
│
├── integrations/                # 외부 서비스
│   ├── __init__.py
│   ├── openai_client.py         # OpenAI 클라이언트
│   └── vector_store.py          # ChromaDB 등
│
├── dependencies/                # FastAPI 의존성
│   ├── __init__.py
│   ├── database.py              # DB 세션
│   └── auth.py                  # 인증
│
├── utils/                       # 유틸리티
│   ├── __init__.py
│   └── logger.py                # 로깅
│
└── tests/                       # 테스트
    ├── test_chat.py
    └── test_memory.py
```

### 다음 섹션에서 배울 내용

이제 아키텍처를 이해했으니:
1. **실제 프로젝트 설정** - 가상환경부터 라이브러리 설치까지
2. **코드 구현** - 각 레이어별 실제 코드
3. **테스트 전략** - pytest로 테스트 작성

---

## 프로젝트 설정 단계별 가이드

### 시작하기 전에

#### 필요한 도구 설치

```
[필수 도구]
1. Python 3.10+
   - 다운로드: https://www.python.org/downloads/
   - 설치 확인: python --version
   - 권장: Python 3.11 (성능 향상)

2. pip (Python 패키지 관리자)
   - Python 설치 시 자동 포함
   - 업그레이드: python -m pip install --upgrade pip

3. IDE (택1)
   - VS Code (추천) + Python Extension
   - PyCharm
   - Cursor

[선택 도구]
4. Docker Desktop
   - PostgreSQL, Redis 실행용
   - https://www.docker.com/products/docker-desktop

5. Postman 또는 HTTPie
   - API 테스트용
   - httpie: pip install httpie
```

#### Python 가상환경 이해하기

**가상환경이란?**
프로젝트마다 **독립적인 Python 환경**을 만드는 것

**왜 필요한가?**
```
문제 상황:
프로젝트 A: FastAPI 0.100 필요
프로젝트 B: FastAPI 0.110 필요

가상환경 없이:
- 하나만 설치 가능
- 버전 충돌 발생 💥

가상환경 사용:
- 프로젝트마다 독립된 환경
- 버전 충돌 없음 ✅
```

**가상환경 방식 비교**:

| 방식 | 장점 | 단점 | 추천 |
|------|------|------|------|
| **venv** | 내장, 간단 | 기본 기능만 | ⭐⭐⭐⭐⭐ |
| **virtualenv** | 빠름, 기능 많음 | 별도 설치 | ⭐⭐⭐⭐ |
| **conda** | 과학 라이브러리 강력 | 무겁고 느림 | ⭐⭐⭐ |
| **poetry** | 의존성 관리 우수 | 학습 곡선 | ⭐⭐⭐⭐ |
| **uv** | 매우 빠름 (Rust 기반) | 신생 도구 | ⭐⭐⭐⭐ |

### 1. 프로젝트 생성 및 가상환경 설정

#### Step 1: 프로젝트 디렉토리 생성

```bash
# 프로젝트 폴더 만들기
mkdir fastapi-ai-service
cd fastapi-ai-service

# Git 초기화 (선택사항)
git init
```

#### Step 2: 가상환경 생성

**venv 사용 (권장)**:
```bash
# Windows
python -m venv venv

# macOS/Linux
python3 -m venv venv
```

**실행 결과**:
```
fastapi-ai-service/
└── venv/                # 가상환경 폴더
    ├── Scripts/         # Windows
    ├── bin/             # macOS/Linux
    ├── Include/
    ├── Lib/
    └── pyvenv.cfg
```

#### Step 3: 가상환경 활성화

```bash
# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate

# 활성화 확인 (프롬프트에 (venv) 표시)
(venv) C:\fastapi-ai-service>
```

**가상환경 활성화 확인**:
```bash
# Python 경로 확인
which python  # macOS/Linux
where python  # Windows

# 출력: .../venv/bin/python 또는 ...\venv\Scripts\python.exe
# → 가상환경의 Python 사용 중! ✅
```

#### Step 4: pip 업그레이드

```bash
python -m pip install --upgrade pip
```

### 2. 의존성 설치

#### requirements.txt 이해하기

**requirements.txt란?**
프로젝트에 필요한 **라이브러리 목록**

**왜 필요한가?**
```
수동 설치:
pip install fastapi
pip install uvicorn
pip install openai
...  # 10개 더

다른 개발자:
"어떤 라이브러리 설치하죠?" 😱

requirements.txt 사용:
pip install -r requirements.txt  # 한 번에!
→ 정확히 같은 환경 구축 ✅
```

#### requirements.txt 작성

```bash
# requirements.txt 생성
touch requirements.txt  # macOS/Linux
type nul > requirements.txt  # Windows
```

**requirements.txt 내용** (상세 주석 포함):

```txt
# ===================================
# 웹 프레임워크
# ===================================

# FastAPI - 메인 웹 프레임워크
# 최신 비동기 Python 웹 프레임워크
fastapi==0.109.0

# Uvicorn - ASGI 서버
# FastAPI 애플리케이션 실행용
# standard: watchfiles, websockets 포함
uvicorn[standard]==0.27.0

# Pydantic - 데이터 검증
# FastAPI의 핵심 (자동 설치되지만 명시)
# v2: 성능 대폭 향상 (Rust 기반)
pydantic==2.5.0
pydantic-settings==2.1.0  # 환경변수 관리


# ===================================
# AI 및 LLM
# ===================================

# OpenAI Python SDK
# GPT-4, GPT-3.5, Embeddings 사용
openai==1.10.0

# LangChain - LLM 오케스트레이션
# 체인, 메모리, 에이전트 등
langchain==0.1.0
langchain-openai==0.0.2  # OpenAI 통합
langchain-community==0.0.13  # 커뮤니티 통합

# Anthropic Claude (선택사항)
anthropic==0.8.0

# Tiktoken - 토큰 계산
# OpenAI 토큰 카운팅 (비용 계산)
tiktoken==0.5.2


# ===================================
# 데이터베이스
# ===================================

# SQLAlchemy - ORM
# Python의 가장 강력한 ORM
sqlalchemy==2.0.25
alembic==1.13.1  # 마이그레이션 도구

# PostgreSQL 드라이버
# 비동기 지원
asyncpg==0.29.0
psycopg2-binary==2.9.9  # 동기 (필요시)

# pgvector - PostgreSQL 벡터 확장
# 임베딩 벡터 저장용
pgvector==0.2.4


# ===================================
# 벡터 데이터베이스
# ===================================

# ChromaDB - 경량 벡터 DB
# 로컬 개발에 최적
chromadb==0.4.22

# Pinecone (선택사항)
# 프로덕션용 관리형 벡터 DB
# pinecone-client==3.0.0

# Weaviate (선택사항)
# 오픈소스 벡터 DB
# weaviate-client==4.4.0


# ===================================
# 캐싱
# ===================================

# Redis Python Client
# 비동기 지원
redis==5.0.1
hiredis==2.3.2  # 성능 향상

# Cachetools - 인메모리 캐시
# 간단한 캐싱용
cachetools==5.3.2


# ===================================
# 유틸리티
# ===================================

# Python-dotenv - 환경변수 관리
# .env 파일 로드
python-dotenv==1.0.0

# Loguru - 로깅
# 더 예쁘고 강력한 로거
loguru==0.7.2

# Python-jose - JWT 토큰
# 인증/인가용
python-jose[cryptography]==3.3.0

# Passlib - 비밀번호 해싱
# bcrypt 알고리즘
passlib[bcrypt]==1.7.4

# Python-multipart - 파일 업로드
# FastAPI 파일 업로드 지원
python-multipart==0.0.6


# ===================================
# 테스트
# ===================================

# Pytest - 테스트 프레임워크
pytest==7.4.4
pytest-asyncio==0.23.3  # 비동기 테스트
pytest-cov==4.1.0  # 코드 커버리지

# HTTPX - HTTP 클라이언트
# FastAPI 테스트용
httpx==0.26.0


# ===================================
# 개발 도구
# ===================================

# Black - 코드 포맷터
# PEP 8 스타일 자동 적용
black==23.12.1

# Ruff - Linter
# 매우 빠른 린터 (Rust 기반)
ruff==0.1.11

# MyPy - 타입 체커
# 정적 타입 검사
mypy==1.8.0


# ===================================
# 모니터링
# ===================================

# Prometheus Client
# 메트릭 수집
prometheus-client==0.19.0

# Sentry SDK - 에러 트래킹
sentry-sdk[fastapi]==1.39.2
```

#### 의존성 설치

```bash
# requirements.txt 기반 설치
pip install -r requirements.txt

# 설치 시간: 약 2~3분
# 설치되는 패키지: 약 50~100개 (의존성 포함)
```

#### 설치 확인

```bash
# 설치된 패키지 목록
pip list

# 특정 패키지 버전 확인
pip show fastapi

# 출력:
# Name: fastapi
# Version: 0.109.0
# Summary: FastAPI framework, high performance, easy to learn...
```

### 3. 프로젝트 구조 생성

```bash
# 디렉토리 생성 (한 번에)
mkdir -p app/{routers,schemas,services,repositories,models,integrations,dependencies,utils}
mkdir tests

# 또는 하나씩 (Windows)
mkdir app
mkdir app\routers
mkdir app\schemas
mkdir app\services
mkdir app\repositories
mkdir app\models
mkdir app\integrations
mkdir app\dependencies
mkdir app\utils
mkdir tests
```

**생성된 구조**:
```
fastapi-ai-service/
├── venv/                     # 가상환경 (Git 제외)
├── app/                      # 메인 애플리케이션
│   ├── __init__.py
│   ├── main.py               # 진입점
│   ├── config.py             # 설정
│   ├── routers/              # API 라우터
│   │   └── __init__.py
│   ├── schemas/              # Pydantic 모델
│   │   └── __init__.py
│   ├── services/             # 비즈니스 로직
│   │   └── __init__.py
│   ├── repositories/         # 데이터 접근
│   │   └── __init__.py
│   ├── models/               # SQLAlchemy 모델
│   │   └── __init__.py
│   ├── integrations/         # 외부 서비스
│   │   └── __init__.py
│   ├── dependencies/         # FastAPI 의존성
│   │   └── __init__.py
│   └── utils/                # 유틸리티
│       └── __init__.py
├── tests/                    # 테스트
│   └── __init__.py
├── requirements.txt          # 의존성
├── .env                      # 환경변수 (Git 제외)
├── .env.example              # 환경변수 예시
├── .gitignore                # Git 제외 파일
└── README.md                 # 프로젝트 설명
```

#### __init__.py 파일 생성

```bash
# 모든 디렉토리에 __init__.py 생성
touch app/__init__.py
touch app/routers/__init__.py
touch app/schemas/__init__.py
touch app/services/__init__.py
touch app/repositories/__init__.py
touch app/models/__init__.py
touch app/integrations/__init__.py
touch app/dependencies/__init__.py
touch app/utils/__init__.py
touch tests/__init__.py

# Windows
type nul > app\__init__.py
type nul > app\routers\__init__.py
# ... (각 디렉토리마다)
```

**__init__.py가 필요한 이유**:
```python
# __init__.py가 없으면:
from app.services.chat_service import ChatService  # ❌ 에러

# __init__.py가 있으면:
from app.services.chat_service import ChatService  # ✅ OK
```

### 4. 설정 파일 작성

#### .env 파일 (환경변수)

```.env
# ===================================
# AI 서비스 설정
# ===================================

# OpenAI API
OPENAI_API_KEY=sk-proj-...
OPENAI_MODEL=gpt-4-turbo-preview
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_MAX_TOKENS=2000
OPENAI_TEMPERATURE=0.7

# Anthropic Claude (선택사항)
ANTHROPIC_API_KEY=sk-ant-...

# ===================================
# 데이터베이스 설정
# ===================================

# PostgreSQL
DATABASE_URL=postgresql+asyncpg://postgres:password@localhost:5432/aiservice
# 동기 버전 (Alembic 마이그레이션용)
DATABASE_URL_SYNC=postgresql://postgres:password@localhost:5432/aiservice

# ===================================
# Redis 설정
# ===================================

REDIS_URL=redis://localhost:6379/0
REDIS_PASSWORD=

# ===================================
# 벡터 데이터베이스
# ===================================

# ChromaDB (로컬)
CHROMA_PERSIST_DIRECTORY=./data/chroma

# Pinecone (선택사항)
# PINECONE_API_KEY=...
# PINECONE_ENVIRONMENT=us-west1-gcp

# ===================================
# 애플리케이션 설정
# ===================================

# 환경 (development, production)
ENVIRONMENT=development

# 서버
HOST=0.0.0.0
PORT=8000
WORKERS=4

# 로그 레벨
LOG_LEVEL=INFO

# JWT 시크릿 (인증용)
SECRET_KEY=your-secret-key-change-in-production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# CORS
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8000

# ===================================
# 모니터링
# ===================================

# Sentry (에러 트래킹)
SENTRY_DSN=

# Prometheus
PROMETHEUS_ENABLED=true
```

#### .env.example (Git 커밋용)

```.env.example
# OpenAI API
OPENAI_API_KEY=sk-proj-your-api-key-here
OPENAI_MODEL=gpt-4-turbo-preview
OPENAI_EMBEDDING_MODEL=text-embedding-3-small

# Database
DATABASE_URL=postgresql+asyncpg://postgres:password@localhost:5432/aiservice

# Redis
REDIS_URL=redis://localhost:6379/0

# Application
ENVIRONMENT=development
SECRET_KEY=change-this-in-production
```

#### config.py (Pydantic Settings)

```python
# app/config.py
"""
애플리케이션 설정 관리

Pydantic Settings를 사용하여 환경변수를 타입 안전하게 관리합니다.
"""

from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field, validator
from typing import List


class Settings(BaseSettings):
    """
    애플리케이션 설정 클래스

    .env 파일의 환경변수를 자동으로 로드하고 검증합니다.
    """

    # ===================================
    # AI 설정
    # ===================================
    openai_api_key: str = Field(..., description="OpenAI API 키")
    openai_model: str = Field("gpt-4-turbo-preview", description="기본 모델")
    openai_embedding_model: str = Field(
        "text-embedding-3-small",
        description="임베딩 모델"
    )
    openai_max_tokens: int = Field(2000, description="최대 토큰 수")
    openai_temperature: float = Field(0.7, ge=0.0, le=2.0, description="Temperature")

    # ===================================
    # 데이터베이스 설정
    # ===================================
    database_url: str = Field(..., description="비동기 DB URL")
    database_url_sync: str = Field(..., description="동기 DB URL (마이그레이션)")

    # ===================================
    # Redis 설정
    # ===================================
    redis_url: str = Field("redis://localhost:6379/0", description="Redis URL")
    redis_password: str | None = Field(None, description="Redis 비밀번호")

    # ===================================
    # 벡터 DB 설정
    # ===================================
    chroma_persist_directory: str = Field(
        "./data/chroma",
        description="ChromaDB 저장 경로"
    )

    # ===================================
    # 애플리케이션 설정
    # ===================================
    environment: str = Field("development", description="실행 환경")
    host: str = Field("0.0.0.0", description="서버 호스트")
    port: int = Field(8000, description="서버 포트")
    workers: int = Field(4, description="Uvicorn 워커 수")
    log_level: str = Field("INFO", description="로그 레벨")

    # ===================================
    # JWT 설정
    # ===================================
    secret_key: str = Field(..., description="JWT 시크릿 키")
    algorithm: str = Field("HS256", description="JWT 알고리즘")
    access_token_expire_minutes: int = Field(30, description="토큰 만료 시간")

    # ===================================
    # CORS 설정
    # ===================================
    allowed_origins: str = Field(
        "http://localhost:3000",
        description="CORS 허용 오리진 (쉼표 구분)"
    )

    @validator("allowed_origins")
    def parse_allowed_origins(cls, v) -> List[str]:
        """쉼표로 구분된 문자열을 리스트로 변환"""
        return [origin.strip() for origin in v.split(",")]

    # ===================================
    # Pydantic Settings 설정
    # ===================================
    model_config = SettingsConfigDict(
        # .env 파일 경로
        env_file=".env",
        # .env 파일이 없어도 에러 안남
        env_file_encoding="utf-8",
        # 환경변수 우선 (대소문자 구분 안함)
        case_sensitive=False,
        # 추가 필드 허용
        extra="ignore"
    )

    # ===================================
    # 계산된 속성
    # ===================================
    @property
    def is_development(self) -> bool:
        """개발 환경인지 확인"""
        return self.environment == "development"

    @property
    def is_production(self) -> bool:
        """프로덕션 환경인지 확인"""
        return self.environment == "production"


# 싱글톤 인스턴스 생성
settings = Settings()


# 사용 예시
if __name__ == "__main__":
    print(f"Environment: {settings.environment}")
    print(f"OpenAI Model: {settings.openai_model}")
    print(f"Database: {settings.database_url}")
    print(f"Is Development: {settings.is_development}")
```

#### .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python

# 가상환경
venv/
env/
ENV/

# 환경변수
.env
.env.local

# IDE
.vscode/
.idea/
*.swp
*.swo

# 데이터베이스
*.db
*.sqlite3

# 로그
*.log
logs/

# 테스트
.pytest_cache/
.coverage
htmlcov/

# ChromaDB 데이터
data/chroma/

# OS
.DS_Store
Thumbs.db
```

### 5. 첫 번째 FastAPI 애플리케이션

#### app/main.py

```python
# app/main.py
"""
FastAPI AI 서비스 메인 애플리케이션
"""

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from contextlib import asynccontextmanager

from app.config import settings
from app.routers import chat
from loguru import logger


# ===================================
# 라이프사이클 이벤트
# ===================================

@asynccontextmanager
async def lifespan(app: FastAPI):
    """
    애플리케이션 시작/종료 이벤트

    시작 시:
    - 데이터베이스 연결
    - Redis 연결
    - 벡터 DB 초기화

    종료 시:
    - 리소스 정리
    """
    # 시작
    logger.info("🚀 Starting FastAPI AI Service...")
    logger.info(f"Environment: {settings.environment}")
    logger.info(f"OpenAI Model: {settings.openai_model}")

    # TODO: 데이터베이스 연결
    # TODO: Redis 연결
    # TODO: 벡터 DB 초기화

    yield  # 애플리케이션 실행

    # 종료
    logger.info("🛑 Shutting down FastAPI AI Service...")
    # TODO: 리소스 정리


# ===================================
# FastAPI 앱 생성
# ===================================

app = FastAPI(
    title="FastAPI AI Service",
    description="Python FastAPI로 구축한 AI 챗봇 서비스",
    version="1.0.0",
    docs_url="/docs",  # Swagger UI
    redoc_url="/redoc",  # ReDoc
    lifespan=lifespan
)


# ===================================
# 미들웨어
# ===================================

# CORS (Cross-Origin Resource Sharing)
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.allowed_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


# ===================================
# 라우터 등록
# ===================================

# 챗봇 라우터
# app.include_router(chat.router)  # TODO: 구현 후 활성화


# ===================================
# 기본 엔드포인트
# ===================================

@app.get("/")
async def root():
    """루트 엔드포인트"""
    return {
        "message": "FastAPI AI Service",
        "version": "1.0.0",
        "environment": settings.environment,
        "docs": "/docs"
    }


@app.get("/health")
async def health_check():
    """헬스 체크 엔드포인트"""
    return {
        "status": "healthy",
        "environment": settings.environment
    }


# ===================================
# 에러 핸들러
# ===================================

@app.exception_handler(Exception)
async def global_exception_handler(request, exc):
    """전역 에러 핸들러"""
    logger.error(f"Unhandled exception: {exc}")
    return JSONResponse(
        status_code=500,
        content={
            "error": "Internal Server Error",
            "message": str(exc) if settings.is_development else "An error occurred"
        }
    )


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(
        "app.main:app",
        host=settings.host,
        port=settings.port,
        reload=settings.is_development,  # 개발 모드에서만 자동 재시작
        log_level=settings.log_level.lower()
    )
```

### 6. 애플리케이션 실행

```bash
# 방법 1: Uvicorn 직접 실행 (개발 모드)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 방법 2: Python 모듈로 실행
python -m app.main

# 방법 3: main.py 직접 실행
python app/main.py
```

**실행 확인**:
```bash
# 브라우저에서 접속
http://localhost:8000          # 루트
http://localhost:8000/docs     # Swagger UI (자동 문서화)
http://localhost:8000/redoc    # ReDoc
http://localhost:8000/health   # 헬스 체크

# 또는 curl/httpie로 테스트
curl http://localhost:8000/health

# HTTPie
http :8000/health
```

**성공 메시지**:
```
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345] using WatchFiles
INFO:     Started server process [12346]
INFO:     Waiting for application startup.
🚀 Starting FastAPI AI Service...
Environment: development
OpenAI Model: gpt-4-turbo-preview
INFO:     Application startup complete.
```

### 설정 완료 체크리스트

```
□ Python 3.10+ 설치 완료
□ 가상환경 생성 및 활성화 완료
□ requirements.txt 작성 및 설치 완료
□ 프로젝트 구조 생성 완료
□ .env 파일 작성 및 API 키 설정 완료
□ config.py 작성 완료
□ main.py 작성 완료
□ 애플리케이션 시작 성공 (http://localhost:8000)
□ Swagger UI 접속 성공 (http://localhost:8000/docs)
□ 헬스 체크 성공 (http://localhost:8000/health)
```

다음 섹션에서는 실제 채팅 기능을 구현합니다!

---

## 핵심 구현 상세 설명

### 구현 전 아키텍처 복습

우리가 구현할 계층 구조:
```
사용자 요청
    ↓
[Router] ← HTTP 요청을 받아서 JSON으로 반환
    ↓
[Service] ← 비즈니스 로직 (AI 호출, 대화 저장)
    ↓
[Repository] ← 데이터베이스 CRUD
    ↓
[Database] ← 실제 데이터 저장소
```

### 구현 순서

이 가이드는 **Bottom-Up** 방식으로 진행합니다:
```
1. Models (SQLAlchemy - DB 테이블)
2. Schemas (Pydantic - 요청/응답 검증)
3. Repository (데이터 접근)
4. Service (비즈니스 로직)
5. Router (API 엔드포인트)
```

### 1. 도메인 모델 - SQLAlchemy ORM

#### SQLAlchemy란?

**일반 설명**: Python의 **ORM (Object-Relational Mapping)** - 객체를 데이터베이스 테이블로 매핑

**비유**:
- SQL: 데이터베이스와 직접 대화 (외국어)
- SQLAlchemy: 통역사 (Python ↔ SQL 자동 변환)

**장점**:
```python
# SQL 직접 작성 (힘듦)
cursor.execute("""
    SELECT * FROM conversations
    WHERE user_id = %s
    ORDER BY created_at DESC
    LIMIT 10
""", (user_id,))
results = cursor.fetchall()

# SQLAlchemy (쉬움)
conversations = session.query(Conversation)\
    .filter(Conversation.user_id == user_id)\
    .order_by(Conversation.created_at.desc())\
    .limit(10)\
    .all()
```

#### models/conversation.py

```python
# app/models/conversation.py
"""
대화 모델 정의

SQLAlchemy ORM을 사용하여 데이터베이스 테이블을 정의합니다.
"""

from sqlalchemy import Column, Integer, String, Text, DateTime, Index, JSON
from sqlalchemy.sql import func
from datetime import datetime
from typing import Optional

from app.models.base import Base  # Base 클래스


class Conversation(Base):
    """
    대화 테이블

    사용자와 AI의 한 번의 대화를 저장합니다.
    """
    __tablename__ = "conversations"

    # ===== 기본 키 =====
    id = Column(
        Integer,
        primary_key=True,
        index=True,
        comment="대화 ID (자동 증가)"
    )

    # ===== 사용자 정보 =====
    user_id = Column(
        String(255),
        nullable=False,
        index=True,
        comment="사용자 고유 ID"
    )

    session_id = Column(
        String(255),
        nullable=False,
        index=True,
        comment="세션 ID (하나의 대화 세션 구분)"
    )

    # ===== 대화 내용 =====
    user_message = Column(
        Text,
        nullable=False,
        comment="사용자 메시지"
    )

    assistant_message = Column(
        Text,
        nullable=False,
        comment="AI 응답 메시지"
    )

    # ===== 메타데이터 =====
    metadata = Column(
        JSON,
        nullable=True,
        comment="추가 메타데이터 (JSON 형식)"
    )

    # ===== 시간 정보 =====
    created_at = Column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
        comment="생성 시간"
    )

    # ===== 성능 메트릭 =====
    token_count = Column(
        Integer,
        nullable=True,
        comment="사용된 토큰 수"
    )

    response_time_ms = Column(
        Integer,
        nullable=True,
        comment="응답 시간 (밀리초)"
    )

    # ===== 인덱스 정의 =====
    __table_args__ = (
        # 복합 인덱스: 사용자별 최근 대화 조회 최적화
        Index("idx_user_created", "user_id", "created_at"),
        # 세션별 대화 조회 최적화
        Index("idx_session", "session_id"),
    )

    def __repr__(self):
        return f"<Conversation(id={self.id}, user_id={self.user_id}, created_at={self.created_at})>"


class ConversationSummary(Base):
    """
    대화 요약 테이블

    오래된 대화를 요약하여 저장합니다.
    """
    __tablename__ = "conversation_summaries"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(String(255), nullable=False, index=True)
    session_id = Column(String(255), nullable=False)
    summary = Column(Text, nullable=False, comment="대화 요약")

    summary_period_start = Column(
        DateTime(timezone=True),
        comment="요약 시작 시간"
    )

    summary_period_end = Column(
        DateTime(timezone=True),
        comment="요약 종료 시간"
    )

    message_count = Column(
        Integer,
        comment="요약된 메시지 수"
    )

    created_at = Column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False
    )

    def __repr__(self):
        return f"<ConversationSummary(id={self.id}, message_count={self.message_count})>"
```

#### models/base.py - Base 클래스

```python
# app/models/base.py
"""
SQLAlchemy Base 클래스
"""

from sqlalchemy.ext.declarative import declarative_base

# 모든 모델의 기본 클래스
Base = declarative_base()
```

#### models/__init__.py - 모델 내보내기

```python
# app/models/__init__.py
"""
모든 모델을 여기서 import

Alembic 마이그레이션이 모든 모델을 인식하도록 합니다.
"""

from app.models.base import Base
from app.models.conversation import Conversation, ConversationSummary

__all__ = ["Base", "Conversation", "ConversationSummary"]
```

### 2. Pydantic 스키마 - 요청/응답 검증

#### schemas/chat.py

```python
# app/schemas/chat.py
"""
채팅 관련 Pydantic 스키마

API 요청/응답의 데이터 구조를 정의하고 자동 검증합니다.
"""

from pydantic import BaseModel, Field, validator
from typing import Optional, Dict, Any
from datetime import datetime


class ChatRequest(BaseModel):
    """
    채팅 요청 스키마

    사용자가 보내는 메시지를 검증합니다.
    """
    message: str = Field(
        ...,  # 필수 필드
        min_length=1,
        max_length=5000,
        description="사용자 메시지 (1~5000자)",
        examples=["오늘 날씨 어때?"]
    )

    user_id: str = Field(
        ...,
        min_length=1,
        max_length=255,
        description="사용자 고유 ID",
        examples=["user123"]
    )

    session_id: Optional[str] = Field(
        None,
        max_length=255,
        description="세션 ID (없으면 자동 생성)",
        examples=["session-abc-123"]
    )

    temperature: float = Field(
        0.7,
        ge=0.0,
        le=2.0,
        description="AI 응답의 창의성 (0.0~2.0)"
    )

    max_tokens: Optional[int] = Field(
        None,
        gt=0,
        le=4000,
        description="최대 토큰 수"
    )

    @validator("message")
    def message_must_not_be_empty(cls, v: str) -> str:
        """빈 문자열 검증"""
        if not v.strip():
            raise ValueError("메시지가 비어있습니다")
        return v.strip()

    class Config:
        json_schema_extra = {
            "example": {
                "message": "Python과 Java의 차이점은?",
                "user_id": "user123",
                "session_id": "session-456",
                "temperature": 0.7
            }
        }


class ChatResponse(BaseModel):
    """
    채팅 응답 스키마

    AI의 응답을 클라이언트에 반환합니다.
    """
    message: str = Field(
        ...,
        description="AI 응답 메시지"
    )

    user_id: str = Field(
        ...,
        description="사용자 ID"
    )

    session_id: str = Field(
        ...,
        description="세션 ID"
    )

    metadata: Optional[Dict[str, Any]] = Field(
        None,
        description="응답 메타데이터 (토큰, 모델 등)"
    )

    created_at: datetime = Field(
        default_factory=datetime.now,
        description="응답 생성 시간"
    )

    class Config:
        json_schema_extra = {
            "example": {
                "message": "Python은 동적 타입, Java는 정적 타입 언어입니다...",
                "user_id": "user123",
                "session_id": "session-456",
                "metadata": {
                    "model": "gpt-4-turbo-preview",
                    "tokens": 256,
                    "response_time_ms": 1200
                },
                "created_at": "2024-02-08T10:30:00Z"
            }
        }


class ConversationHistory(BaseModel):
    """
    대화 히스토리 항목
    """
    id: int
    user_message: str
    assistant_message: str
    created_at: datetime

    class Config:
        from_attributes = True  # SQLAlchemy 모델에서 변환 가능


class ConversationHistoryResponse(BaseModel):
    """
    대화 히스토리 응답
    """
    user_id: str
    session_id: str
    conversations: list[ConversationHistory]
    total_count: int

    class Config:
        json_schema_extra = {
            "example": {
                "user_id": "user123",
                "session_id": "session-456",
                "conversations": [
                    {
                        "id": 1,
                        "user_message": "안녕하세요",
                        "assistant_message": "안녕하세요! 무엇을 도와드릴까요?",
                        "created_at": "2024-02-08T10:30:00Z"
                    }
                ],
                "total_count": 1
            }
        }
```

### 3. 데이터베이스 연결 및 세션 관리

#### dependencies/database.py

```python
# app/dependencies/database.py
"""
데이터베이스 연결 및 세션 관리

SQLAlchemy 비동기 엔진을 사용합니다.
"""

from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import sessionmaker
from typing import AsyncGenerator

from app.config import settings


# ===================================
# 비동기 엔진 생성
# ===================================

# 비동기 PostgreSQL 엔진
engine = create_async_engine(
    settings.database_url,
    echo=settings.is_development,  # 개발 모드에서 SQL 로그 출력
    pool_size=20,  # 커넥션 풀 크기
    max_overflow=10,  # 추가 커넥션 수
    pool_pre_ping=True,  # 연결 체크
)

# 비동기 세션 팩토리
AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,  # 커밋 후에도 객체 사용 가능
    autocommit=False,
    autoflush=False,
)


# ===================================
# 의존성 함수
# ===================================

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """
    데이터베이스 세션 제공

    FastAPI Depends에서 사용:
    @app.get("/")
    async def route(db: AsyncSession = Depends(get_db)):
        ...

    자동으로:
    1. 세션 생성
    2. 함수에 전달
    3. 함수 종료 후 세션 닫기
    """
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()  # 성공 시 커밋
        except Exception:
            await session.rollback()  # 에러 시 롤백
            raise
        finally:
            await session.close()  # 항상 세션 닫기


# ===================================
# 데이터베이스 초기화
# ===================================

async def init_db():
    """
    데이터베이스 테이블 생성

    주의: 프로덕션에서는 Alembic 마이그레이션 사용!
    """
    from app.models import Base

    async with engine.begin() as conn:
        # 모든 테이블 생성
        await conn.run_sync(Base.metadata.create_all)


async def close_db():
    """
    데이터베이스 연결 종료
    """
    await engine.dispose()
```

### 4. Repository 패턴 - 데이터 접근 계층

#### repositories/conversation.py

```python
# app/repositories/conversation.py
"""
대화 Repository

데이터베이스 CRUD 작업을 추상화합니다.
"""

from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, desc, and_
from typing import List, Optional
from datetime import datetime, timedelta

from app.models.conversation import Conversation, ConversationSummary


class ConversationRepository:
    """대화 데이터 접근 객체"""

    def __init__(self, db: AsyncSession):
        self.db = db

    async def create(self, conversation: Conversation) -> Conversation:
        """대화 생성"""
        self.db.add(conversation)
        await self.db.flush()  # ID 생성을 위해 flush
        await self.db.refresh(conversation)
        return conversation

    async def get_by_id(self, conversation_id: int) -> Optional[Conversation]:
        """ID로 대화 조회"""
        result = await self.db.execute(
            select(Conversation).where(Conversation.id == conversation_id)
        )
        return result.scalar_one_or_none()

    async def get_recent_by_user(
        self,
        user_id: str,
        session_id: str,
        limit: int = 10
    ) -> List[Conversation]:
        """
        사용자의 최근 대화 조회

        Args:
            user_id: 사용자 ID
            session_id: 세션 ID
            limit: 최대 개수

        Returns:
            최근 대화 리스트 (최신순)
        """
        result = await self.db.execute(
            select(Conversation)
            .where(
                and_(
                    Conversation.user_id == user_id,
                    Conversation.session_id == session_id
                )
            )
            .order_by(desc(Conversation.created_at))
            .limit(limit)
        )
        conversations = result.scalars().all()

        # 오래된 순서로 변환 (대화 흐름 유지)
        return list(reversed(conversations))

    async def get_recent_by_user_all_sessions(
        self,
        user_id: str,
        limit: int = 50
    ) -> List[Conversation]:
        """모든 세션의 최근 대화 조회"""
        result = await self.db.execute(
            select(Conversation)
            .where(Conversation.user_id == user_id)
            .order_by(desc(Conversation.created_at))
            .limit(limit)
        )
        return result.scalars().all()

    async def delete_old_conversations(
        self,
        before: datetime
    ) -> int:
        """
        오래된 대화 삭제

        Args:
            before: 이 시간 이전의 대화 삭제

        Returns:
            삭제된 개수
        """
        result = await self.db.execute(
            select(Conversation).where(Conversation.created_at < before)
        )
        conversations = result.scalars().all()

        for conv in conversations:
            await self.db.delete(conv)

        return len(conversations)

    async def count_by_user(self, user_id: str, session_id: str) -> int:
        """사용자의 대화 개수"""
        result = await self.db.execute(
            select(Conversation)
            .where(
                and_(
                    Conversation.user_id == user_id,
                    Conversation.session_id == session_id
                )
            )
        )
        return len(result.scalars().all())


class ConversationSummaryRepository:
    """대화 요약 데이터 접근 객체"""

    def __init__(self, db: AsyncSession):
        self.db = db

    async def create(self, summary: ConversationSummary) -> ConversationSummary:
        """요약 생성"""
        self.db.add(summary)
        await self.db.flush()
        await self.db.refresh(summary)
        return summary

    async def get_latest_by_session(
        self,
        user_id: str,
        session_id: str
    ) -> Optional[ConversationSummary]:
        """세션의 최신 요약 조회"""
        result = await self.db.execute(
            select(ConversationSummary)
            .where(
                and_(
                    ConversationSummary.user_id == user_id,
                    ConversationSummary.session_id == session_id
                )
            )
            .order_by(desc(ConversationSummary.created_at))
            .limit(1)
        )
        return result.scalar_one_or_none()

    async def get_recent_by_user(
        self,
        user_id: str,
        limit: int = 5
    ) -> List[ConversationSummary]:
        """사용자의 최근 요약 조회"""
        result = await self.db.execute(
            select(ConversationSummary)
            .where(ConversationSummary.user_id == user_id)
            .order_by(desc(ConversationSummary.created_at))
            .limit(limit)
        )
        return result.scalars().all()
```

### 5. Service Layer - 비즈니스 로직

#### services/chat_service.py

```python
# app/services/chat_service.py
"""
채팅 서비스

AI 채팅의 핵심 비즈니스 로직을 처리합니다.
"""

from openai import AsyncOpenAI
from sqlalchemy.ext.asyncio import AsyncSession
from typing import List, Dict, Any
import uuid
import time

from app.models.conversation import Conversation, ConversationSummary
from app.repositories.conversation import ConversationRepository, ConversationSummaryRepository
from app.schemas.chat import ChatRequest, ChatResponse
from app.config import settings
from loguru import logger


class ChatService:
    """채팅 서비스"""

    def __init__(self, db: AsyncSession):
        self.db = db
        self.conv_repo = ConversationRepository(db)
        self.summary_repo = ConversationSummaryRepository(db)

        # OpenAI 클라이언트
        self.openai_client = AsyncOpenAI(api_key=settings.openai_api_key)

    async def chat(self, request: ChatRequest) -> ChatResponse:
        """
        채팅 처리

        1. 대화 히스토리 로드
        2. 요약 로드
        3. AI 호출
        4. 대화 저장
        5. 응답 반환
        """
        start_time = time.time()

        # 세션 ID 생성 (없으면)
        session_id = request.session_id or str(uuid.uuid4())

        # 1. 대화 히스토리 로드
        history = await self.conv_repo.get_recent_by_user(
            user_id=request.user_id,
            session_id=session_id,
            limit=10
        )

        # 2. 요약 로드
        summary = await self.summary_repo.get_latest_by_session(
            user_id=request.user_id,
            session_id=session_id
        )

        # 3. 메시지 구성
        messages = self._build_messages(history, summary, request.message)

        # 4. OpenAI API 호출
        try:
            response = await self.openai_client.chat.completions.create(
                model=settings.openai_model,
                messages=messages,
                temperature=request.temperature,
                max_tokens=request.max_tokens or settings.openai_max_tokens
            )

            assistant_message = response.choices[0].message.content
            token_count = response.usage.total_tokens

        except Exception as e:
            logger.error(f"OpenAI API 오류: {e}")
            raise

        # 5. 응답 시간 계산
        response_time_ms = int((time.time() - start_time) * 1000)

        # 6. 대화 저장
        conversation = Conversation(
            user_id=request.user_id,
            session_id=session_id,
            user_message=request.message,
            assistant_message=assistant_message,
            token_count=token_count,
            response_time_ms=response_time_ms,
            metadata={
                "model": settings.openai_model,
                "temperature": request.temperature
            }
        )

        await self.conv_repo.create(conversation)
        await self.db.commit()

        # 7. 요약 체크 (50개 이상이면 요약)
        await self._check_and_summarize(request.user_id, session_id)

        # 8. 응답 반환
        return ChatResponse(
            message=assistant_message,
            user_id=request.user_id,
            session_id=session_id,
            metadata={
                "model": settings.openai_model,
                "tokens": token_count,
                "response_time_ms": response_time_ms
            }
        )

    def _build_messages(
        self,
        history: List[Conversation],
        summary: ConversationSummary | None,
        current_message: str
    ) -> List[Dict[str, str]]:
        """
        OpenAI 메시지 구성

        시스템 프롬프트 + 요약 + 대화 히스토리 + 현재 메시지
        """
        messages = []

        # 시스템 프롬프트
        system_prompt = "당신은 도움이 되는 AI 어시스턴트입니다."

        # 요약 추가
        if summary:
            system_prompt += f"\n\n이전 대화 요약:\n{summary.summary}"

        messages.append({
            "role": "system",
            "content": system_prompt
        })

        # 대화 히스토리 추가
        for conv in history:
            messages.append({
                "role": "user",
                "content": conv.user_message
            })
            messages.append({
                "role": "assistant",
                "content": conv.assistant_message
            })

        # 현재 메시지 추가
        messages.append({
            "role": "user",
            "content": current_message
        })

        return messages

    async def _check_and_summarize(self, user_id: str, session_id: str):
        """
        대화 요약 체크

        50개 이상의 대화가 쌓이면 요약 생성
        """
        count = await self.conv_repo.count_by_user(user_id, session_id)

        if count >= 50:
            logger.info(f"대화 요약 시작: {user_id}, {session_id}")
            await self._summarize_conversations(user_id, session_id)

    async def _summarize_conversations(self, user_id: str, session_id: str):
        """
        대화 요약 생성

        1. 모든 대화 가져오기
        2. OpenAI로 요약
        3. 요약 저장
        4. 오래된 대화 삭제 (최근 10개만 유지)
        """
        # 1. 모든 대화 가져오기
        conversations = await self.conv_repo.get_recent_by_user(
            user_id=user_id,
            session_id=session_id,
            limit=100
        )

        if not conversations:
            return

        # 2. 대화 텍스트 구성
        conversation_text = "\n\n".join([
            f"User: {conv.user_message}\nAssistant: {conv.assistant_message}"
            for conv in conversations
        ])

        # 3. OpenAI로 요약
        try:
            response = await self.openai_client.chat.completions.create(
                model=settings.openai_model,
                messages=[
                    {
                        "role": "system",
                        "content": "다음 대화 내용을 핵심 포인트 중심으로 요약해주세요."
                    },
                    {
                        "role": "user",
                        "content": conversation_text
                    }
                ],
                temperature=0.5,
                max_tokens=500
            )

            summary_text = response.choices[0].message.content

        except Exception as e:
            logger.error(f"요약 생성 오류: {e}")
            return

        # 4. 요약 저장
        summary = ConversationSummary(
            user_id=user_id,
            session_id=session_id,
            summary=summary_text,
            summary_period_start=conversations[0].created_at,
            summary_period_end=conversations[-1].created_at,
            message_count=len(conversations)
        )

        await self.summary_repo.create(summary)
        await self.db.commit()

        logger.info(f"대화 요약 완료: {user_id}, {len(conversations)}개 메시지")

    async def get_conversation_history(
        self,
        user_id: str,
        session_id: str,
        limit: int = 20
    ) -> List[Conversation]:
        """대화 히스토리 조회"""
        return await self.conv_repo.get_recent_by_user(
            user_id=user_id,
            session_id=session_id,
            limit=limit
        )
```

### 6. Router - API 엔드포인트

#### routers/chat.py

```python
# app/routers/chat.py
"""
채팅 API 라우터

채팅 관련 엔드포인트를 정의합니다.
"""

from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from typing import Annotated

from app.dependencies.database import get_db
from app.services.chat_service import ChatService
from app.schemas.chat import (
    ChatRequest,
    ChatResponse,
    ConversationHistoryResponse,
    ConversationHistory
)
from loguru import logger


# ===================================
# 라우터 생성
# ===================================

router = APIRouter(
    prefix="/api/v1/chat",
    tags=["chat"],
    responses={
        404: {"description": "Not found"},
        500: {"description": "Internal server error"}
    }
)


# ===================================
# 의존성
# ===================================

async def get_chat_service(
    db: Annotated[AsyncSession, Depends(get_db)]
) -> ChatService:
    """채팅 서비스 의존성"""
    return ChatService(db)


# ===================================
# 엔드포인트
# ===================================

@router.post(
    "",
    response_model=ChatResponse,
    status_code=status.HTTP_200_OK,
    summary="채팅 메시지 전송",
    description="""
    AI 챗봇에 메시지를 전송하고 응답을 받습니다.

    ## 처리 과정
    1. 대화 히스토리 로드
    2. 이전 요약 로드
    3. OpenAI API 호출
    4. 대화 저장
    5. 응답 반환

    ## 예시
    ```json
    {
        "message": "Python과 Java의 차이점은?",
        "user_id": "user123",
        "temperature": 0.7
    }
    ```
    """
)
async def chat(
    request: ChatRequest,
    chat_service: Annotated[ChatService, Depends(get_chat_service)]
):
    """
    채팅 엔드포인트

    사용자 메시지를 받아 AI 응답을 반환합니다.
    """
    try:
        logger.info(f"채팅 요청: user={request.user_id}, message={request.message[:50]}...")

        response = await chat_service.chat(request)

        logger.info(f"채팅 응답: user={request.user_id}, tokens={response.metadata.get('tokens')}")

        return response

    except Exception as e:
        logger.error(f"채팅 오류: {e}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"채팅 처리 중 오류가 발생했습니다: {str(e)}"
        )


@router.get(
    "/history/{user_id}/{session_id}",
    response_model=ConversationHistoryResponse,
    summary="대화 히스토리 조회",
    description="사용자의 대화 히스토리를 조회합니다."
)
async def get_history(
    user_id: str,
    session_id: str,
    limit: int = 20,
    chat_service: Annotated[ChatService, Depends(get_chat_service)]
):
    """
    대화 히스토리 조회

    특정 세션의 대화 히스토리를 반환합니다.
    """
    try:
        conversations = await chat_service.get_conversation_history(
            user_id=user_id,
            session_id=session_id,
            limit=limit
        )

        return ConversationHistoryResponse(
            user_id=user_id,
            session_id=session_id,
            conversations=[
                ConversationHistory.model_validate(conv)
                for conv in conversations
            ],
            total_count=len(conversations)
        )

    except Exception as e:
        logger.error(f"히스토리 조회 오류: {e}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="히스토리 조회 중 오류가 발생했습니다"
        )


@router.delete(
    "/history/{user_id}/{session_id}",
    status_code=status.HTTP_204_NO_CONTENT,
    summary="대화 히스토리 삭제",
    description="사용자의 특정 세션 대화를 삭제합니다."
)
async def delete_history(
    user_id: str,
    session_id: str,
    chat_service: Annotated[ChatService, Depends(get_chat_service)]
):
    """
    대화 히스토리 삭제

    GDPR 등 개인정보 삭제 요청에 대응합니다.
    """
    try:
        # TODO: 구현
        logger.info(f"히스토리 삭제 요청: user={user_id}, session={session_id}")
        return

    except Exception as e:
        logger.error(f"히스토리 삭제 오류: {e}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="히스토리 삭제 중 오류가 발생했습니다"
        )
```

### 7. main.py 업데이트 - 라우터 등록

```python
# app/main.py 업데이트

# ... (기존 코드)

# 라우터 등록
from app.routers import chat

app.include_router(chat.router)

# ... (나머지 코드)
```

### 8. 데이터베이스 마이그레이션 (Alembic)

#### Alembic 초기화

```bash
# Alembic 설치 (requirements.txt에 포함됨)
pip install alembic

# Alembic 초기화
alembic init alembic
```

#### alembic.ini 수정

```ini
# alembic.ini
# ...
sqlalchemy.url = %(DATABASE_URL_SYNC)s  # 환경변수에서 로드
# ...
```

#### alembic/env.py 수정

```python
# alembic/env.py
from app.models import Base
from app.config import settings

# ...

config.set_main_option("sqlalchemy.url", settings.database_url_sync)

target_metadata = Base.metadata

# ...
```

#### 첫 마이그레이션 생성

```bash
# 마이그레이션 파일 생성
alembic revision --autogenerate -m "Create conversations table"

# 마이그레이션 적용
alembic upgrade head

# 마이그레이션 롤백 (필요시)
alembic downgrade -1
```

### 9. 전체 테스트

#### 애플리케이션 실행

```bash
# Docker로 PostgreSQL 실행
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=aiservice \
  -p 5432:5432 \
  postgres:16

# 애플리케이션 실행
uvicorn app.main:app --reload
```

#### API 테스트

```bash
# 1. 헬스 체크
curl http://localhost:8000/health

# 2. 채팅 테스트
curl -X POST http://localhost:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "안녕하세요!",
    "user_id": "test_user",
    "temperature": 0.7
  }'

# 3. 히스토리 조회
curl http://localhost:8000/api/v1/chat/history/test_user/{session_id}

# 4. Swagger UI 접속
open http://localhost:8000/docs
```

### 구현 완료 체크리스트

```
□ SQLAlchemy 모델 작성 완료
□ Pydantic 스키마 작성 완료
□ Repository 작성 완료
□ Service 작성 완료
□ Router 작성 완료
□ 데이터베이스 연결 완료
□ Alembic 마이그레이션 완료
□ API 테스트 성공
□ Swagger UI에서 문서 확인
```

---

## 장기 메모리 관리 완전 가이드

### Python/FastAPI의 메모리 관리 접근법

#### 1. **SQLAlchemy ORM의 강력한 기능**

**세션 관리 - 컨텍스트 매니저**:

```python
# 자동 커밋/롤백
async with AsyncSessionLocal() as session:
    try:
        # 작업 수행
        conversation = Conversation(...)
        session.add(conversation)
        await session.commit()  # 성공
    except Exception:
        await session.rollback()  # 실패 시 자동 롤백
        raise
```

**Lazy Loading vs Eager Loading**:

```python
# ❌ N+1 문제
conversations = await session.execute(
    select(Conversation).where(Conversation.user_id == user_id)
)
for conv in conversations:
    # 각 conversation마다 DB 쿼리 발생!
    print(conv.summary)  # N+1 문제


# ✅ Eager Loading
from sqlalchemy.orm import selectinload

conversations = await session.execute(
    select(Conversation)
    .options(selectinload(Conversation.summary))  # 미리 로드
    .where(Conversation.user_id == user_id)
)
# 한 번의 쿼리로 모든 데이터 로드!
```

#### 2. **Redis 캐싱 전략**

**설치 및 설정**:

```python
# app/dependencies/redis.py
import redis.asyncio as redis
from typing import Optional
import json

from app.config import settings


class RedisClient:
    """Redis 클라이언트"""

    def __init__(self):
        self.redis = redis.from_url(
            settings.redis_url,
            password=settings.redis_password,
            encoding="utf-8",
            decode_responses=True
        )

    async def get(self, key: str) -> Optional[str]:
        """값 가져오기"""
        return await self.redis.get(key)

    async def set(
        self,
        key: str,
        value: str,
        expire: int = 3600
    ):
        """값 저장 (기본 1시간 TTL)"""
        await self.redis.set(key, value, ex=expire)

    async def delete(self, key: str):
        """값 삭제"""
        await self.redis.delete(key)

    async def get_json(self, key: str) -> Optional[dict]:
        """JSON 가져오기"""
        value = await self.get(key)
        return json.loads(value) if value else None

    async def set_json(
        self,
        key: str,
        value: dict,
        expire: int = 3600
    ):
        """JSON 저장"""
        await self.set(key, json.dumps(value), expire)


# 싱글톤 인스턴스
redis_client = RedisClient()
```

**캐싱 적용**:

```python
# app/services/chat_service.py 수정

async def get_conversation_history(
    self,
    user_id: str,
    session_id: str,
    limit: int = 10
) -> List[Conversation]:
    """
    대화 히스토리 조회 (캐싱)

    1. Redis 캐시 확인
    2. 캐시 미스 시 DB 조회
    3. 캐시 저장
    """
    from app.dependencies.redis import redis_client

    # 캐시 키
    cache_key = f"history:{user_id}:{session_id}:{limit}"

    # 1. 캐시 확인
    cached = await redis_client.get_json(cache_key)
    if cached:
        logger.info(f"캐시 히트: {cache_key}")
        # JSON → Conversation 객체 변환
        return [Conversation(**conv) for conv in cached]

    # 2. DB 조회
    logger.info(f"캐시 미스: {cache_key}")
    conversations = await self.conv_repo.get_recent_by_user(
        user_id=user_id,
        session_id=session_id,
        limit=limit
    )

    # 3. 캐시 저장
    await redis_client.set_json(
        cache_key,
        [conv.__dict__ for conv in conversations],
        expire=600  # 10분
    )

    return conversations
```

#### 3. **벡터 메모리 - ChromaDB 통합**

**ChromaDB 설정**:

```python
# app/integrations/vector_store.py
"""
벡터 스토어 통합

ChromaDB를 사용하여 임베딩 벡터를 저장하고 검색합니다.
"""

import chromadb
from chromadb.config import Settings
from typing import List, Dict
import uuid

from app.config import settings
from loguru import logger


class VectorStoreService:
    """벡터 스토어 서비스"""

    def __init__(self):
        # ChromaDB 클라이언트
        self.client = chromadb.Client(Settings(
            chroma_db_impl="duckdb+parquet",
            persist_directory=settings.chroma_persist_directory
        ))

        # 컬렉션 생성 또는 가져오기
        self.collection = self.client.get_or_create_collection(
            name="conversations",
            metadata={"description": "사용자 대화 임베딩"}
        )

        logger.info("ChromaDB 초기화 완료")

    async def add_conversation(
        self,
        user_id: str,
        session_id: str,
        content: str,
        metadata: Dict = None
    ):
        """
        대화를 벡터로 저장

        Args:
            user_id: 사용자 ID
            session_id: 세션 ID
            content: 대화 내용
            metadata: 추가 메타데이터
        """
        doc_id = f"{user_id}_{session_id}_{uuid.uuid4()}"

        # 메타데이터 구성
        doc_metadata = {
            "user_id": user_id,
            "session_id": session_id,
            **(metadata or {})
        }

        # ChromaDB가 자동으로 임베딩 생성
        self.collection.add(
            documents=[content],
            metadatas=[doc_metadata],
            ids=[doc_id]
        )

        logger.debug(f"벡터 저장: {doc_id}")

    async def search_relevant_memories(
        self,
        user_id: str,
        query: str,
        limit: int = 3
    ) -> List[str]:
        """
        관련 메모리 검색

        Args:
            user_id: 사용자 ID
            query: 검색 쿼리
            limit: 최대 개수

        Returns:
            관련 대화 리스트
        """
        # 유사도 검색
        results = self.collection.query(
            query_texts=[query],
            where={"user_id": user_id},
            n_results=limit
        )

        if not results["documents"]:
            return []

        # 문서 반환
        documents = results["documents"][0]

        logger.debug(f"벡터 검색: {len(documents)}개 발견")

        return documents

    async def delete_user_memories(self, user_id: str):
        """사용자의 모든 메모리 삭제"""
        self.collection.delete(
            where={"user_id": user_id}
        )

        logger.info(f"사용자 메모리 삭제: {user_id}")


# 싱글톤 인스턴스
vector_store = VectorStoreService()
```

**ChatService에 벡터 검색 통합**:

```python
# app/services/chat_service.py 수정

from app.integrations.vector_store import vector_store

class ChatService:
    # ...

    async def chat(self, request: ChatRequest) -> ChatResponse:
        # ...

        # 벡터 메모리 검색
        relevant_memories = await vector_store.search_relevant_memories(
            user_id=request.user_id,
            query=request.message,
            limit=3
        )

        # 메시지 구성 시 관련 메모리 추가
        messages = self._build_messages(
            history,
            summary,
            relevant_memories,  # 추가!
            request.message
        )

        # ... (나머지 코드)

        # 대화를 벡터로 저장 (비동기)
        conversation_text = f"User: {request.message}\nAssistant: {assistant_message}"
        await vector_store.add_conversation(
            user_id=request.user_id,
            session_id=session_id,
            content=conversation_text
        )

        # ...

    def _build_messages(
        self,
        history: List[Conversation],
        summary: ConversationSummary | None,
        relevant_memories: List[str],  # 추가!
        current_message: str
    ) -> List[Dict[str, str]]:
        """OpenAI 메시지 구성"""
        messages = []

        # 시스템 프롬프트
        system_prompt = "당신은 도움이 되는 AI 어시스턴트입니다."

        # 요약 추가
        if summary:
            system_prompt += f"\n\n이전 대화 요약:\n{summary.summary}"

        # 관련 메모리 추가
        if relevant_memories:
            system_prompt += "\n\n관련된 과거 대화:"
            for memory in relevant_memories:
                system_prompt += f"\n- {memory}"

        messages.append({
            "role": "system",
            "content": system_prompt
        })

        # ... (나머지 동일)
```

### 메모리 계층 구조 완성

```
[사용자 질문]
      ↓
[1단계: Redis 캐시 확인]
  - 최근 10개 대화 (5ms)
  - 캐시 히트: 즉시 반환
  - 캐시 미스: 다음 단계
      ↓
[2단계: PostgreSQL 조회]
  - 대화 히스토리 (25ms)
  - 요약 정보 (10ms)
      ↓
[3단계: ChromaDB 벡터 검색]
  - 의미적으로 유사한 과거 대화 (50ms)
  - 유사도 0.7 이상만 선택
      ↓
[4단계: 컨텍스트 통합]
  - Redis 캐시 + DB 데이터 + 벡터 메모리
      ↓
[5단계: OpenAI API 호출]
  - 모든 컨텍스트와 함께 전송
      ↓
[6단계: 응답 저장 및 캐시 업데이트]
```

### 메모리 정리 스케줄러

```python
# app/tasks/cleanup.py
"""
주기적인 데이터 정리 작업
"""

from apscheduler.schedulers.asyncio import AsyncIOScheduler
from datetime import datetime, timedelta
from loguru import logger

from app.dependencies.database import AsyncSessionLocal
from app.repositories.conversation import ConversationRepository


# 스케줄러 생성
scheduler = AsyncIOScheduler()


async def cleanup_old_conversations():
    """
    오래된 대화 삭제

    3개월 이상 된 대화를 삭제합니다.
    """
    logger.info("대화 정리 작업 시작")

    async with AsyncSessionLocal() as session:
        repo = ConversationRepository(session)

        # 3개월 전 시간
        threshold = datetime.now() - timedelta(days=90)

        # 삭제
        deleted_count = await repo.delete_old_conversations(threshold)
        await session.commit()

        logger.info(f"대화 정리 완료: {deleted_count}개 삭제")


# 스케줄 등록
def init_scheduler():
    """스케줄러 초기화"""

    # 매일 새벽 3시에 실행
    scheduler.add_job(
        cleanup_old_conversations,
        "cron",
        hour=3,
        minute=0
    )

    scheduler.start()
    logger.info("스케줄러 시작")


# main.py에서 호출
# from app.tasks.cleanup import init_scheduler
# init_scheduler()
```

---

## 성능 최적화 전략

### 1. 비동기 처리 최적화

**병렬 처리**:

```python
import asyncio

# ❌ 순차 처리 (느림)
async def slow_processing():
    history = await get_history()  # 100ms
    summary = await get_summary()  # 100ms
    vector = await search_vector()  # 100ms
    # 총 300ms

# ✅ 병렬 처리 (빠름)
async def fast_processing():
    history, summary, vector = await asyncio.gather(
        get_history(),  # 동시 실행
        get_summary(),  # 동시 실행
        search_vector()  # 동시 실행
    )
    # 총 100ms (가장 느린 작업 기준)
```

**ChatService 최적화**:

```python
async def chat(self, request: ChatRequest) -> ChatResponse:
    # 병렬로 데이터 로드
    history, summary, relevant_memories = await asyncio.gather(
        self.conv_repo.get_recent_by_user(request.user_id, session_id, 10),
        self.summary_repo.get_latest_by_session(request.user_id, session_id),
        vector_store.search_relevant_memories(request.user_id, request.message, 3)
    )

    # ... (나머지 처리)
```

### 2. 데이터베이스 최적화

**인덱스 활용**:

```python
# 이미 설정된 인덱스 확인
CREATE INDEX idx_user_created ON conversations(user_id, created_at);
CREATE INDEX idx_session ON conversations(session_id);

# 쿼리 실행 계획 확인
EXPLAIN ANALYZE
SELECT * FROM conversations
WHERE user_id = 'user123'
ORDER BY created_at DESC
LIMIT 10;
```

**커넥션 풀 튜닝**:

```python
# app/dependencies/database.py
engine = create_async_engine(
    settings.database_url,
    pool_size=20,  # 기본 커넥션 수
    max_overflow=10,  # 추가 커넥션 수
    pool_pre_ping=True,  # 연결 체크
    pool_recycle=3600,  # 1시간마다 재연결
)
```

### 3. 응답 스트리밍

**실시간 스트리밍**:

```python
# app/routers/chat.py
from fastapi.responses import StreamingResponse

@router.post("/chat/stream")
async def chat_stream(
    request: ChatRequest,
    chat_service: Annotated[ChatService, Depends(get_chat_service)]
):
    """
    스트리밍 채팅

    AI 응답을 실시간으로 스트리밍합니다.
    """

    async def generate():
        # OpenAI 스트리밍
        stream = await openai_client.chat.completions.create(
            model=settings.openai_model,
            messages=messages,
            stream=True  # 스트리밍 활성화
        )

        async for chunk in stream:
            if chunk.choices[0].delta.content:
                content = chunk.choices[0].delta.content
                yield f"data: {content}\n\n"

    return StreamingResponse(
        generate(),
        media_type="text/event-stream"
    )
```

---

## 프로덕션 배포 실전 가이드

### 1. Dockerfile

```dockerfile
# Dockerfile
# Multi-stage build로 이미지 크기 최소화

# ===================================
# Stage 1: Builder
# ===================================
FROM python:3.11-slim as builder

WORKDIR /app

# 시스템 의존성 설치
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Python 의존성 설치
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt


# ===================================
# Stage 2: Runtime
# ===================================
FROM python:3.11-slim

WORKDIR /app

# 시스템 의존성
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Python 의존성 복사
COPY --from=builder /root/.local /root/.local

# 애플리케이션 코드 복사
COPY . .

# PATH 설정
ENV PATH=/root/.local/bin:$PATH

# 비root 사용자 생성
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

# 포트 노출
EXPOSE 8000

# 헬스체크
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')"

# 실행
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 2. docker-compose.yml

```yaml
version: '3.8'

services:
  # FastAPI 애플리케이션
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - DATABASE_URL=postgresql+asyncpg://postgres:password@postgres:5432/aiservice
      - REDIS_URL=redis://redis:6379/0
      - ENVIRONMENT=production
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # PostgreSQL
  postgres:
    image: pgvector/pgvector:pg16
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=aiservice
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network

  # Nginx (리버스 프록시)
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### 3. 환경변수 관리

**production.env**:
```env
# 프로덕션 환경 변수
ENVIRONMENT=production
OPENAI_API_KEY=sk-proj-...
DATABASE_URL=postgresql+asyncpg://user:pass@db-host:5432/aiservice
REDIS_URL=redis://redis-host:6379/0
SECRET_KEY=super-secret-production-key
LOG_LEVEL=INFO
SENTRY_DSN=https://...@sentry.io/...
```

**실행**:
```bash
docker-compose --env-file production.env up -d
```

---

## 모니터링 및 운영

### 1. Prometheus 메트릭

```python
# app/monitoring/metrics.py
from prometheus_client import Counter, Histogram, Gauge
import time

# 메트릭 정의
chat_requests_total = Counter(
    "chat_requests_total",
    "Total number of chat requests",
    ["user_id", "status"]
)

chat_duration_seconds = Histogram(
    "chat_duration_seconds",
    "Chat request duration in seconds",
    buckets=[0.1, 0.5, 1.0, 2.0, 5.0, 10.0]
)

active_sessions = Gauge(
    "active_sessions",
    "Number of active chat sessions"
)

# 사용
async def chat(request: ChatRequest):
    start_time = time.time()

    try:
        response = await chat_service.chat(request)
        chat_requests_total.labels(user_id=request.user_id, status="success").inc()
        return response

    except Exception as e:
        chat_requests_total.labels(user_id=request.user_id, status="error").inc()
        raise

    finally:
        duration = time.time() - start_time
        chat_duration_seconds.observe(duration)
```

### 2. 로깅 전략

```python
# app/utils/logger.py
from loguru import logger
import sys

# 로거 설정
logger.remove()  # 기본 핸들러 제거

# 콘솔 출력
logger.add(
    sys.stdout,
    format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level: <8}</level> | <cyan>{name}</cyan>:<cyan>{function}</cyan> - <level>{message}</level>",
    level="INFO"
)

# 파일 출력 (로테이션)
logger.add(
    "logs/app_{time:YYYY-MM-DD}.log",
    rotation="00:00",  # 매일 자정
    retention="30 days",  # 30일 보관
    compression="zip",  # 압축
    level="DEBUG"
)

# 에러 파일
logger.add(
    "logs/errors_{time:YYYY-MM-DD}.log",
    rotation="00:00",
    retention="90 days",
    level="ERROR"
)
```

---

## FAQ

### Q1: FastAPI와 Flask 중 어떤 것을 선택해야 하나요?

**FastAPI 선택 기준**:
- 새로운 프로젝트
- 비동기 처리 필요
- 자동 문서화 원함
- 타입 안정성 중요

**Flask 선택 기준**:
- 기존 Flask 프로젝트 확장
- 간단한 CRUD API
- 성숙한 생태계 필요
- 레거시 라이브러리 사용

### Q2: Python의 GIL 문제를 어떻게 해결하나요?

**해결 방법**:

1. **비동기 I/O** (가장 효과적)
```python
# I/O 대기 중 다른 작업 처리
async def handle_multiple_requests():
    tasks = [call_openai() for _ in range(100)]
    results = await asyncio.gather(*tasks)
```

2. **멀티프로세스**
```bash
# Uvicorn 워커 실행
uvicorn app.main:app --workers 4
```

3. **컨테이너 스케일링**
```yaml
# docker-compose.yml
api:
  deploy:
    replicas: 4  # 4개 인스턴스
```

### Q3: 대용량 트래픽 처리 방법은?

**수평 확장 전략**:

```
[Load Balancer - Nginx]
        ↓
   ┌────┴────┬────┬────┐
   ↓         ↓    ↓    ↓
[API-1] [API-2] [API-3] [API-4]
   ↓         ↓    ↓    ↓
   └────┬────┴────┴────┘
        ↓
  [PostgreSQL]
  [Redis]
  [Vector DB]
```

### Q4: OpenAI API 비용 최적화 방법은?

**비용 절감 전략**:

1. **캐싱**
```python
# 동일한 질문은 캐시에서 반환
cache_key = hash(message)
if cached := await redis_client.get(cache_key):
    return cached
```

2. **토큰 제한**
```python
# max_tokens 설정
max_tokens=500  # 긴 응답 방지
```

3. **저렴한 모델 사용**
```python
# 간단한 작업은 GPT-3.5
if is_simple_task:
    model = "gpt-3.5-turbo"  # $0.5/1M tokens
else:
    model = "gpt-4-turbo"  # $10/1M tokens
```

---

## 결론

### Python/FastAPI를 선택해야 하는 경우

✅ **스타트업/MVP**
- 빠른 시장 진입이 최우선
- 유연한 요구사항 변경
- 작은 팀 규모 (1~10명)

✅ **AI 중심 프로젝트**
- 최신 AI 기술 즉시 적용
- 데이터 사이언티스트와 협업
- 실험적 기능 많음

✅ **중소규모 서비스**
- 사용자: ~100만
- 동시 접속: ~1만
- 빠른 개발 사이클

### Java/Spring을 선택해야 하는 경우

✅ **엔터프라이즈 환경**
- 대규모 조직
- 기존 Java 시스템 통합
- 엄격한 거버넌스

✅ **대규모 트래픽**
- 사용자: 1000만+
- 복잡한 트랜잭션
- 장기 운영 계획

### 하이브리드 접근

**최선의 선택**은 두 가지를 결합:

```
Python FastAPI: AI 처리, 프로토타입
Java Spring: 핵심 비즈니스 로직

→ 각 언어의 강점 활용!
```

### 최종 조언

1. **작게 시작하기**: Python으로 빠르게 검증
2. **필요시 확장**: 트래픽 증가 시 Java 고려
3. **도구는 수단**: 비즈니스 가치에 집중
4. **팀 역량 고려**: 팀이 익숙한 기술 선택

---

## 참고 자료

- [FastAPI 공식 문서](https://fastapi.tiangolo.com/)
- [LangChain Python 문서](https://python.langchain.com/)
- [SQLAlchemy 문서](https://docs.sqlalchemy.org/)
- [OpenAI Python SDK](https://github.com/openai/openai-python)
- [ChromaDB 문서](https://docs.trychroma.com/)
- [Pydantic 문서](https://docs.pydantic.dev/)


================================================================================
## Python + FastAPI AI 서비스 작동 방식 심화
================================================================================

### AI 서비스 전체 흐름도

```
사용자 요청 (HTTP POST)
    ↓
FastAPI 엔드포인트 (@app.post)
    ↓
Pydantic 자동 검증 (타입, 범위, 필수 필드)
    ↓
비즈니스 로직 (async 함수)
    ├─ 세션 메모리 로드 (Redis/DB)
    ├─ 벡터 검색 (RAG - Pinecone/ChromaDB/Weaviate)
    └─ LLM API 호출 (OpenAI/Anthropic)
         ↓
    AI 응답 생성 (await)
         ↓
    응답 후처리 & 저장
         ↓
    JSON 응답 반환
```

### 핵심 특징: 비동기 I/O 기반 아키텍처

Python + FastAPI의 가장 큰 강점은 **ASGI(Asynchronous Server Gateway Interface)**를 통한 비동기 처리입니다.

#### 동기 vs 비동기 비교

**동기 방식 (Flask/Django WSGI)**:
```python
# Flask - 동기 처리
@app.route("/chat")
def chat(message: str):
    # OpenAI API 호출 (2초 대기) → 다른 요청 모두 블로킹!
    response = openai_client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": message}]
    )
    return response

# 요청 처리 시간
# 요청 1: 2초
# 요청 2: 2초 (요청 1 완료 후 시작)
# 요청 3: 2초 (요청 2 완료 후 시작)
# 총: 6초 (순차 처리)
```

**비동기 방식 (FastAPI ASGI)**:
```python
# FastAPI - 비동기 처리
@app.post("/chat")
async def chat(request: ChatRequest):
    # OpenAI API 호출 (2초 대기) → 대기 중 다른 요청 처리 가능!
    response = await openai_client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": request.message}]
    )
    return response

# 요청 처리 시간
# 요청 1, 2, 3: 동시에 API 대기 (비동기)
# 총: 약 2초 (병렬 처리)
```

#### 실전 예시: RAG 기반 AI 챗봇

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel
from typing import List
import asyncio

app = FastAPI()

# 1. Pydantic 모델 정의 (자동 검증)
class ChatRequest(BaseModel):
    message: str
    user_id: str
    session_id: str | None = None
    temperature: float = 0.7

class ChatResponse(BaseModel):
    response: str
    sources: List[str]
    session_id: str

# 2. 의존성 주입 (Dependency Injection)
async def get_vector_store():
    """벡터 DB 연결 제공"""
    return vector_db_client

async def get_chat_history(session_id: str):
    """세션 히스토리 로드"""
    return await redis_client.get(f"history:{session_id}")

# 3. RAG 기반 채팅 엔드포인트
@app.post("/chat", response_model=ChatResponse)
async def chat(
    request: ChatRequest,
    vector_store = Depends(get_vector_store),
    history = Depends(get_chat_history)
):
    """
    RAG 기반 AI 채팅

    작동 순서:
    1. 벡터 검색으로 관련 문서 찾기 (병렬)
    2. 대화 히스토리 로드 (병렬)
    3. 프롬프트 구성
    4. LLM API 호출
    5. 응답 반환 및 저장 (비동기)
    """

    # 1단계: 병렬 작업 (asyncio.gather)
    search_task = vector_store.similarity_search(
        request.message,
        k=5  # 상위 5개 문서
    )

    # 두 작업을 동시에 실행
    relevant_docs, chat_history = await asyncio.gather(
        search_task,
        get_chat_history(request.session_id)
    )

    # 2단계: 프롬프트 구성
    context = "\n".join([doc.content for doc in relevant_docs])
    system_prompt = f"""
    당신은 도움이 되는 AI 어시스턴트입니다.
    다음 참고 자료를 바탕으로 답변하세요:

    {context}
    """

    # 3단계: LLM 호출 (비동기)
    messages = [
        {"role": "system", "content": system_prompt},
        *chat_history,  # 이전 대화 포함
        {"role": "user", "content": request.message}
    ]

    response = await openai_client.chat.completions.create(
        model="gpt-4",
        messages=messages,
        temperature=request.temperature
    )

    # 4단계: 응답 저장 (백그라운드 태스크)
    asyncio.create_task(
        save_conversation(request, response)
    )

    return ChatResponse(
        response=response.choices[0].message.content,
        sources=[doc.metadata["source"] for doc in relevant_docs],
        session_id=request.session_id
    )

# 백그라운드 태스크 (응답 후 실행)
async def save_conversation(request, response):
    """대화 저장 및 벡터 임베딩 생성"""
    # DB 저장
    await db.conversations.insert_one({
        "user_id": request.user_id,
        "session_id": request.session_id,
        "message": request.message,
        "response": response.choices[0].message.content,
        "timestamp": datetime.now()
    })

    # 벡터 임베딩 생성 및 저장
    embedding = await embedding_model.embed(request.message)
    await vector_store.add(embedding, metadata={"session": request.session_id})
```

### Python AI 서비스의 핵심 패턴

#### 1. 라이브러리 즉시 활용 (Import & Use)

Python은 AI 라이브러리를 **설치하면 바로 사용** 가능합니다:

```bash
# 라이브러리 설치 (10초)
pip install langchain openai chromadb tiktoken

# 코드에서 즉시 사용 (import)
```

```python
from langchain.chat_models import ChatOpenAI
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.chains import RetrievalQA
import tiktoken

# 즉시 사용 가능 - 별도 설정 불필요
llm = ChatOpenAI(model="gpt-4")
embeddings = OpenAIEmbeddings()
vectorstore = Chroma(embedding_function=embeddings)

# 토큰 카운팅
encoder = tiktoken.encoding_for_model("gpt-4")
tokens = encoder.encode("Hello, world!")
print(f"Token count: {len(tokens)}")
```

**특징**:
- ✅ 간결한 코드 (보일러플레이트 최소)
- ✅ 빠른 프로토타이핑
- ✅ 인터프리터 언어 (컴파일 불필요)
- ⚠️ 런타임 타입 체크 (실행 시 오류 발견)

#### 2. LangChain 통합 (선택적)

LangChain은 Python에서 **가장 먼저 출시**되고 가장 **기능이 풍부**합니다:

```python
from langchain.chains import ConversationalRetrievalChain
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI
from langchain_community.vectorstores import Pinecone

# 메모리 + RAG + 대화 체인을 5줄로 구성
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

llm = ChatOpenAI(model="gpt-4")

qa_chain = ConversationalRetrievalChain.from_llm(
    llm=llm,
    retriever=vectorstore.as_retriever(),
    memory=memory
)

# 사용
response = qa_chain({"question": "오늘 날씨 어때?"})
```

#### 3. 스트리밍 응답 (SSE)

FastAPI는 **Server-Sent Events**를 통해 실시간 스트리밍을 쉽게 구현:

```python
from fastapi.responses import StreamingResponse

@app.post("/chat/stream")
async def chat_stream(request: ChatRequest):
    async def generate():
        async for chunk in openai_client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": request.message}],
            stream=True  # 스트리밍 활성화
        ):
            if chunk.choices[0].delta.content:
                yield f"data: {chunk.choices[0].delta.content}\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

### 패키징 및 배포

#### 라이브러리 관리

```
requirements.txt (의존성 선언)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
fastapi==0.109.0
uvicorn[standard]==0.27.0
openai==1.12.0
langchain==0.1.6
chromadb==0.4.22
tiktoken==0.5.2

↓ 설치 (pip install -r requirements.txt)

venv/lib/python3.11/site-packages/
├── fastapi/
├── openai/
├── langchain/
└── ... (모든 의존성)
```

#### Docker 컨테이너화

```dockerfile
# Python 베이스 이미지
FROM python:3.11-slim

WORKDIR /app

# 의존성 설치 (레이어 캐싱)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 애플리케이션 코드 복사
COPY . .

# FastAPI 실행
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**최종 이미지 구조**:
```
Docker Image (~500MB)
├── Python 런타임 (150MB)
├── 시스템 라이브러리 (50MB)
├── Python 패키지들 (250MB)
│   ├── fastapi/
│   ├── openai/
│   ├── langchain/
│   └── 기타 의존성
└── 애플리케이션 코드 (10MB)
    └── main.py
```

### 프로덕션 배포 예시

```yaml
# Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-ai-service
spec:
  replicas: 3  # 3개 Pod 실행
  template:
    spec:
      containers:
      - name: api
        image: myregistry/fastapi-ai:latest
        ports:
        - containerPort: 8000
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: ai-secrets
              key: openai-key
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 30
```

### 성능 최적화 전략

#### 1. 비동기 작업 병렬화

```python
import asyncio

@app.post("/analyze")
async def analyze(text: str):
    # 3개 작업을 동시에 실행
    sentiment_task = analyze_sentiment(text)
    entities_task = extract_entities(text)
    summary_task = generate_summary(text)

    # 모두 완료될 때까지 대기 (병렬 실행)
    sentiment, entities, summary = await asyncio.gather(
        sentiment_task,
        entities_task,
        summary_task
    )

    return {
        "sentiment": sentiment,
        "entities": entities,
        "summary": summary
    }
```

#### 2. 캐싱 전략

```python
from functools import lru_cache
import aioredis

# 메모리 캐싱
@lru_cache(maxsize=1000)
def get_embedding(text: str):
    """동일한 텍스트는 캐시에서 반환"""
    return embedding_model.embed(text)

# Redis 캐싱
redis = aioredis.from_url("redis://localhost")

async def get_chat_response(message: str):
    # 캐시 확인
    cached = await redis.get(f"response:{message}")
    if cached:
        return cached

    # LLM 호출
    response = await llm.generate(message)

    # 캐시 저장 (1시간)
    await redis.setex(f"response:{message}", 3600, response)
    return response
```

#### 3. 워커 프로세스 스케일링

```bash
# Uvicorn 워커 프로세스 증가
uvicorn main:app --workers 4 --host 0.0.0.0 --port 8000

# Gunicorn + Uvicorn (프로덕션)
gunicorn main:app \
  --workers 4 \
  --worker-class uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --timeout 120
```

### 장점 요약

#### ✅ 장점

1. **빠른 개발 속도**
   - 간결한 문법
   - 풍부한 AI 라이브러리
   - 빠른 프로토타이핑

2. **AI 생태계 우선 지원**
   - LangChain, Transformers 등 최신 기술 즉시 사용
   - 커뮤니티 활발

3. **비동기 성능**
   - I/O 중심 작업에 뛰어난 성능
   - 낮은 메모리 사용

4. **낮은 진입 장벽**
   - 초보자도 쉽게 시작
   - 데이터 사이언티스트와 협업 용이

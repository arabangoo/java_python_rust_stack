# Java Spring AI를 활용한 엔터프라이즈 AI 서비스 구축 가이드

## 목차
1. [개요](#개요)
2. [AI 서비스 개발 기초 이해하기](#ai-서비스-개발-기초-이해하기)
3. [Python/FastAPI vs Java/Spring AI 비교](#pythonfastapi-vs-javaspring-ai-비교)
4. [Spring AI 아키텍처 완전 이해](#spring-ai-아키텍처-완전-이해)
5. [프로젝트 설정 단계별 가이드](#프로젝트-설정-단계별-가이드)
6. [핵심 구현 상세 설명](#핵심-구현-상세-설명)
7. [장기 메모리 관리 완전 가이드](#장기-메모리-관리-완전-가이드)
8. [성능 최적화 전략](#성능-최적화-전략)
9. [프로덕션 배포 실전 가이드](#프로덕션-배포-실전-가이드)
10. [모니터링 및 운영](#모니터링-및-운영)
11. [FAQ](#faq)

---

## 개요

본 가이드는 **Java와 Spring AI**를 활용하여 엔터프라이즈급 AI 서비스를 구축하는 방법을 다룹니다. Python/FastAPI 기반 솔루션과의 비교를 통해 각 스택의 장단점을 이해하고, 특히 대규모 엔터프라이즈 환경에서 Java/Spring AI가 제공하는 이점을 확인할 수 있습니다.

### 왜 Java/Spring AI인가?

- **엔터프라이즈 생태계 통합**: 기존 Java 기반 레거시 시스템과의 원활한 통합
- **타입 안정성**: 컴파일 타임 타입 체크로 런타임 오류 최소화
- **성능 및 확장성**: JVM의 성숙한 최적화와 멀티스레딩 지원
- **장기 메모리 관리**: JPA/Hibernate를 통한 효율적인 데이터 영속성
- **보안**: Spring Security를 통한 엔터프라이즈급 보안 구현

---

## AI 서비스 개발 기초 이해하기

### AI 챗봇 서비스란 무엇인가?

AI 챗봇 서비스는 마치 **카페의 바리스타와 대화하는 것**과 비슷합니다:

```
고객(사용자) → 주문(질문) → 바리스타(AI) → 커피(답변) → 고객(사용자)
```

이 과정에서 바리스타(AI)는:
- 고객의 주문을 기억해야 함 (메모리)
- 이전 주문 이력을 참고할 수 있음 (대화 히스토리)
- 고객 취향을 학습함 (개인화)
- 빠르게 서비스를 제공해야 함 (성능)

### 엔터프라이즈 AI 서비스가 필요로 하는 핵심 기능

#### 1. **대화 관리 (Conversation Management)**
**일반 설명**: 사용자와 AI의 대화 내용을 저장하고 관리하는 기능

**왜 필요한가?**
- 사용자가 "그것"이라고 말했을 때 이전 대화 맥락을 이해해야 함
- 여러 세션에 걸친 대화를 이어갈 수 있어야 함

**실생활 예시**:
```
사용자: "날씨 알려줘"
AI: "오늘 서울은 맑고 기온은 15도입니다"

[5분 후]
사용자: "그럼 우산 필요해?" ← "그것"이 날씨를 의미함을 이해해야 함
AI: "아니요, 오늘은 비가 오지 않아서 우산이 필요없습니다"
```

#### 2. **벡터 메모리 (Vector Memory)**
**일반 설명**: 대화 내용을 숫자 배열(벡터)로 변환하여 저장하고, 의미적으로 유사한 대화를 찾는 기능

**왜 필요한가?**
- 1년 전 대화 내용도 관련성 있으면 찾아낼 수 있음
- 키워드 검색이 아닌 의미 기반 검색

**비유**:
- 일반적인 검색(키워드): "사과" 검색 → "사과"라는 단어가 포함된 문서만 찾음
- 벡터 검색(의미): "사과" 검색 → "애플", "과일", "빨간 과일" 등 의미가 유사한 것도 찾음

**기술적 원리**:
```
텍스트: "오늘 날씨가 좋네요"
↓ (임베딩 변환)
벡터: [0.2, -0.5, 0.8, 0.1, ..., 0.3] (1536차원 배열)
```

이 벡터를 데이터베이스에 저장하고, 새로운 질문이 들어오면:
```
질문: "오늘 기후 어때요?"
↓ (임베딩 변환)
벡터: [0.3, -0.4, 0.7, 0.2, ..., 0.4]
↓ (유사도 계산)
"오늘 날씨가 좋네요"와 의미가 비슷함! (유사도: 0.92)
```

#### 3. **메모리 계층 구조 (Memory Hierarchy)**
**일반 설명**: 메모리를 단기/장기로 나누어 효율적으로 관리

**실생활 비유**: 사람의 기억과 같음
- **단기 기억**: 방금 전 대화 (Working Memory) - 빠르지만 용량 제한
- **장기 기억**: 오래된 대화 요약 (Long-term Memory) - 느리지만 대용량
- **중요한 기억**: 사용자 선호도, 중요 사실 (Vector Memory) - 언제든 빠르게 검색

**시스템 구조**:
```
[사용자 질문]
      ↓
[1단계: 단기 메모리 로드 (빠름)]
  - 최근 10개 대화
  - Redis 캐시에서 조회 (5ms)
      ↓
[2단계: 장기 메모리 요약 로드]
  - 과거 대화 요약본
  - 데이터베이스에서 조회 (25ms)
      ↓
[3단계: 벡터 메모리 검색 (선택적)]
  - 질문과 관련된 과거 대화
  - Vector DB에서 의미 검색 (50ms)
      ↓
[4단계: 컨텍스트 통합 및 AI 호출]
```

#### 4. **트랜잭션 관리 (Transaction Management)**
**일반 설명**: 여러 작업이 하나의 단위로 성공하거나 실패하도록 보장

**왜 중요한가?**
은행 거래와 같은 원리:
```
[성공 케이스]
1. 대화 저장 → 성공
2. 벡터 임베딩 생성 → 성공
3. 캐시 업데이트 → 성공
→ 모두 저장됨 ✅

[실패 케이스]
1. 대화 저장 → 성공
2. 벡터 임베딩 생성 → 실패! 💥
3. 캐시 업데이트 → 실행 안됨
→ 1번도 롤백되어 아무것도 저장 안됨 ✅
```

**Java/Spring의 강점**:
```java
@Transactional  // 이 어노테이션 하나로 트랜잭션 자동 관리!
public void saveConversation(String message, String response) {
    // 1. DB에 저장
    conversationRepository.save(conversation);

    // 2. 벡터 저장
    vectorStore.add(embedding);

    // 3. 캐시 업데이트
    cache.put(key, value);

    // 하나라도 실패하면 자동으로 모두 롤백됨!
}
```

#### 5. **확장성 (Scalability)**
**일반 설명**: 사용자가 늘어나도 서비스가 잘 작동하도록 시스템을 늘리는 능력

**레스토랑 비유**:
```
손님 10명: 요리사 1명으로 충분
손님 100명: 요리사 10명 필요
손님 1000명: 요리사 100명 + 주방 확장 필요
```

**시스템 확장 전략**:
```
[수직 확장 - Scale Up]
서버 1대의 성능을 올림
CPU: 4코어 → 16코어
메모리: 8GB → 32GB
👍 장점: 간단함
👎 단점: 한계가 있음, 비쌈

[수평 확장 - Scale Out]
서버 대수를 늘림
서버 1대 → 서버 10대
👍 장점: 무한 확장 가능
👎 단점: 복잡함, 데이터 동기화 필요
```

### Spring AI vs 다른 프레임워크의 핵심 차이

#### 타입 안정성이란?

**Python (동적 타입)**:
```python
def process_message(message):  # message가 뭐든 받음
    return message.upper()      # 실행 시점에 에러 발생 가능

# 실행하기 전까지 오류를 모름
process_message(123)  # 💥 런타임 에러: int에는 upper()가 없음
```

**Java (정적 타입)**:
```java
public String processMessage(String message) {  // String만 받음
    return message.toUpperCase();
}

// 컴파일 시점에 오류 감지
processMessage(123);  // 💥 컴파일 에러: int는 String이 아님
// → 코드 실행 전에 IDE에서 빨간 줄로 표시됨!
```

**왜 엔터프라이즈에서 중요한가?**
- 대규모 팀 개발: 다른 사람이 작성한 코드 사용 시 실수 방지
- 리팩토링: 코드 수정 시 어디서 문제가 생길지 미리 알 수 있음
- 유지보수: 몇 년 후에도 안전하게 코드 수정 가능

#### JVM의 성숙한 생태계

**JVM이란?**
Java Virtual Machine - Java 코드를 실행하는 가상 머신

**20년 이상 발전한 기술**:
```
[메모리 관리 (GC - Garbage Collection)]
자동으로 사용하지 않는 메모리를 정리
→ 메모리 누수 방지
→ 개발자는 비즈니스 로직에 집중

[JIT 컴파일러 (Just-In-Time)]
실행 중에 자주 사용되는 코드를 최적화
→ 실행 시간이 지날수록 더 빨라짐

[멀티스레딩]
여러 작업을 동시에 처리
→ CPU 코어를 최대한 활용
```

**실제 성능 차이 예시**:
```
동일한 작업 (1000명이 동시에 질문):

Python (uvicorn):
- 단일 스레드 기반
- 비동기로 처리 필요 (async/await)
- 처리 속도: 3,200 req/s

Java (Spring):
- 멀티스레드 기반
- 자연스러운 동시 처리
- 처리 속도: 4,800 req/s
```

### 언제 어떤 기술을 선택해야 할까?

#### 프로젝트 단계별 권장 스택

**1단계: 아이디어 검증 (MVP)**
```
추천: Python/FastAPI
이유:
- 빠른 개발 (1-2주)
- 적은 코드량
- AI 라이브러리 풍부
예산: 낮음 ($10K-$50K)
팀 크기: 1-3명
```

**2단계: 베타 서비스**
```
추천: Python/FastAPI 또는 Java/Spring AI
이유:
- 사용자 피드백 수집
- 성능 요구사항 파악
- 아키텍처 설계 개선
예산: 중간 ($50K-$200K)
팀 크기: 3-10명
```

**3단계: 프로덕션 서비스**
```
추천: Java/Spring AI
이유:
- 안정성과 확장성 필요
- 장기 유지보수 계획
- 엔터프라이즈 통합
예산: 높음 ($200K+)
팀 크기: 10명+
```

#### 의사결정 플로우차트

```
[시작]
  ↓
팀에 Java 개발자가 있나요?
  ↓ YES                    ↓ NO
기존 Java 시스템과      빠른 프로토타입이
통합이 필요한가요?       최우선인가요?
  ↓ YES    ↓ NO            ↓ YES    ↓ NO
Java/      사용자가       Python/   일단 Python으로
Spring AI  10만명+?       FastAPI   시작 후 필요시
          ↓ YES  ↓ NO              전환 고려
          Java   Python
          추천   가능
```

### 다음 섹션 미리보기

이제 기본 개념을 이해했으니, 다음 섹션에서는:
1. Python과 Java의 **구체적인 코드 비교**
2. Spring AI의 **상세한 아키텍처 구조**
3. **실전 프로젝트 구축 방법**

을 단계별로 학습합니다.

---

## Python/FastAPI vs Java/Spring AI 비교

### 아키텍처 비교표

| 측면 | Python/FastAPI | Java/Spring AI |
|------|----------------|----------------|
| **개발 속도** | ⭐⭐⭐⭐⭐<br/>빠른 프로토타이핑, 간결한 코드 | ⭐⭐⭐<br/>초기 설정 복잡, 보일러플레이트 많음 |
| **타입 안정성** | ⭐⭐⭐<br/>Pydantic으로 런타임 검증 | ⭐⭐⭐⭐⭐<br/>컴파일 타임 타입 체크 |
| **성능** | ⭐⭐⭐<br/>단일 스레드 기본, async 필요 | ⭐⭐⭐⭐<br/>멀티스레딩, JVM 최적화 |
| **메모리 관리** | ⭐⭐⭐<br/>GC 제어 제한적 | ⭐⭐⭐⭐<br/>정교한 GC 튜닝 가능 |
| **엔터프라이즈 통합** | ⭐⭐<br/>별도 어댑터 필요 | ⭐⭐⭐⭐⭐<br/>네이티브 통합 |
| **AI 라이브러리** | ⭐⭐⭐⭐⭐<br/>풍부한 생태계 (Transformers, LangChain) | ⭐⭐⭐⭐<br/>Spring AI, DJL (성장 중) |
| **배포 복잡도** | ⭐⭐⭐⭐<br/>간단한 컨테이너화 | ⭐⭐⭐<br/>빌드 프로세스 복잡 |
| **장기 메모리 관리** | ⭐⭐⭐<br/>SQLAlchemy, Redis 통합 | ⭐⭐⭐⭐⭐<br/>JPA/Hibernate, 트랜잭션 관리 탁월 |

### 코드 비교: 간단한 챗봇 엔드포인트

#### Python/FastAPI

```python
from fastapi import FastAPI
from langchain.chat_models import ChatOpenAI
from langchain.memory import ConversationBufferMemory

app = FastAPI()
llm = ChatOpenAI(model="gpt-4")
memory = ConversationBufferMemory()

@app.post("/chat")
async def chat(message: str):
    response = llm.predict(message)
    memory.save_context({"input": message}, {"output": response})
    return {"response": response}
```

#### Java/Spring AI

```java
@RestController
@RequestMapping("/api")
public class ChatController {

    private final ChatClient chatClient;
    private final ConversationService conversationService;

    public ChatController(ChatClient.Builder builder,
                         ConversationService conversationService) {
        this.chatClient = builder.build();
        this.conversationService = conversationService;
    }

    @PostMapping("/chat")
    public ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest request) {
        String response = chatClient.prompt()
            .user(request.getMessage())
            .call()
            .content();

        conversationService.saveConversation(
            request.getMessage(),
            response,
            request.getUserId()
        );

        return ResponseEntity.ok(new ChatResponse(response));
    }
}
```

### 성능 벤치마크 (1000 동시 요청 기준)

```
Python/FastAPI (uvicorn workers=4):
- 평균 응답시간: 245ms
- Throughput: 3,200 req/s
- 메모리 사용: 512MB

Java/Spring AI (Tomcat):
- 평균 응답시간: 180ms
- Throughput: 4,800 req/s
- 메모리 사용: 768MB (초기), 안정화 후 512MB
```

---

## Spring AI 아키텍처 완전 이해

### 아키텍처 설계 철학

Spring AI는 **관심사의 분리(Separation of Concerns)** 원칙을 따릅니다. 이를 레스토랑에 비유하면:

```
레스토랑 구조              →    Spring AI 구조
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
웨이터 (주문 받기)         →    Controller (HTTP 요청 처리)
주방장 (요리 결정)         →    Service (비즈니스 로직)
보조 요리사 (재료 준비)    →    Spring AI Layer (AI 통신)
창고 관리자 (재료 보관)    →    Repository (데이터 저장/조회)
창고 (실제 저장소)         →    Database, Cache, Vector DB
```

### 레이어 구조 상세 설명

```
[Presentation Layer - Controllers]
  역할: HTTP 요청 받기 → 검증 → Service 호출 → 응답 반환
  예시: POST /api/chat 요청 처리
  기술: @RestController, @RequestMapping
                    ↓
[Service Layer - Business Logic]
  역할: 비즈니스 규칙 적용 (권한, 로깅, 트랜잭션)
  예시: "이 사용자가 채팅할 수 있는가?", "대화 저장"
  기술: @Service, @Transactional
                    ↓
[Spring AI Integration Layer]
  역할: AI 모델과의 실제 통신 담당
  컴포넌트:
    - ChatClient: OpenAI/Claude 등 LLM 호출
    - VectorStore: 임베딩 저장/검색
    - EmbeddingClient: 텍스트를 벡터로 변환
    - DocumentReader: 파일/문서 읽기
                    ↓
[Data Access Layer - Repositories]
  역할: 데이터베이스 CRUD 작업 추상화
  예시: "대화 히스토리 조회", "사용자 정보 저장"
  기술: JPA, Spring Data
                    ↓
[Infrastructure - DB, Cache, Queue]
  실제 저장소:
    - PostgreSQL: 구조화된 데이터 (대화, 사용자)
    - Redis: 빠른 캐시 (세션, 최근 대화)
    - Vector DB: 임베딩 벡터 (의미 검색)
```

### 데이터 흐름 상세 예시

사용자가 "오늘 날씨 어때?"라고 질문하면:

```
[1단계: Controller - 요청 받기]
POST /api/v1/chat
{
  "userId": "user123",
  "sessionId": "session456",
  "message": "오늘 날씨 어때?"
}
↓
ChatController.chat() 메서드 실행
- 요청 데이터 검증 (@Valid)
- userId 헤더 확인
- Service 레이어 호출

[2단계: Service - 비즈니스 로직]
ChatService.chat() 실행
↓
2-1. 권한 확인
    "user123은 채팅 가능한 사용자인가?" ✅
↓
2-2. 대화 히스토리 로드 (Repository)
    conversationRepository.findRecent(user123, session456)
    → 최근 10개 대화 가져오기
    [캐시 확인: Redis → 있으면 바로 반환, 없으면 DB 조회]
↓
2-3. 벡터 메모리 검색 (Spring AI Layer)
    vectorStore.similaritySearch("오늘 날씨 어때?")
    → 과거에 날씨 관련 대화가 있었는지 검색
    결과: ["지난주에 비가 왔었어요", "우산 챙기세요"]

[3단계: Spring AI Layer - AI 호출]
3-1. 프롬프트 구성
    System: "당신은 친절한 AI입니다. 과거 대화: [지난주에 비가 왔었어요]"
    User History: [최근 10개 대화]
    User: "오늘 날씨 어때?"
↓
3-2. ChatClient로 OpenAI/Claude 호출
    chatClient.prompt()
        .system(systemPrompt)
        .messages(history)
        .user("오늘 날씨 어때?")
        .call()
↓
3-3. AI 응답 받기
    "오늘은 맑고 기온은 15도입니다. 지난주와 달리 우산은 필요없어요!"

[4단계: Service - 후처리]
4-1. 대화 저장 (Repository)
    Conversation conversation = new Conversation();
    conversation.setUserMessage("오늘 날씨 어때?");
    conversation.setAssistantMessage("오늘은 맑고...");
    conversationRepository.save(conversation);
    [트랜잭션 보장: 실패하면 모두 롤백]
↓
4-2. 벡터 임베딩 생성 및 저장 (비동기)
    @Async updateVectorMemoryAsync()
    embeddingClient.embed("User: 오늘 날씨... Assistant: 오늘은...")
    → [0.2, -0.5, 0.8, ..., 0.3] (1536차원 벡터)
    vectorStore.add(embedding)
↓
4-3. 캐시 무효화
    cacheManager.evict("history:user123:session456")
    → 다음 요청 때 새로운 데이터 로드되도록

[5단계: Controller - 응답 반환]
{
  "message": "오늘은 맑고 기온은 15도입니다...",
  "sessionId": "session456",
  "metadata": {
    "model": "gpt-4",
    "tokens": 256
  }
}
```

### 핵심 컴포넌트 상세 설명

#### 1. ChatClient - AI 통신의 중심

**역할**: 다양한 LLM(OpenAI, Claude, Ollama 등)과의 통신을 표준화

**일반 설명**:
마치 **번역기**와 같습니다. 각 AI 모델은 다른 API 형식을 사용하는데, ChatClient가 이를 통일된 인터페이스로 제공합니다.

**왜 필요한가?**
```
ChatClient 없이:
OpenAI 사용: openAIClient.complete(prompt, model, temperature, ...)
Claude 사용: claudeClient.chat(messages, config, options, ...)
→ AI 모델 변경 시 모든 코드 수정 필요 😱

ChatClient 사용:
chatClient.prompt()
    .user("질문")
    .call()
→ 어떤 AI든 동일한 코드! 설정만 변경하면 됨 ✅
```

**실전 예시**:
```java
// 기본 사용
String response = chatClient.prompt()
    .user("Java와 Python의 차이는?")
    .call()
    .content();

// 고급 사용 - 대화 히스토리 포함
ChatResponse response = chatClient.prompt()
    .system("당신은 프로그래밍 전문가입니다")
    .messages(conversationHistory)  // 이전 대화
    .user("Spring Boot란?")
    .call()
    .chatResponse();

// 스트리밍 응답 (실시간 출력)
Flux<String> stream = chatClient.prompt()
    .user("긴 이야기 들려줘")
    .stream()
    .content();
// → "안" → "녕" → "하" → "세" → "요" (한 글자씩 도착)
```

#### 2. VectorStore - 의미 검색의 핵심

**역할**: 텍스트를 벡터로 변환하여 저장하고, 의미가 유사한 내용을 찾아줌

**일반 설명**:
마치 **도서관의 주제별 분류 시스템**과 같습니다:
- 일반 검색: 책 제목으로만 찾기
- 벡터 검색: 내용의 주제/의미로 찾기

**기술 원리**:
```
[1단계: 임베딩 생성]
텍스트: "강아지가 공원에서 뛰어놀아요"
↓ EmbeddingClient.embed()
벡터: [0.2, 0.5, -0.3, 0.8, ..., 0.1]  (1536개 숫자)

[2단계: 저장]
vectorStore.add(Document(content="강아지가...", embedding=[...]))

[3단계: 검색]
질문: "개가 놀고 있어요"
↓ 임베딩 변환
질문 벡터: [0.3, 0.4, -0.2, 0.7, ..., 0.2]
↓ 유사도 계산 (코사인 유사도)
"강아지가 공원에서..."와 유사도: 0.89 (89% 유사)
→ 검색 결과로 반환! ✅
```

**지원하는 Vector DB**:
```
PostgreSQL + pgvector:
  장점: 기존 DB 활용, 관리 단순
  단점: 대용량 벡터 처리 시 느림
  추천: 중소규모 서비스 (~100만 벡터)

Pinecone:
  장점: 관리형 서비스, 매우 빠름
  단점: 비용 발생
  추천: 대규모 서비스, 빠른 속도 필요

Weaviate:
  장점: 오픈소스, 기능 풍부
  단점: 별도 서버 관리 필요
  추천: 복잡한 검색 요구사항

Chroma:
  장점: 설치 간단, 경량
  단점: 프로덕션 기능 제한적
  추천: 개발/테스트 환경
```

#### 3. MessageChatMemoryAdvisor - 대화 컨텍스트 관리자

**역할**: 대화 히스토리를 자동으로 관리하고 AI에게 전달

**일반 설명**:
마치 **회의록 작성자**와 같습니다. 회의 내용을 기록하고, 다음 회의 때 이전 논의 사항을 상기시켜줍니다.

**작동 원리**:
```java
// Advisor 설정
MessageChatMemoryAdvisor advisor = new MessageChatMemoryAdvisor(
    chatMemory,      // 대화 저장소
    "conversationId", // 대화 식별자
    10               // 최근 10개 메시지 유지
);

chatClient.prompt()
    .user("질문")
    .advisors(advisor)  // Advisor 적용
    .call();

// Advisor가 자동으로:
// 1. 이전 대화 10개를 chatMemory에서 로드
// 2. AI 호출 시 함께 전송
// 3. 새로운 대화를 chatMemory에 저장
```

**메모리 전략**:
```
[1. 슬라이딩 윈도우]
최근 N개만 유지
장점: 간단, 빠름
단점: 오래된 정보 손실

[2. 요약 기반]
오래된 대화는 요약본만 유지
장점: 메모리 효율적
단점: 요약 과정 필요 (비용 발생)

[3. 토큰 제한 기반]
LLM 토큰 한도(4K, 8K 등)에 맞춰 조정
장점: 항상 최대한 활용
단점: 메시지 개수 가변적
```

#### 4. DocumentReader - 문서 처리 엔진

**역할**: 다양한 형식의 문서를 읽어서 AI가 이해할 수 있는 형태로 변환

**지원 형식**:
```
PDF: PdfDocumentReader
Word: TikaDocumentReader
JSON: JsonReader
CSV: CsvReader
HTML/Web: WebDocumentReader
```

**실전 예시**:
```java
// PDF 읽기
DocumentReader pdfReader = new PdfDocumentReader(
    new FileSystemResource("manual.pdf")
);
List<Document> documents = pdfReader.read();

// 청크 분할 (긴 문서를 작은 조각으로)
TextSplitter splitter = new TokenTextSplitter(
    500,  // 최대 500 토큰
    100   // 100 토큰 오버랩 (문맥 유지)
);
List<Document> chunks = splitter.split(documents);

// 벡터 DB에 저장
vectorStore.add(chunks);

// 이제 PDF 내용을 검색할 수 있음!
List<Document> results = vectorStore.similaritySearch(
    "제품 보증 기간은?"
);
```

#### 5. OutputParser - 구조화된 응답 파싱

**역할**: AI의 자유로운 텍스트 응답을 구조화된 데이터로 변환

**왜 필요한가?**
```
AI 응답 (텍스트):
"이 제품의 가격은 50,000원이고 재고는 25개입니다."

프로그램이 원하는 형태 (구조화):
{
  "price": 50000,
  "stock": 25
}
```

**실전 예시**:
```java
// 1. 응답 형식 정의
record ProductInfo(
    @JsonProperty("price") int price,
    @JsonProperty("stock") int stock,
    @JsonProperty("available") boolean available
) {}

// 2. OutputParser 생성
BeanOutputParser<ProductInfo> parser =
    new BeanOutputParser<>(ProductInfo.class);

// 3. AI에게 형식 지정
String response = chatClient.prompt()
    .user("이 제품 정보 알려줘: " + productName)
    .system("응답 형식: " + parser.getFormat())
    .call()
    .content();

// 4. 파싱
ProductInfo info = parser.parse(response);
System.out.println(info.price());   // 50000
System.out.println(info.stock());   // 25
```

### 아키텍처 설계 Best Practices

#### 1. 계층 간 의존성 규칙

```
[올바른 의존성 방향]
Controller → Service → Repository
         ↘ Spring AI ↗

[잘못된 의존성]
Controller ← Service (❌)
Controller → Repository 직접 접근 (❌)
Repository → Service (❌)
```

#### 2. 트랜잭션 경계 설정

```java
// ✅ 올바른 예: Service 레이어에서 트랜잭션
@Service
@Transactional
public class ChatService {
    public void saveChat(...) {
        // 여러 DB 작업을 하나의 트랜잭션으로
    }
}

// ❌ 잘못된 예: Controller에서 트랜잭션
@RestController
@Transactional  // 너무 큰 범위!
public class ChatController {
    // HTTP 요청 전체가 트랜잭션에 포함됨
}
```

#### 3. 비동기 처리 분리

```
[동기 처리] - 사용자 대기
사용자 질문 → AI 응답 → 응답 반환
(3초)

[비동기 처리] - 사용자는 바로 응답 받음
사용자 질문 → AI 응답 → 응답 반환
                    ↓
              [백그라운드 작업]
              벡터 저장 (비동기)
              로그 기록 (비동기)
              알림 발송 (비동기)
```

### 다음 섹션에서 배울 내용

이제 아키텍처를 이해했으니:
1. **실제 프로젝트 설정** - Maven/Gradle 설정부터 시작
2. **코드 구현** - 각 레이어별 실제 코드
3. **테스트 전략** - 단위/통합 테스트

---

## 프로젝트 설정 단계별 가이드

### 프로젝트 설정 시작하기 전에

#### 필요한 도구 설치

```
[필수 도구]
1. JDK 21+
   - 다운로드: https://adoptium.net/
   - 설치 확인: java -version

2. Maven 3.8+ 또는 Gradle 8+
   - Maven: https://maven.apache.org/download.cgi
   - 확인: mvn -version

3. IDE (택1)
   - IntelliJ IDEA (추천)
   - Eclipse
   - VS Code + Java Extension Pack

[선택 도구]
4. Docker Desktop
   - PostgreSQL, Redis 실행용
   - https://www.docker.com/products/docker-desktop

5. Postman 또는 cURL
   - API 테스트용
```

#### 프로젝트 생성 방법

**방법 1: Spring Initializr 웹사이트 (초보자 추천)**

```
1. https://start.spring.io/ 접속

2. 프로젝트 메타데이터 입력:
   - Project: Maven
   - Language: Java
   - Spring Boot: 3.2.0+
   - Group: com.cloudai
   - Artifact: springai-service
   - Java: 21

3. Dependencies 추가 (Add Dependencies 버튼):
   - Spring Web
   - Spring Data JPA
   - PostgreSQL Driver
   - Spring Boot Actuator
   - Validation
   - Lombok

4. GENERATE 버튼 클릭 → ZIP 다운로드 → 압축 해제
```

**방법 2: IntelliJ IDEA에서 직접 생성**

```
1. File → New → Project
2. Spring Initializr 선택
3. 위와 동일한 설정 입력
4. Create
```

### 의존성 이해하기

Maven의 `pom.xml`은 프로젝트의 **레시피**와 같습니다. 어떤 재료(라이브러리)가 필요한지 명시합니다.

#### pom.xml 구조 설명

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <!--
    ============================================
    1. 프로젝트 기본 정보
    ============================================
    -->
    <modelVersion>4.0.0</modelVersion>

    <!--
    Spring Boot 부모 POM 상속
    → Spring Boot의 기본 설정과 의존성 버전 자동 관리
    -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <!--
    우리 프로젝트 식별 정보
    groupId: 회사/조직 도메인 (역순)
    artifactId: 프로젝트 이름
    version: 프로젝트 버전
    -->
    <groupId>com.cloudai</groupId>
    <artifactId>springai-service</artifactId>
    <version>1.0.0</version>

    <!--
    ============================================
    2. 프로젝트 설정
    ============================================
    -->
    <properties>
        <!-- Java 버전: 21 사용 (최신 기능 활용) -->
        <java.version>21</java.version>

        <!-- Spring AI 버전 고정 (안정성) -->
        <spring-ai.version>1.0.0-M3</spring-ai.version>
    </properties>

    <!--
    ============================================
    3. 의존성 (Dependencies)
    ============================================
    실제로 사용할 라이브러리들
    -->
    <dependencies>
        <!--
        ===================================
        Spring AI 핵심 라이브러리
        ===================================
        -->

        <!--
        Spring AI + OpenAI 통합
        → ChatClient, EmbeddingClient 제공
        → OpenAI API 자동 설정
        -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
            <!-- 버전은 spring-ai.version 변수에서 가져옴 -->
        </dependency>

        <!--
        Vector Store (PostgreSQL + pgvector)
        → 임베딩 벡터 저장 및 검색
        → PostgreSQL의 pgvector 확장 사용

        왜 pgvector?
        - 별도 Vector DB 불필요 (비용 절감)
        - 기존 PostgreSQL 활용
        - 트랜잭션 보장
        -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
        </dependency>

        <!--
        ===================================
        Spring Boot 기본 기능
        ===================================
        -->

        <!--
        Spring Web
        → @RestController, @RequestMapping
        → HTTP API 제공 기능
        → 내장 Tomcat 서버
        -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--
        Spring Data JPA
        → @Entity, @Repository
        → 데이터베이스 CRUD 자동화
        → Hibernate ORM 포함

        왜 JPA?
        - SQL 작성 최소화
        - 객체 지향적 DB 접근
        - 트랜잭션 관리 자동화
        -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!--
        Spring Cache
        → @Cacheable, @CacheEvict
        → Redis 또는 메모리 캐시

        왜 필요?
        - 대화 히스토리 빠른 조회
        - DB 부하 감소
        - 응답 시간 단축 (50ms → 5ms)
        -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>

        <!--
        Validation
        → @Valid, @NotNull, @Size
        → 입력 데이터 자동 검증

        예시:
        @NotNull String message;
        → message가 null이면 자동으로 400 Bad Request
        -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!--
        ===================================
        데이터베이스 및 캐시
        ===================================
        -->

        <!--
        PostgreSQL 드라이버
        → Java와 PostgreSQL 연결
        → pgvector 확장 지원
        -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <!-- 버전은 부모 POM에서 자동 관리 -->
        </dependency>

        <!--
        Redis (분산 캐시)
        → 여러 서버에서 캐시 공유
        → 세션 저장소로도 사용 가능

        언제 사용?
        - 서버가 2대 이상일 때
        - 빠른 데이터 조회 필요 시
        -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!--
        ===================================
        개발 편의 도구
        ===================================
        -->

        <!--
        Lombok
        → @Getter, @Setter, @NoArgsConstructor
        → 보일러플레이트 코드 자동 생성

        예시:
        @Getter @Setter
        public class User {
            private String name;  // getter/setter 자동 생성
        }
        -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>  <!-- 런타임에 불필요 -->
        </dependency>

        <!--
        ===================================
        모니터링 및 운영
        ===================================
        -->

        <!--
        Spring Boot Actuator
        → /actuator/health 헬스체크
        → /actuator/metrics 메트릭 수집
        → /actuator/info 애플리케이션 정보

        프로덕션 필수!
        - 서버 상태 모니터링
        - 성능 메트릭 수집
        - 로드밸런서 헬스체크
        -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!--
        Micrometer Prometheus
        → 메트릭을 Prometheus 형식으로 노출
        → Grafana로 시각화 가능

        모니터링 스택:
        Actuator → Micrometer → Prometheus → Grafana
        -->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
    </dependencies>

    <!--
    ============================================
    4. 의존성 버전 관리 (BOM)
    ============================================
    Bill of Materials - 의존성 버전 일괄 관리
    -->
    <dependencyManagement>
        <dependencies>
            <!--
            Spring AI BOM
            → Spring AI 관련 모든 라이브러리 버전 통일
            → 버전 충돌 방지
            -->
            <dependency>
                <groupId>org.springframework.ai</groupId>
                <artifactId>spring-ai-bom</artifactId>
                <version>${spring-ai.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!--
    ============================================
    5. 빌드 설정
    ============================================
    -->
    <build>
        <plugins>
            <!--
            Spring Boot Maven Plugin
            → 실행 가능한 JAR 생성
            → mvn spring-boot:run 명령어 제공

            생성되는 JAR:
            - 모든 의존성 포함 (Fat JAR)
            - java -jar app.jar로 실행 가능
            -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- Lombok 제외 (런타임 불필요) -->
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 1. 전체 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <groupId>com.cloudai</groupId>
    <artifactId>springai-service</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>21</java.version>
        <spring-ai.version>1.0.0-M3</spring-ai.version>
    </properties>

    <dependencies>
        <!-- Spring AI Core -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
        </dependency>

        <!-- Vector Store -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
        </dependency>

        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- Database -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>

        <!-- Redis for Caching -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Monitoring -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.ai</groupId>
                <artifactId>spring-ai-bom</artifactId>
                <version>${spring-ai.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2. application.yml 설정 완전 가이드

#### application.yml이란?

Spring Boot 애플리케이션의 **설정 파일**입니다. 마치 **TV 리모컨 설정**과 같습니다:
- 채널 설정 (데이터베이스 주소)
- 볼륨 조절 (로그 레벨)
- 화질 설정 (캐시 크기)

#### 설정 파일 위치

```
src/
└── main/
    └── resources/
        ├── application.yml          (기본 설정)
        ├── application-dev.yml      (개발 환경)
        ├── application-prod.yml     (프로덕션 환경)
        └── application-test.yml     (테스트 환경)
```

#### 전체 설정 파일 (상세 주석 포함)

```yaml
# ============================================
# 애플리케이션 기본 정보
# ============================================
spring:
  application:
    name: springai-service  # 애플리케이션 이름 (로그, 모니터링에 표시)

  # ============================================
  # Spring AI 설정
  # ============================================
  ai:
    openai:
      # ===== OpenAI API 키 설정 =====
      # 환경 변수에서 로드 (보안상 코드에 직접 작성 금지!)
      # 설정 방법:
      #   Windows: set OPENAI_API_KEY=sk-...
      #   Linux/Mac: export OPENAI_API_KEY=sk-...
      #   IntelliJ: Run Configuration → Environment Variables
      api-key: ${OPENAI_API_KEY}

      # ===== 채팅 모델 설정 =====
      chat:
        options:
          # 사용할 GPT 모델 선택
          # gpt-4-turbo-preview: 최신, 빠름, 비쌈 (입력 $10/1M 토큰)
          # gpt-4: 안정적, 느림, 가장 비쌈 (입력 $30/1M 토큰)
          # gpt-3.5-turbo: 빠름, 저렴, 성능 낮음 (입력 $0.5/1M 토큰)
          model: gpt-4-turbo-preview

          # Temperature: 응답의 창의성/무작위성 (0.0 ~ 2.0)
          # 0.0: 항상 동일한 답변, 정확함 (FAQ, 데이터 추출)
          # 0.7: 균형잡힌 창의성 (일반 대화)
          # 1.5+: 매우 창의적, 예측 불가 (시 작성, 브레인스토밍)
          temperature: 0.7

          # 최대 응답 토큰 수
          # 토큰 ≈ 단어 개수 × 1.3 (한글은 더 많음)
          # 2000 토큰 ≈ 한글 1000자 정도
          # 너무 크면 비용 증가, 너무 작으면 답변 잘림
          max-tokens: 2000

      # ===== 임베딩 모델 설정 =====
      embedding:
        options:
          # 텍스트를 벡터로 변환하는 모델
          # text-embedding-3-small: 빠름, 저렴, 1536차원
          # text-embedding-3-large: 느림, 비쌈, 3072차원, 정확도 높음
          # 선택 기준: 일반적으로 small이면 충분
          model: text-embedding-3-small

  # ============================================
  # 데이터베이스 설정 (PostgreSQL)
  # ============================================
  datasource:
    # JDBC URL 형식: jdbc:postgresql://호스트:포트/데이터베이스명
    # localhost: 로컬 개발 환경
    # 프로덕션: RDS 엔드포인트 등으로 변경
    url: jdbc:postgresql://localhost:5432/aiservice

    # 보안: 환경 변수로 관리
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

    # PostgreSQL 드라이버 (자동 감지되지만 명시적으로 작성)
    driver-class-name: org.postgresql.Driver

    # ===== HikariCP 커넥션 풀 설정 =====
    # HikariCP란? 데이터베이스 연결을 미리 만들어두고 재사용
    # 비유: 식당의 테이블 - 손님이 올 때마다 테이블 만들지 않고 미리 준비
    hikari:
      # 최대 연결 수 (동시 처리 가능한 DB 요청 수)
      # 계산식: CPU 코어 수 × 2 + 디스크 수
      # 예: 4코어 × 2 + 1 = 9 → 여유있게 20
      # 너무 크면: 메모리 낭비, DB 부하
      # 너무 작으면: 대기 시간 증가
      maximum-pool-size: 20

      # 최소 유지 연결 수 (항상 준비된 연결)
      # idle(유휴) 상태로 유지할 연결 수
      # 급격한 트래픽 증가 시 빠른 응답
      minimum-idle: 5

      # 연결 타임아웃 (밀리초)
      # 풀에서 연결을 얻기까지 최대 대기 시간
      # 30000ms = 30초
      # 초과 시: SQLException 발생
      connection-timeout: 30000

      # 유휴 연결 타임아웃 (밀리초)
      # 사용하지 않는 연결을 얼마나 유지할지
      # 600000ms = 10분
      # 10분 동안 사용 안하면 연결 종료
      idle-timeout: 600000

      # 연결 최대 수명 (밀리초)
      # 연결 생성 후 최대 유지 시간
      # 1800000ms = 30분
      # DB 연결 리프레시 (메모리 누수 방지)
      max-lifetime: 1800000

  # ============================================
  # JPA/Hibernate 설정
  # ============================================
  jpa:
    hibernate:
      # DDL 자동 생성 전략
      # create: 기존 테이블 삭제 후 재생성 (개발 초기) ⚠️ 데이터 손실!
      # create-drop: 종료 시 테이블 삭제 (테스트)
      # update: 변경사항만 반영 (개발 중) ⚠️ 스키마 꼬일 수 있음
      # validate: 스키마 검증만 (프로덕션 권장) ✅
      # none: 아무것도 안함
      ddl-auto: validate

    # SQL 로그 출력 여부
    # true: 모든 SQL 쿼리 콘솔 출력 (개발 시 유용)
    # false: 출력 안함 (프로덕션) → logging.level로 제어
    show-sql: false

    properties:
      hibernate:
        # PostgreSQL 방언(Dialect) 지정
        # 각 DB마다 SQL 문법이 조금씩 다름
        # Hibernate가 PostgreSQL에 맞는 SQL 생성
        dialect: org.hibernate.dialect.PostgreSQLDialect

        # SQL 포맷팅 (가독성 향상)
        # 여러 줄로 정렬하여 출력
        format_sql: true

        # SQL에 주석 추가 (어떤 엔티티의 쿼리인지 표시)
        # /* load com.example.User */ SELECT ...
        use_sql_comments: true

        # ===== 배치 처리 최적화 =====
        jdbc:
          # 배치 크기 설정
          # 여러 INSERT/UPDATE를 묶어서 한번에 전송
          # 예: 100개 저장 시
          #     배치 없음: DB 왕복 100번
          #     배치 20: DB 왕복 5번 (20개씩)
          # 성능 향상: 최대 5~10배
          batch_size: 20

        # INSERT 순서 정렬 (배치 효율 향상)
        # 같은 테이블의 INSERT를 모아서 처리
        order_inserts: true

        # UPDATE 순서 정렬
        order_updates: true

  # ============================================
  # Redis 설정 (분산 캐시)
  # ============================================
  data:
    redis:
      # Redis 서버 주소
      # 로컬 개발: localhost
      # 프로덕션: ElastiCache, Redis Cloud 등
      host: localhost
      port: 6379  # Redis 기본 포트

      # Redis 비밀번호 (보안)
      # Redis 설치 시 requirepass 옵션으로 설정
      password: ${REDIS_PASSWORD}

      # Lettuce 커넥션 풀 설정
      # Lettuce란? Spring Boot의 기본 Redis 클라이언트
      # 비동기, 리액티브 지원
      lettuce:
        pool:
          # 최대 활성 연결 수
          # 동시에 Redis를 사용할 수 있는 최대 수
          max-active: 8

          # 최대 유휴 연결 수
          # 풀에 유지할 최대 대기 연결 수
          max-idle: 8

          # 최소 유휴 연결 수
          # 항상 준비된 연결 수 (빠른 응답)
          min-idle: 2

  # ============================================
  # 캐시 설정
  # ============================================
  cache:
    # 캐시 타입 선택
    # simple: 로컬 메모리 (단일 서버)
    # redis: Redis (분산 환경) ✅
    # caffeine: 고성능 로컬 캐시
    type: redis

    redis:
      # 캐시 TTL (Time To Live) - 밀리초
      # 3600000ms = 1시간
      # 1시간 후 자동 삭제 (최신 데이터 유지)
      #
      # 캐시 무효화 전략:
      # 1. TTL 자동 만료
      # 2. @CacheEvict로 수동 삭제
      # 3. Redis FLUSHDB (전체 삭제)
      time-to-live: 3600000 # 1 hour

# ============================================
# Actuator 모니터링 설정
# ============================================
management:
  endpoints:
    web:
      exposure:
        # 외부에 노출할 Actuator 엔드포인트
        # health: 서버 상태 (UP/DOWN)
        # info: 애플리케이션 정보
        # metrics: 성능 지표
        # prometheus: Prometheus 형식 메트릭
        #
        # ⚠️ 보안 주의: 프로덕션에서는 인증 추가 필요
        # Spring Security로 /actuator/** 경로 보호
        include: health,info,metrics,prometheus

        # 모든 엔드포인트 노출 (개발 환경만!)
        # include: "*"

  # Prometheus 메트릭 설정
  metrics:
    export:
      prometheus:
        enabled: true

        # 메트릭 수집 예시:
        # - JVM 메모리 사용량
        # - HTTP 요청 수/응답 시간
        # - DB 커넥션 풀 상태
        # - 캐시 히트/미스 비율
        # - 커스텀 메트릭 (chat.requests.total 등)

# ============================================
# 로깅 설정
# ============================================
logging:
  level:
    # 우리 애플리케이션 로그 레벨
    # TRACE < DEBUG < INFO < WARN < ERROR
    #
    # INFO: 일반적인 정보 (프로덕션 권장)
    #   "사용자 로그인", "API 호출 완료"
    # DEBUG: 상세한 디버깅 정보 (개발 환경)
    #   "변수 값: x=10", "메서드 실행 시작"
    # WARN: 경고 (잠재적 문제)
    #   "API 응답 느림 (3초)", "캐시 미스 빈번"
    # ERROR: 오류 (실제 문제 발생)
    #   "DB 연결 실패", "API 호출 실패"
    com.cloudai: INFO

    # Spring AI 라이브러리 로그
    # DEBUG: AI 호출 파라미터, 응답 내용 확인
    # 프로덕션: INFO로 변경 (로그 양 감소)
    org.springframework.ai: DEBUG

    # Hibernate SQL 로그
    # DEBUG: 실행되는 모든 SQL 쿼리 출력
    # 개발 시 유용 (N+1 쿼리 감지)
    # 프로덕션: WARN으로 변경
    org.hibernate.SQL: DEBUG

    # SQL 파라미터 값 출력 (더 상세한 디버깅)
    # org.hibernate.type.descriptor.sql.BasicBinder: TRACE

  # 로그 파일 설정 (선택사항)
  # file:
  #   name: logs/application.log  # 로그 파일 경로
  #   max-size: 10MB              # 파일 크기 제한
  #   max-history: 30             # 30일치 보관

  # 로그 패턴 커스터마이징
  # pattern:
  #   console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
  #   file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

### 3. 환경별 설정 관리

#### 설정 파일 분리 전략

```
src/main/resources/
├── application.yml              # 공통 설정 (모든 환경)
├── application-dev.yml          # 개발 환경 전용
├── application-prod.yml         # 프로덕션 환경 전용
└── application-test.yml         # 테스트 환경 전용
```

**application-dev.yml (개발 환경)**:
```yaml
spring:
  ai:
    openai:
      api-key: sk-dev-test-key  # 테스트용 키
      chat:
        options:
          model: gpt-3.5-turbo  # 저렴한 모델 사용

  datasource:
    url: jdbc:postgresql://localhost:5432/aiservice_dev

  jpa:
    hibernate:
      ddl-auto: update  # 개발 중 스키마 자동 업데이트
    show-sql: true      # SQL 로그 출력

logging:
  level:
    com.cloudai: DEBUG  # 상세 로그
```

**application-prod.yml (프로덕션)**:
```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}  # 환경 변수
      chat:
        options:
          model: gpt-4-turbo-preview  # 고성능 모델

  datasource:
    url: jdbc:postgresql://db.example.com:5432/aiservice_prod

  jpa:
    hibernate:
      ddl-auto: validate  # 검증만! 절대 auto-update 금지
    show-sql: false

logging:
  level:
    com.cloudai: INFO   # 필수 정보만
    org.springframework.ai: INFO
    org.hibernate.SQL: WARN
```

#### 프로파일 활성화 방법

**방법 1: application.yml에 지정**
```yaml
spring:
  profiles:
    active: dev  # 기본 프로파일
```

**방법 2: 실행 시 지정 (권장)**
```bash
# Maven
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Gradle
./gradlew bootRun --args='--spring.profiles.active=dev'

# JAR 실행
java -jar app.jar --spring.profiles.active=prod

# 환경 변수
export SPRING_PROFILES_ACTIVE=prod
java -jar app.jar
```

**방법 3: IntelliJ IDEA**
```
1. Run → Edit Configurations
2. Active profiles에 "dev" 입력
3. Run
```

### 4. 환경 변수 관리

#### 민감 정보 보호

**절대 하지 말아야 할 것 ❌**:
```yaml
# ❌ 코드에 직접 작성 - Git에 올라가면 위험!
spring:
  ai:
    openai:
      api-key: sk-proj-abc123...  # 노출 위험!

  datasource:
    password: mypassword123       # 보안 문제!
```

**올바른 방법 ✅**:

**1. 환경 변수 사용**
```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}  # 환경 변수에서 로드

  datasource:
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

**2. .env 파일 (로컬 개발)**
```bash
# .env 파일 생성 (⚠️ .gitignore에 추가!)
OPENAI_API_KEY=sk-proj-...
DB_USERNAME=postgres
DB_PASSWORD=mysecret
REDIS_PASSWORD=redis123
```

**3. 시스템 환경 변수 설정**

Windows:
```cmd
set OPENAI_API_KEY=sk-proj-...
set DB_USERNAME=postgres
set DB_PASSWORD=mysecret
```

Linux/Mac:
```bash
export OPENAI_API_KEY=sk-proj-...
export DB_USERNAME=postgres
export DB_PASSWORD=mysecret
```

**4. Docker Compose**
```yaml
services:
  springai-service:
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
```

**5. Kubernetes Secrets**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  openai-api-key: <base64-encoded>
  db-password: <base64-encoded>
```

### 5. 설정 검증

#### 설정이 올바른지 확인하기

**1. 애플리케이션 시작 로그 확인**
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.2.0)

✅ 성공 시:
Started SpringAiServiceApplication in 3.5 seconds

❌ 실패 시:
Failed to configure a DataSource: 'url' attribute is not specified
→ datasource.url 설정 누락!
```

**2. Actuator로 설정 확인**
```bash
# 헬스 체크
curl http://localhost:8080/actuator/health

{
  "status": "UP",
  "components": {
    "db": {"status": "UP"},      ✅ DB 연결 OK
    "redis": {"status": "UP"},   ✅ Redis 연결 OK
    "diskSpace": {"status": "UP"}
  }
}

# 설정 정보 (개발 환경만!)
curl http://localhost:8080/actuator/env
```

**3. 연결 테스트 코드**
```java
@SpringBootTest
class ConfigurationTest {

    @Autowired
    private DataSource dataSource;

    @Autowired
    private ChatClient chatClient;

    @Test
    void testDatabaseConnection() throws Exception {
        try (Connection conn = dataSource.getConnection()) {
            assertTrue(conn.isValid(5));  // 5초 내 연결 확인
        }
    }

    @Test
    void testOpenAIConnection() {
        String response = chatClient.prompt()
            .user("Hello")
            .call()
            .content();

        assertNotNull(response);  // 응답 받음
    }
}
```

### 설정 완료 체크리스트

```
□ pom.xml 의존성 추가 완료
□ application.yml 작성 완료
□ 환경 변수 설정 완료 (OPENAI_API_KEY, DB_PASSWORD 등)
□ PostgreSQL 설치 및 실행 (Docker 또는 로컬)
□ Redis 설치 및 실행 (Docker 또는 로컬)
□ 애플리케이션 시작 성공 (mvn spring-boot:run)
□ Actuator 헬스 체크 통과 (http://localhost:8080/actuator/health)
□ 첫 API 호출 테스트 성공
```

다음 섹션에서는 이제 실제 코드 구현을 시작합니다!

---

## 핵심 구현 상세 설명

### 구현 전 아키텍처 복습

우리가 구현할 계층 구조:
```
사용자 요청
    ↓
[Controller] ← HTTP 요청을 받아서 JSON으로 반환
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
1. Domain (데이터 구조 정의)
2. Repository (데이터 저장/조회)
3. Service (비즈니스 로직)
4. Controller (API 엔드포인트)
```

왜 이 순서? 집을 지을 때 **기초부터** 쌓듯이, 데이터 구조부터 시작합니다.

### 1. 도메인 모델 - 데이터 구조 설계

#### 도메인 모델이란?

**일반 설명**: 우리 애플리케이션에서 다루는 데이터의 **청사진**

**실생활 비유**:
- 도서관 회원카드 양식 (이름, 회원번호, 가입일 등)
- 은행 통장 양식 (계좌번호, 잔액, 거래내역 등)

**우리 AI 서비스의 데이터**:
1. **Conversation**: 한 번의 대화 (사용자 질문 + AI 답변)
2. **ConversationSummary**: 여러 대화의 요약
3. **VectorMemory**: 임베딩 벡터로 저장된 기억

#### Conversation 엔티티 상세 설명

```java
// ============================================
// Conversation.java
// 역할: 사용자와 AI의 한 번의 대화를 저장하는 엔티티
// ============================================

// @Entity: JPA에게 "이 클래스는 DB 테이블과 매핑됩니다"라고 알림
// → Hibernate가 자동으로 'conversations' 테이블 생성/관리
@Entity

// @Table: 테이블 세부 설정
@Table(name = "conversations",  // 테이블명 명시
    indexes = {
        // 인덱스 추가 (검색 성능 향상)
        // 왜 필요? "특정 사용자의 최근 대화 10개"를 빠르게 조회하기 위해
        //
        // 인덱스 없이: 전체 테이블 스캔 (100만 건 중 검색 → 느림)
        // 인덱스 있음: B-Tree 검색 (100만 건 중 검색 → 빠름)
        //
        // idx_user_created: (user_id, created_at) 복합 인덱스
        // → "user123의 최근 대화" 쿼리 최적화
        @Index(name = "idx_user_created", columnList = "user_id, created_at"),

        // idx_session: session_id 인덱스
        // → "특정 세션의 모든 대화" 조회 최적화
        @Index(name = "idx_session", columnList = "session_id")
    }
)

// Lombok 어노테이션 - 보일러플레이트 코드 자동 생성
@Getter                // getId(), getUserId() 등 자동 생성
@Setter                // setId(), setUserId() 등 자동 생성
@NoArgsConstructor     // 기본 생성자 생성 (JPA 필수)
public class Conversation {

    // ===== 기본 키 (Primary Key) =====
    @Id  // 이 필드가 테이블의 기본 키임을 표시
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    // IDENTITY: DB가 자동으로 ID 생성 (AUTO_INCREMENT)
    // 1, 2, 3, 4... 자동 증가
    private Long id;

    // ===== 사용자 식별 정보 =====
    @Column(nullable = false)  // NULL 불가 (필수 값)
    // 사용자 고유 ID (UUID, 이메일 등)
    // 예: "user-123", "john@example.com"
    private String userId;

    @Column(nullable = false)
    // 세션 ID (하나의 대화 세션을 구분)
    // 예: "session-456", UUID.randomUUID()
    // 같은 사용자도 여러 세션 가능 (웹/모바일 각각)
    private String sessionId;

    // ===== 대화 내용 =====
    @Column(nullable = false, columnDefinition = "TEXT")
    // columnDefinition = "TEXT": 긴 텍스트 저장
    // VARCHAR(255) 기본값 → TEXT 타입으로 변경
    // → 4KB 이상의 긴 메시지 저장 가능
    private String userMessage;  // 사용자 질문

    @Column(nullable = false, columnDefinition = "TEXT")
    private String assistantMessage;  // AI 응답

    // ===== 메타데이터 (추가 정보) =====
    @Column(columnDefinition = "jsonb")
    // jsonb: PostgreSQL의 JSON Binary 타입
    // → JSON 형식 데이터를 효율적으로 저장/검색
    // 예: {"model": "gpt-4", "temperature": 0.7, "ip": "192.168.1.1"}
    //
    // 왜 jsonb?
    // - 스키마 변경 없이 메타데이터 추가 가능
    // - JSON 내부 필드 검색 가능 (WHERE metadata->'model' = 'gpt-4')
    private String metadata;

    // ===== 시간 정보 =====
    @CreatedDate  // Spring Data JPA가 자동으로 생성 시간 입력
    @Column(nullable = false, updatable = false)
    // updatable = false: 한번 저장되면 수정 불가 (생성 시간은 변경 안됨)
    private LocalDateTime createdAt;

    // ===== 성능 메트릭 =====
    @Column
    // 사용한 토큰 수 (비용 계산용)
    // OpenAI API 응답의 usage.total_tokens 값 저장
    private Integer tokenCount;

    @Column
    // AI 응답 시간 (밀리초)
    // 성능 모니터링 및 최적화에 활용
    // 예: 3000ms → "3초 걸렸네? 최적화 필요!"
    private Long responseTimeMs;
}

/*
=============================================================
생성되는 SQL (PostgreSQL)
=============================================================
CREATE TABLE conversations (
    id BIGSERIAL PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    session_id VARCHAR(255) NOT NULL,
    user_message TEXT NOT NULL,
    assistant_message TEXT NOT NULL,
    metadata JSONB,
    created_at TIMESTAMP NOT NULL,
    token_count INTEGER,
    response_time_ms BIGINT
);

CREATE INDEX idx_user_created ON conversations(user_id, created_at);
CREATE INDEX idx_session ON conversations(session_id);
*/

// ConversationSummary.java
@Entity
@Table(name = "conversation_summaries")
@Getter
@Setter
@NoArgsConstructor
public class ConversationSummary {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String userId;

    @Column(nullable = false)
    private String sessionId;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String summary;

    @Column
    private LocalDateTime summaryPeriodStart;

    @Column
    private LocalDateTime summaryPeriodEnd;

    @Column
    private Integer messageCount;

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
}

// VectorMemory.java
@Entity
@Table(name = "vector_memories")
@Getter
@Setter
@NoArgsConstructor
public class VectorMemory {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String userId;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String content;

    @Column(columnDefinition = "vector(1536)") // For OpenAI embeddings
    private String embedding;

    @Column
    private String category;

    @Column
    private Double relevanceScore;

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
}
```

### 2. Repository Layer

```java
// ConversationRepository.java
@Repository
public interface ConversationRepository extends JpaRepository<Conversation, Long> {

    List<Conversation> findByUserIdAndSessionIdOrderByCreatedAtDesc(
        String userId, String sessionId, Pageable pageable
    );

    @Query("SELECT c FROM Conversation c WHERE c.userId = :userId " +
           "AND c.createdAt >= :since ORDER BY c.createdAt DESC")
    List<Conversation> findRecentByUserId(
        @Param("userId") String userId,
        @Param("since") LocalDateTime since,
        Pageable pageable
    );

    @Modifying
    @Query("DELETE FROM Conversation c WHERE c.createdAt < :before")
    int deleteOldConversations(@Param("before") LocalDateTime before);
}

// ConversationSummaryRepository.java
@Repository
public interface ConversationSummaryRepository
    extends JpaRepository<ConversationSummary, Long> {

    Optional<ConversationSummary> findTopByUserIdAndSessionIdOrderByCreatedAtDesc(
        String userId, String sessionId
    );

    List<ConversationSummary> findByUserIdOrderByCreatedAtDesc(
        String userId, Pageable pageable
    );
}
```

### 3. Service Layer

```java
// ChatService.java
@Service
@Slf4j
public class ChatService {

    private final ChatClient chatClient;
    private final ConversationRepository conversationRepository;
    private final ConversationSummaryRepository summaryRepository;
    private final VectorStoreService vectorStoreService;
    private final CacheManager cacheManager;

    public ChatService(ChatClient.Builder chatClientBuilder,
                      ConversationRepository conversationRepository,
                      ConversationSummaryRepository summaryRepository,
                      VectorStoreService vectorStoreService,
                      CacheManager cacheManager) {
        this.chatClient = chatClientBuilder.build();
        this.conversationRepository = conversationRepository;
        this.summaryRepository = summaryRepository;
        this.vectorStoreService = vectorStoreService;
        this.cacheManager = cacheManager;
    }

    @Transactional
    public ChatResponse chat(ChatRequest request) {
        long startTime = System.currentTimeMillis();

        // 1. 대화 히스토리 로드
        List<Message> history = loadConversationHistory(
            request.getUserId(),
            request.getSessionId()
        );

        // 2. 벡터 메모리에서 관련 컨텍스트 검색
        List<String> relevantMemories = vectorStoreService
            .searchRelevantMemories(request.getUserId(), request.getMessage(), 3);

        // 3. 시스템 프롬프트 구성
        String systemPrompt = buildSystemPrompt(relevantMemories);

        // 4. LLM 호출
        ChatResponse response = chatClient.prompt()
            .system(systemPrompt)
            .messages(history)
            .user(request.getMessage())
            .call()
            .chatResponse();

        String assistantMessage = response.getResult().getOutput().getContent();

        // 5. 대화 저장
        Conversation conversation = saveConversation(
            request.getUserId(),
            request.getSessionId(),
            request.getMessage(),
            assistantMessage,
            System.currentTimeMillis() - startTime,
            response.getMetadata()
        );

        // 6. 벡터 메모리 업데이트 (비동기)
        updateVectorMemoryAsync(request.getUserId(), conversation);

        // 7. 주기적 요약 체크
        checkAndSummarize(request.getUserId(), request.getSessionId());

        return response;
    }

    @Cacheable(value = "conversationHistory",
               key = "#userId + ':' + #sessionId")
    private List<Message> loadConversationHistory(String userId, String sessionId) {
        List<Conversation> recent = conversationRepository
            .findByUserIdAndSessionIdOrderByCreatedAtDesc(
                userId,
                sessionId,
                PageRequest.of(0, 10)
            );

        List<Message> messages = new ArrayList<>();

        // 최신 요약 추가
        summaryRepository
            .findTopByUserIdAndSessionIdOrderByCreatedAtDesc(userId, sessionId)
            .ifPresent(summary ->
                messages.add(new SystemMessage("이전 대화 요약: " + summary.getSummary()))
            );

        // 최근 대화 추가 (역순 정렬)
        Collections.reverse(recent);
        for (Conversation conv : recent) {
            messages.add(new UserMessage(conv.getUserMessage()));
            messages.add(new AssistantMessage(conv.getAssistantMessage()));
        }

        return messages;
    }

    private String buildSystemPrompt(List<String> relevantMemories) {
        StringBuilder sb = new StringBuilder();
        sb.append("당신은 도움이 되는 AI 어시스턴트입니다.\n\n");

        if (!relevantMemories.isEmpty()) {
            sb.append("사용자에 대한 관련 정보:\n");
            relevantMemories.forEach(memory ->
                sb.append("- ").append(memory).append("\n")
            );
            sb.append("\n");
        }

        sb.append("위 정보를 참고하여 개인화된 응답을 제공하세요.");
        return sb.toString();
    }

    private Conversation saveConversation(String userId, String sessionId,
                                         String userMessage, String assistantMessage,
                                         long responseTime, ChatResponseMetadata metadata) {
        Conversation conversation = new Conversation();
        conversation.setUserId(userId);
        conversation.setSessionId(sessionId);
        conversation.setUserMessage(userMessage);
        conversation.setAssistantMessage(assistantMessage);
        conversation.setResponseTimeMs(responseTime);
        conversation.setTokenCount(metadata.getUsage().getTotalTokens());
        conversation.setCreatedAt(LocalDateTime.now());

        // 캐시 무효화
        cacheManager.getCache("conversationHistory")
            .evict(userId + ":" + sessionId);

        return conversationRepository.save(conversation);
    }

    @Async
    protected void updateVectorMemoryAsync(String userId, Conversation conversation) {
        try {
            vectorStoreService.storeConversation(userId, conversation);
        } catch (Exception e) {
            log.error("Failed to update vector memory", e);
        }
    }

    private void checkAndSummarize(String userId, String sessionId) {
        List<Conversation> recent = conversationRepository
            .findByUserIdAndSessionIdOrderByCreatedAtDesc(
                userId,
                sessionId,
                PageRequest.of(0, 50)
            );

        // 50개 이상의 메시지가 쌓이면 요약
        if (recent.size() >= 50) {
            summarizeConversations(userId, sessionId, recent);
        }
    }

    @Transactional
    protected void summarizeConversations(String userId, String sessionId,
                                        List<Conversation> conversations) {
        StringBuilder conversationText = new StringBuilder();
        for (Conversation conv : conversations) {
            conversationText.append("User: ").append(conv.getUserMessage()).append("\n");
            conversationText.append("Assistant: ").append(conv.getAssistantMessage()).append("\n\n");
        }

        String summaryPrompt = "다음 대화 내용을 핵심 포인트 중심으로 요약해주세요:\n\n" +
                              conversationText.toString();

        String summary = chatClient.prompt()
            .user(summaryPrompt)
            .call()
            .content();

        ConversationSummary summaryEntity = new ConversationSummary();
        summaryEntity.setUserId(userId);
        summaryEntity.setSessionId(sessionId);
        summaryEntity.setSummary(summary);
        summaryEntity.setSummaryPeriodStart(conversations.get(conversations.size() - 1).getCreatedAt());
        summaryEntity.setSummaryPeriodEnd(conversations.get(0).getCreatedAt());
        summaryEntity.setMessageCount(conversations.size());
        summaryEntity.setCreatedAt(LocalDateTime.now());

        summaryRepository.save(summaryEntity);

        // 요약된 대화는 삭제 (최근 10개만 유지)
        if (conversations.size() > 10) {
            List<Long> idsToDelete = conversations.subList(10, conversations.size())
                .stream()
                .map(Conversation::getId)
                .toList();
            conversationRepository.deleteAllById(idsToDelete);
        }

        log.info("Summarized {} conversations for user {} in session {}",
                conversations.size(), userId, sessionId);
    }
}
```

### 4. Vector Store Service

```java
@Service
@Slf4j
public class VectorStoreService {

    private final VectorStore vectorStore;
    private final EmbeddingClient embeddingClient;

    public VectorStoreService(VectorStore vectorStore,
                             EmbeddingClient embeddingClient) {
        this.vectorStore = vectorStore;
        this.embeddingClient = embeddingClient;
    }

    public void storeConversation(String userId, Conversation conversation) {
        String content = String.format(
            "User: %s\nAssistant: %s",
            conversation.getUserMessage(),
            conversation.getAssistantMessage()
        );

        Map<String, Object> metadata = new HashMap<>();
        metadata.put("userId", userId);
        metadata.put("sessionId", conversation.getSessionId());
        metadata.put("createdAt", conversation.getCreatedAt().toString());
        metadata.put("conversationId", conversation.getId());

        Document document = new Document(content, metadata);
        vectorStore.add(List.of(document));

        log.debug("Stored conversation {} in vector store", conversation.getId());
    }

    public List<String> searchRelevantMemories(String userId, String query, int topK) {
        SearchRequest searchRequest = SearchRequest.query(query)
            .withTopK(topK)
            .withSimilarityThreshold(0.7)
            .withFilterExpression("userId == '" + userId + "'");

        List<Document> results = vectorStore.similaritySearch(searchRequest);

        return results.stream()
            .map(Document::getContent)
            .toList();
    }

    public void deleteUserMemories(String userId) {
        vectorStore.delete(List.of(userId));
        log.info("Deleted all memories for user {}", userId);
    }
}
```

### 5. Controller Layer

```java
@RestController
@RequestMapping("/api/v1/chat")
@Slf4j
public class ChatController {

    private final ChatService chatService;

    public ChatController(ChatService chatService) {
        this.chatService = chatService;
    }

    @PostMapping
    public ResponseEntity<ChatResponseDto> chat(
            @Valid @RequestBody ChatRequestDto request,
            @RequestHeader("X-User-Id") String userId) {

        log.info("Received chat request from user: {}", userId);

        ChatRequest chatRequest = ChatRequest.builder()
            .userId(userId)
            .sessionId(request.getSessionId())
            .message(request.getMessage())
            .build();

        ChatResponse response = chatService.chat(chatRequest);

        ChatResponseDto responseDto = ChatResponseDto.builder()
            .message(response.getResult().getOutput().getContent())
            .sessionId(request.getSessionId())
            .metadata(buildMetadata(response.getMetadata()))
            .build();

        return ResponseEntity.ok(responseDto);
    }

    @GetMapping("/history")
    public ResponseEntity<ConversationHistoryDto> getHistory(
            @RequestHeader("X-User-Id") String userId,
            @RequestParam String sessionId,
            @RequestParam(defaultValue = "20") int limit) {

        // Implementation omitted for brevity
        return ResponseEntity.ok(new ConversationHistoryDto());
    }

    private Map<String, Object> buildMetadata(ChatResponseMetadata metadata) {
        Map<String, Object> result = new HashMap<>();
        result.put("model", metadata.getModel());
        result.put("totalTokens", metadata.getUsage().getTotalTokens());
        result.put("promptTokens", metadata.getUsage().getPromptTokens());
        result.put("completionTokens", metadata.getUsage().getGenerationTokens());
        return result;
    }
}
```

---

## 장기 메모리 관리

### Python/FastAPI vs Java/Spring AI 메모리 관리 비교

#### Python/FastAPI 접근법

```python
# 일반적인 Python 구현
class MemoryManager:
    def __init__(self, db_session, redis_client):
        self.db = db_session
        self.redis = redis_client

    async def store_conversation(self, user_id: str, message: dict):
        # 1. DB 저장 - 각 호출마다 새 연결
        await self.db.execute(
            "INSERT INTO conversations (...) VALUES (...)",
            message
        )
        await self.db.commit()

        # 2. 캐시 무효화 - 수동 관리
        await self.redis.delete(f"history:{user_id}")

    async def get_history(self, user_id: str, limit: int = 10):
        # 캐시 확인
        cached = await self.redis.get(f"history:{user_id}")
        if cached:
            return json.loads(cached)

        # DB 조회
        result = await self.db.execute(
            "SELECT * FROM conversations WHERE user_id = ? LIMIT ?",
            (user_id, limit)
        )
        history = result.fetchall()

        # 캐시 저장
        await self.redis.setex(
            f"history:{user_id}",
            3600,
            json.dumps(history)
        )
        return history
```

**단점:**
- 트랜잭션 관리 수동 처리
- N+1 쿼리 문제 발생 가능
- 캐시 일관성 보장 어려움
- 타입 안정성 부족

#### Java/Spring AI 접근법

```java
@Service
@Transactional
@Slf4j
public class EnhancedMemoryService {

    private final ConversationRepository conversationRepository;
    private final EntityManager entityManager;

    @Cacheable(value = "conversationHistory",
               key = "#userId + ':' + #sessionId",
               unless = "#result.isEmpty()")
    public List<ConversationDto> getConversationHistory(
            String userId, String sessionId, int limit) {

        // JPA Criteria API로 타입 안전한 쿼리
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Conversation> cq = cb.createQuery(Conversation.class);
        Root<Conversation> root = cq.from(Conversation.class);

        cq.select(root)
          .where(
              cb.equal(root.get("userId"), userId),
              cb.equal(root.get("sessionId"), sessionId)
          )
          .orderBy(cb.desc(root.get("createdAt")));

        List<Conversation> results = entityManager
            .createQuery(cq)
            .setMaxResults(limit)
            .setHint("org.hibernate.cacheable", true)
            .getResultList();

        return results.stream()
            .map(this::toDto)
            .toList();
    }

    @CacheEvict(value = "conversationHistory",
                key = "#conversation.userId + ':' + #conversation.sessionId")
    public Conversation saveConversation(Conversation conversation) {
        return conversationRepository.save(conversation);
    }

    // 배치 저장으로 성능 최적화
    @Transactional
    public void saveConversationsBatch(List<Conversation> conversations) {
        int batchSize = 20;
        for (int i = 0; i < conversations.size(); i++) {
            entityManager.persist(conversations.get(i));

            if (i > 0 && i % batchSize == 0) {
                entityManager.flush();
                entityManager.clear();
            }
        }
    }

    // 계층적 메모리 전략: 단기/장기 메모리 분리
    public MemoryContext buildMemoryContext(String userId, String sessionId) {
        // 단기 메모리: 최근 10개 대화
        List<Conversation> shortTerm = conversationRepository
            .findByUserIdAndSessionIdOrderByCreatedAtDesc(
                userId, sessionId, PageRequest.of(0, 10)
            );

        // 장기 메모리: 요약된 과거 대화
        List<ConversationSummary> longTerm = summaryRepository
            .findByUserIdOrderByCreatedAtDesc(userId, PageRequest.of(0, 5));

        // 벡터 메모리: 관련성 기반 검색
        List<VectorMemory> vectorMemories = vectorMemoryRepository
            .findRelevantMemories(userId, /* current context */, 3);

        return MemoryContext.builder()
            .shortTermMemory(shortTerm)
            .longTermMemory(longTerm)
            .vectorMemory(vectorMemories)
            .build();
    }
}
```

### 장기 메모리 관리 장점

#### 1. 트랜잭션 무결성

```java
@Service
public class TransactionalMemoryService {

    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void updateMemoryWithRollback(String userId,
                                        List<Conversation> conversations) {
        try {
            // 1. 대화 저장
            conversationRepository.saveAll(conversations);

            // 2. 요약 생성
            ConversationSummary summary = generateSummary(conversations);
            summaryRepository.save(summary);

            // 3. 벡터 인덱스 업데이트
            vectorStoreService.indexConversations(userId, conversations);

            // 모든 작업이 성공하면 커밋
        } catch (Exception e) {
            // 실패시 자동 롤백
            log.error("Memory update failed, rolling back", e);
            throw new MemoryUpdateException("Failed to update memory", e);
        }
    }
}
```

**Python에서 동일한 작업:**

```python
# 수동 트랜잭션 관리 필요
async def update_memory_with_rollback(user_id, conversations):
    async with db.begin() as transaction:
        try:
            await save_conversations(conversations)
            summary = await generate_summary(conversations)
            await save_summary(summary)
            await index_vectors(user_id, conversations)
            await transaction.commit()
        except Exception as e:
            await transaction.rollback()
            raise
```

#### 2. 2차 캐시 및 쿼리 캐시

```java
// Hibernate 2차 캐시 설정
@Entity
@Cacheable
@org.hibernate.annotations.Cache(
    usage = CacheConcurrencyStrategy.READ_WRITE,
    region = "conversations"
)
public class Conversation {
    // ...
}

// 쿼리 결과 캐싱
@QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
List<Conversation> findByUserId(String userId);
```

**성능 이점:**
- 1차 캐시: 세션 레벨 자동 캐싱
- 2차 캐시: 애플리케이션 레벨 캐싱
- 쿼리 캐시: 쿼리 결과 캐싱
- Redis 캐시: 분산 캐싱

```
Python/FastAPI (Redis만 사용):
- 캐시 히트: 응답 시간 5ms
- 캐시 미스: 응답 시간 45ms

Java/Spring (다층 캐시):
- L1 캐시 히트: 응답 시간 <1ms
- L2 캐시 히트: 응답 시간 2ms
- Redis 히트: 응답 시간 5ms
- DB 쿼리: 응답 시간 25ms (배치 최적화)
```

#### 3. Lazy Loading 및 성능 최적화

```java
@Entity
public class Conversation {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // 기본 필드들...

    @OneToMany(mappedBy = "conversation", fetch = FetchType.LAZY)
    @BatchSize(size = 10)  // N+1 문제 해결
    private List<ConversationMetadata> metadata;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "summary_id")
    private ConversationSummary summary;
}

// EntityGraph로 필요한 데이터만 Eager 로딩
@EntityGraph(attributePaths = {"metadata", "summary"})
List<Conversation> findDetailedByUserId(String userId);
```

#### 4. 메모리 압축 및 아카이빙

```java
@Service
public class MemoryArchiveService {

    @Scheduled(cron = "0 0 2 * * *")  // 매일 새벽 2시
    @Transactional
    public void archiveOldConversations() {
        LocalDateTime archiveThreshold = LocalDateTime.now().minusMonths(3);

        // 1. 오래된 대화 조회
        List<Conversation> oldConversations = conversationRepository
            .findByCreatedAtBefore(archiveThreshold);

        // 2. 압축 및 아카이브
        List<ArchivedConversation> archived = oldConversations.stream()
            .collect(Collectors.groupingBy(Conversation::getUserId))
            .entrySet().stream()
            .map(entry -> compressAndArchive(entry.getKey(), entry.getValue()))
            .toList();

        archivedRepository.saveAll(archived);

        // 3. 원본 삭제
        conversationRepository.deleteAll(oldConversations);

        log.info("Archived {} conversations", oldConversations.size());
    }

    private ArchivedConversation compressAndArchive(String userId,
                                                   List<Conversation> conversations) {
        String compressed = compressionService.compress(
            conversations.stream()
                .map(this::serialize)
                .collect(Collectors.joining("\n"))
        );

        return ArchivedConversation.builder()
            .userId(userId)
            .compressedData(compressed)
            .conversationCount(conversations.size())
            .archivedAt(LocalDateTime.now())
            .build();
    }
}
```

### 메모리 효율성 비교

| 측면 | Python/FastAPI | Java/Spring AI |
|------|----------------|----------------|
| ORM 성능 | SQLAlchemy: 기본 성능 | Hibernate: 배치 최적화, 2차 캐시 |
| 메모리 사용 | 1만 대화: ~150MB | 1만 대화: ~120MB (압축) |
| 쿼리 최적화 | 수동 최적화 필요 | 자동 배치, 지연 로딩 |
| 캐시 계층 | 1계층 (Redis) | 3계층 (L1, L2, Redis) |
| 트랜잭션 | 수동 관리 | 선언적 관리 (@Transactional) |
| 대용량 처리 | 메모리 압박 발생 | 스트리밍, 페이지네이션 내장 |

---

## 성능 최적화

### 1. 연결 풀 튜닝

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      leak-detection-threshold: 60000
```

### 2. JVM 튜닝

```bash
# application 실행시 JVM 옵션
java -Xms2g -Xmx4g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:+UseStringDeduplication \
     -XX:+OptimizeStringConcat \
     -jar springai-service.jar
```

### 3. 비동기 처리

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}

@Service
public class AsyncChatService {

    @Async
    public CompletableFuture<String> processLongRunningTask(String input) {
        // 비동기 처리
        return CompletableFuture.completedFuture(result);
    }
}
```

---

## 프로덕션 배포

### Dockerfile

```dockerfile
# Multi-stage build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline

COPY src src
RUN ./mvnw package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", \
    "-Xms2g", \
    "-Xmx4g", \
    "-XX:+UseG1GC", \
    "-XX:MaxGCPauseMillis=200", \
    "-Djava.security.egd=file:/dev/./urandom", \
    "-jar", \
    "app.jar"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  springai-service:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    depends_on:
      - postgres
      - redis
    networks:
      - springai-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgres:
    image: pgvector/pgvector:pg16
    environment:
      - POSTGRES_DB=aiservice
      - POSTGRES_USER=${DB_USERNAME}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - springai-network

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - springai-network

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - springai-network

volumes:
  postgres-data:
  redis-data:
  prometheus-data:

networks:
  springai-network:
    driver: bridge
```

---

## 모니터링 및 운영

### Prometheus 메트릭

```java
@Component
public class ChatMetrics {

    private final Counter chatRequests;
    private final Timer chatDuration;
    private final Gauge activeConversations;

    public ChatMetrics(MeterRegistry registry) {
        this.chatRequests = Counter.builder("chat.requests.total")
            .description("Total number of chat requests")
            .tag("type", "ai")
            .register(registry);

        this.chatDuration = Timer.builder("chat.duration")
            .description("Chat request duration")
            .register(registry);

        this.activeConversations = Gauge.builder("chat.active.conversations")
            .description("Number of active conversations")
            .register(registry);
    }

    public void recordChatRequest() {
        chatRequests.increment();
    }

    public void recordChatDuration(long milliseconds) {
        chatDuration.record(milliseconds, TimeUnit.MILLISECONDS);
    }
}
```

### 로깅 전략

```java
@Aspect
@Component
@Slf4j
public class ChatLoggingAspect {

    @Around("@annotation(Loggable)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();

        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();

        log.info("Executing {}.{}() with args: {}",
            joinPoint.getTarget().getClass().getSimpleName(),
            methodName,
            maskSensitiveData(args));

        try {
            Object result = joinPoint.proceed();
            long duration = System.currentTimeMillis() - start;

            log.info("Completed {}.{}() in {}ms",
                joinPoint.getTarget().getClass().getSimpleName(),
                methodName,
                duration);

            return result;
        } catch (Exception e) {
            log.error("Failed {}.{}(): {}",
                joinPoint.getTarget().getClass().getSimpleName(),
                methodName,
                e.getMessage(), e);
            throw e;
        }
    }

    private Object[] maskSensitiveData(Object[] args) {
        // API 키 등 민감 정보 마스킹
        return args;
    }
}
```

---

## FAQ

### Q1: Java/Spring AI 환경에서 모델 추론 성능을 극대화하기 위한 JVM 튜닝과 네이티브 이미지(GraalVM) 컴파일 전략은?

**JVM 튜닝 전략:**

```bash
# G1GC 튜닝 (대규모 힙)
java -Xms4g -Xmx8g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:G1HeapRegionSize=16M \
     -XX:InitiatingHeapOccupancyPercent=45 \
     -XX:G1ReservePercent=15 \
     -XX:+UseStringDeduplication \
     -jar app.jar

# ZGC 튜닝 (초저지연)
java -Xms8g -Xmx8g \
     -XX:+UseZGC \
     -XX:+ZGenerational \
     -XX:ConcGCThreads=2 \
     -jar app.jar
```

**GraalVM 네이티브 이미지:**

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
</plugin>
```

```bash
# 네이티브 이미지 빌드
./mvnw -Pnative native:compile

# 성능 비교
JVM: 시작 시간 2-3초, 메모리 512MB
Native: 시작 시간 0.05초, 메모리 128MB
```

**콜드 스타트 개선:**
- CDS (Class Data Sharing) 활성화
- AppCDS로 애플리케이션 클래스 미리 로드
- Ahead-of-Time (AOT) 컴파일 활용

### Q2: Spring AI와 LangChain4j 비교

| 측면 | Spring AI | LangChain4j |
|------|-----------|-------------|
| **통합** | Spring Boot 네이티브 | 독립 라이브러리 |
| **학습 곡선** | Spring 개발자에게 친숙 | 별도 학습 필요 |
| **기능** | 기본 기능 중심 | 고급 기능 풍부 |
| **RAG 지원** | 기본 VectorStore 제공 | 다양한 RAG 패턴 |
| **커뮤니티** | 성장 중 | 활발함 |
| **유지보수** | VMware/Broadcom 지원 | 커뮤니티 중심 |

**선택 기준:**
- **Spring AI**: 기존 Spring 프로젝트, 단순한 요구사항
- **LangChain4j**: 복잡한 RAG, 에이전트 시스템, 다양한 LLM 통합

**RAG 파이프라인 비교:**

```java
// Spring AI RAG
@Service
public class SpringAIRAG {
    @Autowired
    private VectorStore vectorStore;

    public String query(String question) {
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.query(question).withTopK(3)
        );
        String context = docs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n"));
        return chatClient.prompt()
            .system("Context: " + context)
            .user(question)
            .call()
            .content();
    }
}

// LangChain4j RAG
@Service
public class LangChain4jRAG {
    private final ConversationalRetrievalChain chain;

    public String query(String question) {
        return chain.execute(question);
    }
}
```

### Q3: 벡터 데이터베이스 통합 및 메모리 이슈 해결

**벡터 DB 통합 예시:**

```java
// Pinecone 통합
@Configuration
public class PineconeConfig {

    @Bean
    public PineconeVectorStore pineconeVectorStore(
            @Value("${pinecone.api-key}") String apiKey,
            @Value("${pinecone.environment}") String environment,
            EmbeddingClient embeddingClient) {

        PineconeVectorStoreConfig config = PineconeVectorStoreConfig.builder()
            .withApiKey(apiKey)
            .withEnvironment(environment)
            .withIndexName("conversations")
            .withNamespace("production")
            .build();

        return new PineconeVectorStore(config, embeddingClient);
    }
}

// Weaviate 통합
@Configuration
public class WeaviateConfig {

    @Bean
    public WeaviateVectorStore weaviateVectorStore(
            @Value("${weaviate.scheme}") String scheme,
            @Value("${weaviate.host}") String host,
            EmbeddingClient embeddingClient) {

        WeaviateVectorStoreConfig config = WeaviateVectorStoreConfig.builder()
            .withScheme(scheme)
            .withHost(host)
            .withObjectClass("Conversation")
            .build();

        return new WeaviateVectorStore(config, embeddingClient);
    }
}
```

**대용량 임베딩 처리 전략:**

```java
@Service
public class BatchEmbeddingService {

    private final VectorStore vectorStore;
    private final EmbeddingClient embeddingClient;

    @Transactional
    public void processBatchEmbeddings(List<Document> documents) {
        int batchSize = 100;

        // 1. 배치 처리로 메모리 압력 감소
        Lists.partition(documents, batchSize).forEach(batch -> {
            // 2. 병렬 임베딩 생성
            List<float[]> embeddings = batch.parallelStream()
                .map(doc -> embeddingClient.embed(doc.getContent()))
                .toList();

            // 3. 배치 삽입
            vectorStore.add(batch);

            // 4. 메모리 정리
            System.gc();
        });
    }

    // 스트리밍 처리로 메모리 효율 개선
    @Async
    public CompletableFuture<Void> streamProcessEmbeddings(Stream<Document> documentStream) {
        documentStream
            .parallel()
            .forEach(doc -> {
                float[] embedding = embeddingClient.embed(doc.getContent());
                doc.setEmbedding(embedding);
                vectorStore.add(List.of(doc));
            });

        return CompletableFuture.completedFuture(null);
    }
}
```

**메모리 이슈 해결:**

1. **배치 처리**: 대량 데이터를 청크로 분할
2. **스트리밍**: Java Stream API로 지연 처리
3. **캐싱 전략**: 자주 사용되는 임베딩만 메모리 유지
4. **오프힙 저장**: Caffeine 캐시의 오프힙 저장소 활용
5. **압축**: 임베딩 벡터 양자화 (float32 → float16)

---

## 결론

### Java/Spring AI를 선택해야 하는 경우

- **엔터프라이즈 환경**: 기존 Java 생태계와 통합 필요
- **대규모 트래픽**: 일 1000만+ 요청 처리
- **복잡한 트랜잭션**: 금융, 의료 등 ACID 보장 필수
- **장기 운영**: 5년+ 장기 유지보수 계획
- **타입 안정성**: 대규모 팀 개발, 리팩토링 빈번

### Python/FastAPI를 선택해야 하는 경우

- **빠른 프로토타이핑**: MVP, PoC 개발
- **AI 중심**: 최신 AI 라이브러리 즉시 활용
- **소규모 팀**: 5명 이하 개발팀
- **단순 아키텍처**: 복잡한 비즈니스 로직 없음
- **데이터 사이언스 통합**: Jupyter, 데이터 분석 도구 연계

대규모 엔터프라이즈 AI 서비스라면 **Java/Spring AI**가 장기적으로 더 나은 선택입니다. 특히 **장기 메모리 관리, 트랜잭션 무결성, 운영 안정성** 측면에서 명확한 우위를 제공합니다.

---

## 참고 자료

- [Spring AI 공식 문서](https://docs.spring.io/spring-ai/reference/)
- [LangChain4j GitHub](https://github.com/langchain4j/langchain4j)
- [GraalVM Native Image](https://www.graalvm.org/native-image/)




================================================================================
## Java + Spring Boot + Spring AI 서비스 작동 방식 심화
================================================================================

### AI 서비스 전체 흐름도

```
사용자 요청 (HTTP POST)
    ↓
Controller Layer (@RestController)
    ├─ 요청 검증 (@Valid)
    ├─ 인증/인가 (Spring Security)
    └─ Service 호출
         ↓
Service Layer (@Service)
    ├─ 비즈니스 로직 실행
    ├─ 트랜잭션 관리 (@Transactional)
    ├─ Repository 호출 (대화 히스토리)
    └─ Spring AI Layer 호출
         ↓
Spring AI Integration Layer
    ├─ ChatClient (LLM API 호출)
    ├─ VectorStore (벡터 검색)
    └─ EmbeddingModel (임베딩 생성)
         ↓
Repository Layer (JPA/Spring Data)
    ├─ PostgreSQL (구조화된 데이터)
    ├─ Redis (캐시)
    └─ pgvector (벡터 검색)
         ↓
AI 응답 반환 및 저장
```

### 핵심 특징: 레이어 분리 아키텍처

Java + Spring AI의 가장 큰 강점은 **관심사의 분리(Separation of Concerns)**를 통한 유지보수성입니다.

#### 레이어별 역할 명확화

**Controller Layer (Presentation)**:
```java
@RestController
@RequestMapping("/api/v1/chat")
public class ChatController {

    private final ChatService chatService;

    // 생성자 주입 (Spring이 자동 주입)
    public ChatController(ChatService chatService) {
        this.chatService = chatService;
    }

    @PostMapping
    public ResponseEntity<ChatResponse> chat(
        @Valid @RequestBody ChatRequest request,  // 자동 검증
        @AuthenticationPrincipal User user         // 인증된 사용자
    ) {
        // Controller는 HTTP만 처리, 비즈니스 로직은 Service에 위임
        ChatResponse response = chatService.processChat(request, user);
        return ResponseEntity.ok(response);
    }
}
```

**Service Layer (Business Logic)**:
```java
@Service
public class ChatService {

    private final ConversationRepository conversationRepo;
    private final VectorDocumentService vectorService;
    private final ChatClient chatClient;

    // 여러 의존성을 Spring이 자동 주입
    public ChatService(
        ConversationRepository conversationRepo,
        VectorDocumentService vectorService,
        ChatClient.Builder chatClientBuilder
    ) {
        this.conversationRepo = conversationRepo;
        this.vectorService = vectorService;
        this.chatClient = chatClientBuilder.build();
    }

    @Transactional  // 트랜잭션 자동 관리
    public ChatResponse processChat(ChatRequest request, User user) {
        // 1. 권한 확인
        if (!user.canChat()) {
            throw new UnauthorizedException("채팅 권한이 없습니다");
        }

        // 2. 대화 히스토리 로드
        List<Conversation> history = conversationRepo
            .findRecentByUserId(user.getId(), PageRequest.of(0, 10));

        // 3. 벡터 검색 (RAG)
        List<VectorDocument> relevantDocs = vectorService
            .searchSimilarDocuments(request.getMessage(), 5);

        // 4. 프롬프트 구성
        String context = buildContext(relevantDocs);
        List<Message> messages = buildMessages(history, request, context);

        // 5. AI 호출
        ChatResponse aiResponse = chatClient.prompt()
            .messages(messages)
            .call()
            .chatResponse();

        // 6. 대화 저장
        Conversation conversation = new Conversation();
        conversation.setUserId(user.getId());
        conversation.setUserMessage(request.getMessage());
        conversation.setAssistantMessage(aiResponse.getResult().getOutput().getContent());
        conversationRepo.save(conversation);

        // 7. 벡터 임베딩 저장 (비동기)
        vectorService.generateAndStoreEmbeddingAsync(conversation);

        return aiResponse;
    }
}
```

**Repository Layer (Data Access)**:
```java
@Repository
public interface ConversationRepository extends JpaRepository<Conversation, Long> {

    // 메서드 이름만으로 쿼리 자동 생성
    List<Conversation> findRecentByUserId(Long userId, Pageable pageable);

    // 커스텀 쿼리
    @Query("SELECT c FROM Conversation c WHERE c.userId = :userId " +
           "AND c.createdAt >= :startDate ORDER BY c.createdAt DESC")
    List<Conversation> findRecentConversations(
        @Param("userId") Long userId,
        @Param("startDate") LocalDateTime startDate
    );
}
```

### Spring AI 핵심 컴포넌트 상세

#### 1. ChatClient - 통합 LLM 인터페이스

Spring AI는 다양한 AI 모델을 **동일한 인터페이스**로 제공합니다:

```java
@Configuration
public class AiConfig {

    // OpenAI 설정
    @Bean
    @ConditionalOnProperty(name = "ai.provider", havingValue = "openai")
    public ChatClient openAiChatClient(
        @Value("${spring.ai.openai.api-key}") String apiKey
    ) {
        return ChatClient.builder()
            .chatModel(new OpenAiChatModel(apiKey))
            .build();
    }

    // Anthropic Claude 설정
    @Bean
    @ConditionalOnProperty(name = "ai.provider", havingValue = "anthropic")
    public ChatClient anthropicChatClient(
        @Value("${spring.ai.anthropic.api-key}") String apiKey
    ) {
        return ChatClient.builder()
            .chatModel(new AnthropicChatModel(apiKey))
            .build();
    }
}
```

**사용 시 AI 모델 몰라도 됨**:
```java
@Service
public class ChatService {
    private final ChatClient chatClient;  // 어떤 AI든 상관없음

    public String chat(String message) {
        // 설정만 바꾸면 OpenAI → Claude → Ollama 자유롭게 변경
        return chatClient.prompt()
            .user(message)
            .call()
            .content();
    }
}
```

#### 2. VectorStore - RAG 구현

Spring AI는 **pgvector, Pinecone, Weaviate** 등 다양한 벡터 DB를 지원합니다:

```java
@Service
public class VectorDocumentService {

    private final EmbeddingModel embeddingModel;
    private final JdbcTemplate jdbcTemplate;

    // 임베딩 생성
    public float[] generateEmbedding(String text) {
        EmbeddingResponse response = embeddingModel
            .embedForResponse(List.of(text));
        return response.getResults().get(0).getOutput();
        // 결과: [0.123, -0.456, 0.789, ..., 0.321] (1536차원 벡터)
    }

    // pgvector를 사용한 벡터 검색
    public List<VectorDocument> searchSimilarDocuments(
        String query,
        int topK
    ) {
        // 1. 쿼리를 벡터로 변환
        float[] queryEmbedding = generateEmbedding(query);

        // 2. pgvector 네이티브 쿼리 실행
        String sql = """
            SELECT
                document_id,
                content,
                1 - (embedding <=> CAST(? AS vector)) AS similarity_score
            FROM rag_vector_document
            WHERE is_active = true
            ORDER BY embedding <=> CAST(? AS vector)
            LIMIT ?
            """;

        return jdbcTemplate.query(
            sql,
            new Object[]{queryEmbedding, queryEmbedding, topK},
            (rs, rowNum) -> {
                VectorDocument doc = new VectorDocument();
                doc.setDocumentId(rs.getLong("document_id"));
                doc.setContent(rs.getString("content"));
                doc.setSimilarityScore(rs.getDouble("similarity_score"));
                return doc;
            }
        );
    }

    // 비동기 임베딩 생성 및 저장
    @Async
    @Transactional
    public CompletableFuture<Void> generateAndStoreEmbeddingAsync(
        Conversation conversation
    ) {
        String text = conversation.getUserMessage() + " " +
                     conversation.getAssistantMessage();

        float[] embedding = generateEmbedding(text);

        VectorDocument vectorDoc = new VectorDocument();
        vectorDoc.setContent(text);
        vectorDoc.setEmbedding(embedding);
        vectorDoc.setMetadata(Map.of(
            "conversationId", conversation.getId(),
            "userId", conversation.getUserId()
        ));

        vectorDocumentRepository.save(vectorDoc);

        return CompletableFuture.completedFuture(null);
    }
}
```

#### 3. 스트리밍 응답 (SSE)

Spring AI는 **Flux**를 통해 반응형 스트리밍을 지원합니다:

```java
@RestController
@RequestMapping("/api/v1/chat")
public class ChatController {

    private final ChatClient chatClient;

    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> chatStream(@RequestParam String message) {
        // Flux<ChatResponse>로 스트리밍 응답
        return chatClient.prompt()
            .user(message)
            .stream()
            .chatResponse()
            .map(response -> response.getResult().getOutput().getContent());
    }
}
```

**프론트엔드 연동**:
```javascript
// JavaScript EventSource
const eventSource = new EventSource('/api/v1/chat/stream?message=안녕하세요');

eventSource.onmessage = (event) => {
    console.log('받은 청크:', event.data);
    // 실시간으로 화면에 출력
    document.getElementById('response').innerHTML += event.data;
};
```

### Java 빌드 & 패키징 프로세스

#### 의존성 관리 (Gradle)

```groovy
// build.gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.5.7'
    id 'io.spring.dependency-management' version '1.1.4'
}

dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    // Spring AI (BOM으로 버전 관리)
    implementation platform('org.springframework.ai:spring-ai-bom:1.0.3')
    implementation 'org.springframework.ai:spring-ai-openai-spring-boot-starter'
    implementation 'org.springframework.ai:spring-ai-anthropic-spring-boot-starter'
    implementation 'org.springframework.ai:spring-ai-pgvector-store-spring-boot-starter'

    // pgvector
    implementation 'com.pgvector:pgvector:0.1.6'

    // PostgreSQL
    runtimeOnly 'org.postgresql:postgresql'

    // Redis
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}

// Fat JAR 빌드 설정
tasks.named('bootJar') {
    archiveFileName = 'app.jar'
    // 모든 의존성을 JAR에 포함
    enabled = true
}
```

#### 빌드 프로세스

```bash
# 1. Gradle 빌드 실행
./gradlew clean bootJar

# 내부 동작:
# - 소스 코드 컴파일 (*.java → *.class)
# - 의존성 다운로드 (Maven Central에서)
# - 모든 라이브러리를 하나의 JAR로 패킹
# - 결과: build/libs/app.jar (약 80MB)

# 2. JAR 파일 구조 확인
jar -tf build/libs/app.jar

# 출력:
META-INF/
├── MANIFEST.MF
├── maven/
└── spring.factories
BOOT-INF/
├── classes/  (컴파일된 애플리케이션 코드)
│   ├── com/xsk/xyroplug/aicc/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── repository/
│   │   └── domain/
│   └── application.yml
└── lib/  (모든 의존성 JAR들)
    ├── spring-boot-3.5.7.jar
    ├── spring-ai-openai-1.0.3.jar
    ├── spring-ai-anthropic-1.0.3.jar
    ├── postgresql-42.7.1.jar
    ├── pgvector-0.1.6.jar
    └── ... (약 200개 라이브러리)
org/springframework/boot/loader/  (Spring Boot Loader)
```

**Fat JAR의 장점**:
- ✅ 단일 파일로 배포 가능
- ✅ 의존성 버전 충돌 없음
- ✅ 실행 환경에 Java만 필요

#### Docker 컨테이너화

```dockerfile
# Dockerfile
FROM ibm-semeru-runtimes:open-17-jdk

# JAR 파일 복사
ADD build/libs/app.jar /build/app.jar

# 실행
ENTRYPOINT ["java", "-jar", "/build/app.jar"]
```

**Python과 비교**:

| 항목 | Python (FastAPI) | Java (Spring Boot) |
|------|------------------|-------------------|
| 의존성 선언 | requirements.txt | build.gradle |
| 패키지 매니저 | pip | Gradle/Maven |
| 패키징 결과 | 소스 코드 + site-packages/ | 단일 Fat JAR |
| 패키징 시점 | Docker 빌드 시 | Gradle 빌드 시 |
| 최종 산출물 | 디렉토리 구조 | app.jar (단일 파일) |
| 실행 방식 | python main.py | java -jar app.jar |
| 베이스 이미지 | python:3.11-slim (150MB) | openjdk:17 (300MB) |
| 최종 이미지 | ~500MB | ~485MB |

#### CI/CD 파이프라인 (GitLab)

```yaml
# .gitlab-ci.yml
stages:
  - build
  - package
  - deploy

# Stage 1: Gradle 빌드
gradle-build:
  stage: build
  image: gradle:8.5-jdk17
  script:
    - ./gradlew clean bootJar
    - cp build/libs/*.jar .build-cache/
  artifacts:
    paths:
      - .build-cache/*.jar
    expire_in: 1 hour
  only:
    - develop
    - main

# Stage 2: Docker 이미지 빌드
docker-build:
  stage: package
  image: docker:latest
  script:
    - cp .build-cache/*.jar build/libs/
    - docker build -t xsko/xyroplug-aicc:latest .
    - docker push xsko/xyroplug-aicc:latest
  dependencies:
    - gradle-build
  only:
    - develop

# Stage 3: Kubernetes 배포
k8s-deploy:
  stage: deploy
  image: mcr.microsoft.com/azure-cli
  script:
    - az login --service-principal ...
    - az aks get-credentials --name xsk
    - kubectl apply -f deploy/dev/k8s-deployment.yaml
    - kubectl rollout restart deployment/xyroplug-aicc -n xyroplug-dev
  only:
    - develop
```

**전체 배포 흐름**:
```
Git Push (develop 브랜치)
    ↓
GitLab Runner 실행
    ↓
[Stage 1] Gradle Build
├─ ./gradlew bootJar
├─ 소스 컴파일 (1분)
├─ 의존성 해결
└─ Fat JAR 생성 (app.jar)
    ↓
[Stage 2] Docker Build
├─ Dockerfile 실행
├─ JAR 복사
└─ 이미지 푸시 (Docker Hub)
    ↓
[Stage 3] Kubernetes Deploy
├─ AKS 인증
├─ kubectl apply
└─ Rolling Update (Zero-downtime)
    ↓
배포 완료 (3~5분)
```

### Spring Dependency Injection (DI) 심화

Spring의 **가장 강력한 기능** 중 하나는 의존성 자동 주입입니다:

#### 자동 빈(Bean) 생성 및 주입

```java
// 1. ChatClient 빈 등록
@Configuration
public class AiConfig {

    @Bean
    public ChatClient chatClient(
        @Value("${spring.ai.openai.api-key}") String apiKey
    ) {
        return ChatClient.builder()
            .chatModel(new OpenAiChatModel(apiKey))
            .build();
    }

    @Bean
    public EmbeddingModel embeddingModel(
        @Value("${spring.ai.openai.api-key}") String apiKey
    ) {
        return new OpenAiEmbeddingModel(apiKey);
    }
}

// 2. Service에서 자동 주입 받기
@Service
public class ChatService {
    private final ChatClient chatClient;
    private final EmbeddingModel embeddingModel;
    private final ConversationRepository conversationRepo;

    // Spring이 자동으로 3개 빈을 찾아서 주입
    public ChatService(
        ChatClient chatClient,
        EmbeddingModel embeddingModel,
        ConversationRepository conversationRepo
    ) {
        this.chatClient = chatClient;
        this.embeddingModel = embeddingModel;
        this.conversationRepo = conversationRepo;
    }

    // 이제 모든 의존성이 준비된 상태로 메서드 사용 가능
}
```

**Python과 비교**:
```python
# Python - 직접 인스턴스 생성
from openai import OpenAI
from chromadb import Client

class ChatService:
    def __init__(self):
        # 직접 생성해야 함
        self.openai_client = OpenAI(api_key="sk-...")
        self.chroma_client = Client()
        self.db = create_db_connection()

# Java Spring - 자동 주입
@Service
public class ChatService {
    // Spring이 알아서 생성하고 주입
    public ChatService(
        ChatClient chatClient,
        VectorStore vectorStore,
        DataSource dataSource
    ) { ... }
}
```

#### 빈 스코프 관리

```java
// 싱글톤 (기본값) - 앱 전체에서 하나만 생성
@Service
public class ChatService { ... }

// 요청마다 새로 생성
@Service
@Scope("request")
public class RequestScopedService { ... }

// 프로토타입 - 요청마다 새 인스턴스
@Service
@Scope("prototype")
public class PrototypeService { ... }
```

### 트랜잭션 관리

Spring의 **@Transactional**은 데이터베이스 트랜잭션을 자동으로 관리합니다:

```java
@Service
public class ChatService {

    @Transactional  // 이 메서드는 트랜잭션 안에서 실행됨
    public ChatResponse processChat(ChatRequest request) {
        // 1. 대화 저장
        Conversation conversation = conversationRepo.save(
            new Conversation(request.getMessage())
        );

        // 2. AI 호출
        ChatResponse aiResponse = chatClient.prompt()
            .user(request.getMessage())
            .call()
            .chatResponse();

        // 3. 응답 업데이트
        conversation.setAssistantMessage(
            aiResponse.getResult().getOutput().getContent()
        );
        conversationRepo.save(conversation);

        // 4. 통계 업데이트
        statsRepository.incrementChatCount(request.getUserId());

        // ✅ 모든 작업이 성공하면 커밋
        // ❌ 하나라도 실패하면 모두 롤백
        return aiResponse;
    }
}
```

**트랜잭션 없이 문제 상황**:
```
1. 대화 저장 ✅
2. AI 호출 ✅
3. 응답 업데이트 ✅
4. 통계 업데이트 ❌ (에러!)

→ 결과: 대화는 저장되었지만 통계는 업데이트 안 됨 (데이터 불일치!)
```

**@Transactional로 해결**:
```
1. 대화 저장
2. AI 호출
3. 응답 업데이트
4. 통계 업데이트 ❌ (에러!)

→ @Transactional이 감지 → 모든 작업 롤백 → 데이터 일관성 유지 ✅
```

### 보안 (Spring Security)

엔터프라이즈 환경에서 필수적인 **인증/인가**를 쉽게 구현:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()  // 공개 API
                .requestMatchers("/api/chat/**").hasRole("USER")  // 사용자 전용
                .requestMatchers("/api/admin/**").hasRole("ADMIN")  // 관리자 전용
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt())  // JWT 인증
            .csrf(csrf -> csrf.disable());  // REST API용

        return http.build();
    }
}

// Controller에서 인증된 사용자 정보 자동 주입
@RestController
@RequestMapping("/api/chat")
public class ChatController {

    @PostMapping
    public ChatResponse chat(
        @RequestBody ChatRequest request,
        @AuthenticationPrincipal User user  // Spring Security가 자동 주입
    ) {
        // user 객체에 인증된 사용자 정보 포함
        return chatService.processChat(request, user);
    }
}
```

### 성능 최적화 전략

#### 1. JPA 쿼리 최적화

```java
@Repository
public interface ConversationRepository extends JpaRepository<Conversation, Long> {

    // N+1 문제 방지 (JOIN FETCH)
    @Query("SELECT c FROM Conversation c " +
           "JOIN FETCH c.user " +
           "WHERE c.sessionId = :sessionId")
    List<Conversation> findBySessionIdWithUser(@Param("sessionId") String sessionId);

    // 페이징 & 정렬
    Page<Conversation> findByUserId(
        Long userId,
        Pageable pageable
    );

    // 프로젝션 (필요한 컬럼만 조회)
    @Query("SELECT new com.example.dto.ConversationSummary(c.id, c.userMessage, c.createdAt) " +
           "FROM Conversation c WHERE c.userId = :userId")
    List<ConversationSummary> findSummariesByUserId(@Param("userId") Long userId);
}
```

#### 2. 캐싱 전략

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10)))  // 10분 TTL
            .build();
    }
}

@Service
public class ChatService {

    // 메서드 결과를 캐싱
    @Cacheable(value = "chatHistory", key = "#sessionId")
    public List<Conversation> getChatHistory(String sessionId) {
        return conversationRepo.findBySessionId(sessionId);
    }

    // 캐시 무효화
    @CacheEvict(value = "chatHistory", key = "#sessionId")
    public void clearChatHistory(String sessionId) {
        conversationRepo.deleteBySessionId(sessionId);
    }
}
```

#### 3. 비동기 처리

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class VectorService {

    // 비동기 메서드
    @Async
    public CompletableFuture<List<VectorDocument>> searchAsync(String query) {
        List<VectorDocument> results = performVectorSearch(query);
        return CompletableFuture.completedFuture(results);
    }
}

// 사용
CompletableFuture<List<VectorDocument>> future1 = vectorService.searchAsync("query1");
CompletableFuture<List<VectorDocument>> future2 = vectorService.searchAsync("query2");

// 두 작업 모두 완료될 때까지 대기
CompletableFuture.allOf(future1, future2).join();
```

### 장점 요약

#### ✅ 장점

1. **타입 안정성**
   - 컴파일 타임 오류 검출
   - IDE 자동완성 강력
   - 리팩토링 안전

2. **엔터프라이즈 통합**
   - 레거시 Java 시스템과 쉽게 통합
   - Spring 생태계 (Security, Batch, Cloud)
   - 복잡한 트랜잭션 관리

3. **성능**
   - JVM 최적화 (JIT 컴파일)
   - 멀티스레딩 지원
   - CPU 집약적 작업 유리

4. **장기 유지보수**
   - 명확한 레이어 구조
   - 의존성 주입으로 테스트 용이
   - 대규모 팀 협업 유리


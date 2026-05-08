# Rust AI Core Stack Guide

Python 백엔드와 AI 생태계를 활용하면서, 병목이 되는 핵심 엔진은 Rust로 만드는 전략을 정리한 가이드입니다.

이 문서는 Rust를 처음부터 AI 서비스 전체의 주 언어로 쓰자는 주장이 아닙니다. 오히려 현실적인 결론은 반대입니다.

> Python으로 빠르게 만들고, 실제 병목이 드러난 핵심 코어만 Rust로 옮긴다.

---

## 목차

1. [핵심 결론](#핵심-결론)
2. [한 문장으로 이해하기](#한-문장으로-이해하기)
3. [왜 Python + Rust 조합인가](#왜-python--rust-조합인가)
4. [Python이 강한 영역](#python이-강한-영역)
5. [Python이 약해지는 지점](#python이-약해지는-지점)
6. [Rust가 강한 영역](#rust가-강한-영역)
7. [Rust로 만들면 좋은 AI 코어 모듈](#rust로-만들면-좋은-ai-코어-모듈)
8. [최신 AI 논문 구현에서의 역할 분리](#최신-ai-논문-구현에서의-역할-분리)
9. [추천 아키텍처](#추천-아키텍처)
10. [개발 단계별 전략](#개발-단계별-전략)
11. [Python과 Rust 연결 방법](#python과-rust-연결-방법)
12. [Rust AI Core 모듈 설계 패턴](#rust-ai-core-모듈-설계-패턴)
13. [대표 Rust 기반 AI/데이터 프로젝트](#대표-rust-기반-ai데이터-프로젝트)
14. [언제 Rust를 쓰면 안 되는가](#언제-rust를-쓰면-안-되는가)
15. [학습 로드맵](#학습-로드맵)
16. [최종 정리](#최종-정리)
17. [참고 자료](#참고-자료)

---

## 핵심 결론

가장 현실적이고 강력한 구조는 다음과 같습니다.

```text
Python = AI orchestration
Rust = high-performance core engine
CUDA/C++ = GPU inference kernel
```

즉, 전체 AI 서비스를 Rust로 다시 만드는 것이 목표가 아닙니다.

실무에서 가장 좋은 전략은:

1. Python으로 AI 파이프라인을 빠르게 만든다.
2. 실제 트래픽과 데이터로 병목을 측정한다.
3. 병목이 되는 부분만 Rust 코어 모듈로 분리한다.
4. PyO3, maturin, FFI, gRPC 등으로 Python 서비스에 연결한다.
5. GPU 커널이 필요한 영역은 CUDA/C++ 또는 TensorRT-LLM 같은 전문 런타임에 맡긴다.

이 방식은 개발 속도와 운영 성능을 동시에 잡기 위한 하이브리드 전략입니다.

---

## 한 문장으로 이해하기

AI 서비스에서 Python은 두뇌를 빠르게 조립하는 언어이고, Rust는 그 두뇌가 반복적으로 쓰는 고성능 근육을 만드는 언어입니다.

조금 더 구체적으로 말하면:

| 역할 | 추천 기술 | 이유 |
|---|---|---|
| 논문 이해와 실험 | Python | PyTorch, Jupyter, Transformers 생태계가 압도적 |
| 서비스 API | Python/FastAPI | 빠른 개발, 풍부한 라이브러리 |
| 병목 코어 | Rust | 메모리 안정성, 병렬 처리, 낮은 지연 시간 |
| GPU 추론 | CUDA/C++, TensorRT-LLM, vLLM | GPU 커널과 추론 스케줄링 전문 영역 |
| 대규모 운영 | Kubernetes, Redis, Kafka/NATS, OpenTelemetry | 배포, 캐싱, 메시징, 관측성 |

---

## 왜 Python + Rust 조합인가

AI 서비스는 크게 두 층으로 나뉩니다.

첫 번째는 빠르게 바뀌는 실험과 오케스트레이션 영역입니다.

예를 들면:

- 어떤 모델을 쓸 것인가
- 프롬프트를 어떻게 구성할 것인가
- RAG 검색 결과를 어떻게 섞을 것인가
- 평가 지표를 어떻게 볼 것인가
- Agent workflow를 어떻게 조립할 것인가

이 영역은 Python이 강합니다.

두 번째는 반복적으로 대량 처리되는 엔진 영역입니다.

예를 들면:

- 토큰화
- 문서 파싱
- chunking
- 벡터 변환
- ANN 검색
- JSON 직렬화/역직렬화
- request routing
- stream multiplexing
- rate limiting
- batching
- scheduler
- cache 관리

이 영역은 Rust가 강합니다.

AI 서비스는 처음에는 연구와 제품 실험 문제처럼 보이지만, 운영 규모가 커질수록 점점 시스템 엔지니어링 문제가 됩니다. 그래서 Python만으로 충분한 구간과 Rust를 넣어야 하는 구간을 구분하는 능력이 중요합니다.

---

## Python이 강한 영역

Python은 AI 생태계의 중심 언어입니다.

대표 생태계:

- PyTorch
- Transformers
- LangChain
- LlamaIndex
- vLLM
- Ray
- NumPy
- Pandas
- scikit-learn
- MLflow
- Weights & Biases

Python이 강한 작업:

- 논문 재현
- 모델 실험
- training loop 작성
- fine-tuning
- prompt engineering
- agent workflow 구성
- RAG pipeline 구성
- evaluation
- 빠른 API 개발
- 데이터 과학자와 협업

Python의 가장 큰 장점은 속도입니다. 여기서 말하는 속도는 프로그램 실행 속도가 아니라 개발 속도입니다.

AI 제품의 초기 단계에서는 실행 성능보다 더 중요한 것이 있습니다.

- 아이디어가 맞는가
- 사용자가 원하는가
- 모델 품질이 충분한가
- 검색 품질이 충분한가
- latency 목표가 어느 정도인가
- 비용 구조가 감당 가능한가

이 질문에 답하기 전부터 Rust로 모든 것을 만들면, 아직 바뀔 가능성이 큰 것을 너무 단단하게 만들어버리는 문제가 생깁니다.

---

## Python이 약해지는 지점

Python이 약하다는 말은 Python이 나쁜 언어라는 뜻이 아닙니다. Python은 실험과 오케스트레이션에 매우 좋습니다. 다만 AI 서비스가 커지면 반복 작업이 늘어나고, 이 반복 작업에서는 Python의 비용이 커집니다.

### 1. GIL

CPython에는 GIL(Global Interpreter Lock)이 있습니다.

간단히 말하면 Python 코드가 CPU를 많이 쓰는 작업을 여러 스레드로 동시에 돌릴 때 제약이 생길 수 있습니다.

문제가 되는 작업:

- tokenizer
- parser
- text normalization
- vector transform
- embedding post-processing
- JSON parsing
- compression/decompression
- ranking feature 계산
- rule engine

I/O 중심 작업은 `asyncio`로 상당히 잘 처리할 수 있습니다. 하지만 CPU를 많이 쓰는 작업이 섞이면 event loop가 막히거나 worker가 버벅일 수 있습니다.

### 2. 메모리 관리

AI 서비스는 일반 웹 서비스보다 큰 객체를 많이 다룹니다.

예:

- 긴 문맥
- 대화 기록
- 대량 문서 chunk
- embedding 배열
- multimodal metadata
- stream buffer
- tenant별 cache

Python 객체가 많아지면:

- 메모리 단편화
- RSS 증가
- GC 부담
- copy 비용
- serialization 비용

이 누적될 수 있습니다.

특히 다음 환경에서 심해집니다.

- long context
- websocket
- SSE streaming
- multi-tenant
- batch inference
- 대량 문서 ingestion

### 3. 안정적인 latency

AI 서비스에서는 평균 latency보다 tail latency가 중요합니다.

평균 200ms가 나와도 p99가 5초면 사용자는 서비스가 느리다고 느낍니다.

Python에서 tail latency가 커지는 원인:

- GIL 경합
- event loop blocking
- worker hang
- 큰 객체 복사
- JSON serialization overhead
- GC pause
- 외부 라이브러리의 blocking call

Rust는 메모리와 스레드를 더 예측 가능하게 제어할 수 있어 이런 영역에서 장점이 큽니다.

---

## Rust가 강한 영역

Rust의 핵심 장점은 다음과 같습니다.

- 메모리 안정성
- 높은 성능
- 예측 가능한 지연 시간
- 강력한 타입 시스템
- zero-cost abstraction
- 안전한 동시성
- C/C++와 가까운 제어력
- Python 바인딩 가능
- async runtime 생태계

Rust는 C/C++처럼 빠른 저수준 제어를 제공하면서도, 소유권(ownership)과 빌림(borrowing) 시스템으로 많은 메모리 오류를 컴파일 단계에서 막습니다.

Rust가 특히 빛나는 상황:

- 문자열을 대량 처리한다.
- 메모리를 직접 아끼고 싶다.
- 많은 요청을 안정적으로 처리해야 한다.
- 병렬 CPU 작업이 많다.
- long-running process가 안정적이어야 한다.
- Python에서 특정 함수가 너무 자주 호출된다.
- 큰 배열, byte buffer, vector를 복사 없이 넘기고 싶다.

---

## Rust로 만들면 좋은 AI 코어 모듈

Rust는 전체 AI 서비스보다 핵심 모듈에 넣을 때 ROI가 가장 좋습니다.

### 1. Tokenizer

Tokenizer는 텍스트를 모델이 이해할 수 있는 token id로 바꾸는 모듈입니다.

예:

```text
"오늘 날씨 어때?"
    ↓
[42123, 1987, 992, 30]
```

Tokenizer는 단순해 보이지만 실제로는 매우 까다롭습니다.

- Unicode 처리
- normalization
- byte-level BPE
- sentencepiece 계열 처리
- special token 처리
- offset alignment
- batch tokenization
- truncation/padding

Hugging Face Tokenizers가 Rust 기반인 이유도 여기에 있습니다. 공식 문서에서도 fast tokenizer는 Rust 기반 구현을 사용하며, batch tokenization에서 Python 구현보다 훨씬 빠른 점을 강조합니다.

Rust tokenizer가 좋은 이유:

- 문자열 처리 성능
- SIMD 최적화 가능
- 병렬 batch 처리
- offset tracking
- memory allocation 제어

추천 사용처:

- custom tokenizer
- prompt length 계산
- 대량 문서 token counting
- chunk boundary 계산
- long context trimming
- multi-model tokenizer router

### 2. Document Parser / Ingestion Engine

RAG 시스템에서 가장 과소평가되는 영역이 문서 ingestion입니다.

모델 품질이 좋아도 문서 처리 품질이 낮으면 RAG 답변은 망가집니다.

Rust로 만들기 좋은 문서 처리:

- PDF text extraction pipeline
- HTML cleanup
- Markdown parser
- code parser
- AST parser
- log parser
- table extraction preprocessor
- chunking engine
- metadata normalizer

왜 Rust가 좋은가:

- 대량 파일 처리 성능
- memory pressure 감소
- streaming parser 작성에 유리
- binary format 처리에 강함
- 실패 격리가 쉬움

예시 구조:

```text
Python ingestion workflow
    ↓
Rust document parser
    ↓
Rust chunking engine
    ↓
Python embedding call
    ↓
Vector DB
```

### 3. Vector Transform / Retrieval Core

RAG는 단순히 embedding을 저장하고 검색하는 것이 아닙니다.

실제 서비스에서는 검색 전후에 많은 처리가 들어갑니다.

예:

- vector normalization
- dense/sparse hybrid score merge
- metadata filtering
- reranker input formatting
- top-k merge
- threshold filtering
- deduplication
- diversity selection
- graph traversal
- HNSW tuning

Rust가 좋은 이유:

- 벡터 연산이 CPU와 메모리를 많이 사용함
- 대량 배열 처리에서 copy 비용이 중요함
- p99 latency를 낮추기 좋음
- 자료구조를 세밀하게 제어 가능

대표 사례:

- Qdrant: Rust 기반 vector search engine
- LanceDB: Rust, Python, TypeScript SDK를 제공하는 AI 데이터/검색 계층

### 4. Streaming Engine

LLM 서비스에서 streaming은 단순히 글자를 조금씩 보내는 기능이 아닙니다.

실제로는 다음을 처리해야 합니다.

- SSE
- websocket
- token stream multiplexing
- client disconnect
- backpressure
- heartbeat
- timeout
- partial output
- retry
- cancellation
- audit logging

Rust의 Tokio는 이런 네트워크/비동기 작업에 강합니다.

추천 사용처:

- LLM token streaming proxy
- multi-client stream fanout
- agent event stream
- realtime chat gateway
- streaming log collector

예시:

```text
Browser
  ↓ SSE/WebSocket
Rust Streaming Gateway
  ↓
Python AI Orchestrator
  ↓
vLLM / OpenAI / Claude / Local model
```

### 5. AI Gateway

AI Gateway는 AI 서비스 앞단에서 요청을 통제하는 계층입니다.

역할:

- auth
- rate limiting
- quota
- model routing
- prompt policy
- caching
- request batching
- retry
- circuit breaker
- tenant isolation
- observability

Rust가 적합한 이유:

- 낮은 latency
- 안정적인 concurrent connection 처리
- 메모리 효율
- 네트워크 서버 작성에 강함
- Tokio, Axum, Tower 생태계 활용 가능

AI Gateway는 Python으로도 만들 수 있지만, 트래픽이 많고 정책이 복잡해질수록 Rust/Go 계열이 유리해집니다.

### 6. Scheduler / Batching Engine

LLM 추론에서는 scheduler가 매우 중요합니다.

GPU가 비싼 자원이기 때문입니다.

좋은 scheduler는 다음을 결정합니다.

- 어떤 요청을 먼저 처리할지
- batch를 어떻게 묶을지
- stream 요청과 non-stream 요청을 어떻게 섞을지
- 긴 요청과 짧은 요청을 어떻게 공정하게 처리할지
- KV cache를 어떻게 재사용할지
- timeout된 요청을 어떻게 취소할지

Rust가 좋은 이유:

- 동시성 제어
- 자료구조 최적화
- predictable latency
- memory layout 제어

다만 GPU kernel 자체는 Rust보다 CUDA/C++ 또는 기존 inference framework를 쓰는 것이 현실적입니다.

### 7. Rule Engine / Policy Engine

AI 서비스에는 보안, 비용, 정책 판단이 들어갑니다.

예:

- 이 사용자는 어떤 모델을 쓸 수 있는가
- 하루 토큰 한도는 얼마인가
- 이 요청은 민감한 데이터인가
- 어떤 route로 보낼 것인가
- cache hit 조건은 무엇인가

Rust로 policy engine을 만들면:

- 빠른 판단
- 안전한 병렬 처리
- config 기반 동작
- Python 서비스와 분리된 안정성

을 얻을 수 있습니다.

---

## 최신 AI 논문 구현에서의 역할 분리

중요한 기준은 이것입니다.

```text
논문 재현은 Python이 강하다.
논문 엔진화/서비스화는 Rust가 강하다.
```

### Python이 좋은 단계

논문을 처음 읽고 실험할 때:

- PyTorch로 모델 구조 작성
- notebook에서 tensor shape 확인
- 작은 데이터로 ablation
- loss curve 확인
- prompt 실험
- baseline 비교
- evaluation script 작성

이 단계에서는 Python이 압도적으로 좋습니다.

### Rust가 좋은 단계

논문 아이디어를 운영 가능한 엔진으로 만들 때:

- inference scheduler
- KV cache manager
- batching algorithm
- vector index
- stream runtime
- parser engine
- memory optimizer
- distributed runtime
- gateway policy

이 단계에서는 Rust가 강력합니다.

### 왜 최신 AI는 시스템 최적화 경쟁인가

초기 딥러닝 경쟁은 모델 구조 혁신의 비중이 컸습니다.

하지만 LLM 서비스 경쟁에서는 다음 요소가 매우 중요해졌습니다.

- inference efficiency
- KV cache reuse
- paged attention
- speculative decoding
- continuous batching
- cache locality
- distributed execution
- quantization
- memory bandwidth
- GPU utilization

즉 AI는 이제 단순히 모델을 잘 쓰는 문제가 아니라, 분산 시스템과 런타임 엔지니어링 문제가 되었습니다.

Rust는 이 시스템 레벨 경쟁에서 강한 도구입니다.

---

## 추천 아키텍처

### 기본형

```text
Client
  ↓
FastAPI Backend
  ↓
Python AI Orchestration
  ↓
Rust Core Modules
  ↓
External AI Serving / Vector DB / Storage
```

### 실무형

```text
Client
  ↓
Rust AI Gateway
  ├─ auth
  ├─ rate limit
  ├─ routing
  ├─ caching
  └─ streaming
       ↓
Python FastAPI Orchestrator
  ├─ prompt composition
  ├─ RAG workflow
  ├─ agent execution
  └─ evaluation hooks
       ↓
Rust Core Engines
  ├─ tokenizer
  ├─ parser
  ├─ chunker
  ├─ vector transform
  ├─ reranker preprocessor
  └─ stream multiplexer
       ↓
AI Serving
  ├─ vLLM
  ├─ TensorRT-LLM
  ├─ OpenAI API
  ├─ Anthropic API
  └─ local model runtime
```

### 대규모형

```text
Client
  ↓
Load Balancer
  ↓
Rust AI Gateway Cluster
  ↓
Message Bus
  ├─ Kafka
  └─ NATS
       ↓
Python Orchestrator Workers
       ↓
Rust Core Engine Services
       ↓
GPU Inference Cluster
       ↓
Vector DB / Object Storage / Observability
```

---

## 개발 단계별 전략

### Phase 1: Python prototype

처음에는 전부 Python으로 만듭니다.

목표:

- 아이디어 검증
- 모델 품질 확인
- RAG 품질 확인
- workflow 검증
- 사용자 반응 확인
- 비용 구조 확인

이 단계의 산출물:

```text
prototype.py
app/main.py
app/services/chat_service.py
app/rag/pipeline.py
app/evaluation/eval.py
```

이 단계에서는 Rust를 참는 것이 좋습니다. 아직 무엇이 병목인지 모르기 때문입니다.

### Phase 2: Profiling

사용자가 생기거나 실제 데이터가 들어오면 병목을 측정합니다.

볼 것:

- CPU hotspot
- memory hotspot
- event loop blocking
- serialization overhead
- tokenization time
- parsing time
- vector search time
- p95/p99 latency
- worker restart 빈도
- RSS 증가 추세

도구 예:

- py-spy
- scalene
- cProfile
- memray
- OpenTelemetry
- Prometheus
- flamegraph

중요한 원칙:

> 느릴 것 같은 곳을 Rust로 바꾸지 말고, 실제로 느린 곳을 Rust로 바꾼다.

### Phase 3: Rust core extraction

병목이 확인된 부분을 Rust crate로 분리합니다.

예:

```text
rust_ai_core_tokenizer
rust_ai_core_parser
rust_ai_core_chunker
rust_ai_core_vector
rust_ai_core_gateway
rust_ai_core_stream
```

이때 Python과 Rust 사이의 경계를 잘 잡아야 합니다.

나쁜 경계:

```text
Python이 Rust 함수를 너무 자주 작게 호출
```

좋은 경계:

```text
큰 batch를 Rust로 넘기고, Rust가 내부에서 한 번에 처리
```

### Phase 4: Python binding

Python에서 Rust 모듈을 import해서 씁니다.

대표 도구:

- PyO3
- maturin

예시:

```python
from rust_ai_core_tokenizer import count_tokens_batch

texts = ["hello", "world", "long document ..."]
counts = count_tokens_batch(texts)
```

### Phase 5: Service separation

모듈이 더 커지고 독립적인 scaling이 필요해지면 Rust를 별도 서비스로 분리할 수 있습니다.

연결 방식:

- HTTP
- gRPC
- NATS
- Kafka
- Unix domain socket

선택 기준:

| 연결 방식 | 적합한 경우 |
|---|---|
| PyO3 | 같은 프로세스에서 빠르게 호출 |
| gRPC | 언어 독립 서비스화 |
| HTTP | 단순한 내부 API |
| NATS/Kafka | 비동기 pipeline |
| FFI | 극단적인 성능 최적화 |

---

## Python과 Rust 연결 방법

### 1. PyO3

PyO3는 Rust로 Python native extension module을 만들 수 있게 해주는 대표 도구입니다.

쉽게 말하면 Rust 코드를 Python 패키지처럼 import할 수 있게 해줍니다.

```text
Rust function
    ↓ PyO3
Python import
```

예시:

```rust
use pyo3::prelude::*;

#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
    Ok((a + b).to_string())
}

#[pymodule]
fn rust_ai_core(_py: Python<'_>, m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    Ok(())
}
```

Python에서는:

```python
import rust_ai_core

print(rust_ai_core.sum_as_string(5, 20))
```

PyO3가 좋은 경우:

- Python 프로젝트 안에서 Rust 함수를 직접 쓰고 싶다.
- packaging을 Python wheel로 배포하고 싶다.
- tokenizer, parser, vector transform처럼 함수형 코어가 있다.
- Python API를 유지하면서 내부만 Rust로 바꾸고 싶다.

주의할 점:

- Python 객체를 너무 자주 Rust로 넘기면 overhead가 생긴다.
- 작은 함수 1억 번 호출보다 큰 batch 1번 호출이 낫다.
- GIL을 언제 잡고 언제 풀지 설계해야 한다.

### 2. maturin

maturin은 Rust 기반 Python package를 빌드하고 배포하기 쉽게 해주는 도구입니다.

일반적인 흐름:

```bash
python -m venv .venv
.venv\Scripts\activate
pip install maturin
maturin init --bindings pyo3
maturin develop
python -c "import rust_ai_core"
```

개발 중에는:

```bash
maturin develop
```

배포 wheel을 만들 때는:

```bash
maturin build --release
```

### 3. gRPC

Rust 모듈이 별도 서비스로 커졌다면 gRPC가 좋습니다.

장점:

- 언어 독립
- schema 기반
- 내부 서비스 통신에 적합
- streaming RPC 가능

단점:

- 배포 단위가 늘어남
- 운영 복잡도 증가
- 로컬 함수 호출보다 overhead 있음

적합한 예:

- Rust vector service
- Rust parser service
- Rust streaming gateway
- Rust policy engine

### 4. HTTP

가장 단순한 방식입니다.

장점:

- 디버깅 쉬움
- curl로 테스트 가능
- 운영자가 이해하기 쉬움

단점:

- schema 안정성이 gRPC보다 약함
- high-throughput 내부 통신에는 상대적으로 불리할 수 있음

### 5. FFI

FFI는 C ABI를 통해 Rust와 다른 언어를 연결하는 방식입니다.

좋은 경우:

- Python뿐 아니라 C/C++/Java 등에서도 써야 한다.
- 매우 낮은 overhead가 필요하다.
- binary SDK 형태로 배포하고 싶다.

주의:

- 메모리 소유권 경계를 명확히 해야 한다.
- 안전하지 않은 코드가 들어갈 수 있다.
- API 안정성을 신중하게 설계해야 한다.

---

## Rust AI Core 모듈 설계 패턴

### 패턴 1: Batch-first API

Python과 Rust 경계에서는 batch 단위가 좋습니다.

나쁜 예:

```python
for text in texts:
    count = rust_count_tokens(text)
```

좋은 예:

```python
counts = rust_count_tokens_batch(texts)
```

이유:

- Python-Rust 호출 overhead 감소
- Rust 내부 병렬 처리 가능
- memory allocation 최적화 가능

### 패턴 2: Pure function core

가능하면 Rust core는 상태가 적은 순수 함수처럼 설계합니다.

좋은 예:

```text
input documents
    ↓
parse_and_chunk()
    ↓
chunks + metadata
```

장점:

- 테스트 쉬움
- Python과 연결 쉬움
- 병렬화 쉬움
- 장애 원인 추적 쉬움

### 패턴 3: Config-driven engine

AI 서비스는 설정이 자주 바뀝니다.

따라서 Rust에 값을 하드코딩하지 말고 config로 받는 것이 좋습니다.

예:

```toml
[chunking]
max_tokens = 800
overlap_tokens = 80
split_by_heading = true

[retrieval]
top_k = 20
deduplicate = true
min_score = 0.72
```

### 패턴 4: Zero-copy를 목표로 하되 집착하지 않기

Rust를 쓰면 zero-copy를 떠올리기 쉽습니다.

하지만 처음부터 모든 것을 zero-copy로 만들면 설계가 너무 어려워질 수 있습니다.

추천:

1. 먼저 명확하고 안전한 API를 만든다.
2. benchmark로 copy 비용을 확인한다.
3. 정말 필요한 곳만 zero-copy로 최적화한다.

### 패턴 5: Error boundary 명확화

Rust error를 Python exception으로 어떻게 바꿀지 정해야 합니다.

예:

| Rust error | Python exception |
|---|---|
| InvalidConfig | ValueError |
| ParseError | RuntimeError 또는 custom ParseError |
| Timeout | TimeoutError |
| InternalError | RuntimeError |

사용자에게 보일 오류와 내부 로그용 오류를 분리해야 합니다.

### 패턴 6: Observability 내장

Rust core도 관측 가능해야 합니다.

필요한 것:

- 처리 시간
- batch size
- input byte size
- output count
- error count
- memory allocation trend
- p95/p99 latency

추천:

- tracing
- OpenTelemetry
- Prometheus metrics

---

## 대표 Rust 기반 AI/데이터 프로젝트

### Hugging Face Tokenizers

Rust 기반 tokenizer의 대표 사례입니다.

의미:

- Python AI 생태계 안에서도 성능 핵심은 Rust로 만들 수 있다.
- Python 사용자는 Python API를 쓰지만 내부 핵심 엔진은 Rust일 수 있다.

이 패턴은 이 문서의 핵심 전략과 정확히 맞습니다.

```text
Python Transformers API
    ↓
Rust Tokenizers core
```

### Qdrant

Qdrant는 Rust 기반 vector search engine입니다.

AI 서비스에서 의미 검색, RAG, recommendation, hybrid search 같은 영역에 사용됩니다.

Rust가 적합한 이유:

- vector search는 CPU와 메모리 접근 패턴이 중요함
- metadata filtering과 ANN search를 안정적으로 처리해야 함
- 운영 단계에서 latency와 memory efficiency가 중요함

### LanceDB

LanceDB는 AI용 multimodal lakehouse와 retrieval 계층을 지향합니다.

특징:

- vector search
- full-text search
- SQL
- multimodal data
- Python, TypeScript, Rust SDK
- Lance format 기반

AI 서비스가 단순 vector DB를 넘어 데이터 관리, 검색, 학습 데이터 curation까지 확장될 때 참고할 만합니다.

### Polars

Polars는 Rust로 작성된 빠른 DataFrame/query engine입니다.

의미:

- Python 인터페이스를 제공하지만 core는 Rust
- multi-threaded
- lazy/eager execution
- query optimization
- streaming 처리

이 역시 Python + Rust 조합의 강력한 사례입니다.

---

## 언제 Rust를 쓰면 안 되는가

Rust는 강력하지만 만능이 아닙니다.

### 1. 아직 제품 방향이 자주 바뀌는 경우

요구사항이 매일 바뀌는 단계에서는 Python이 낫습니다.

Rust는 구조가 단단한 대신 변경 속도가 느릴 수 있습니다.

### 2. 병목이 확인되지 않은 경우

느릴 것 같은 느낌만으로 Rust를 도입하면 안 됩니다.

먼저 측정해야 합니다.

### 3. 팀에 Rust 운영 경험이 없는 경우

Rust는 배우면 강력하지만 진입 장벽이 있습니다.

특히 다음을 다룰 사람이 필요합니다.

- ownership
- lifetime
- async Rust
- PyO3
- packaging
- cross-compilation
- profiling
- unsafe boundary

### 4. Training 연구 중심인 경우

모델 학습 연구는 여전히 Python/PyTorch가 압도적으로 좋습니다.

Rust ML 생태계도 성장 중이지만, training ecosystem 전체를 대체하기에는 Python이 훨씬 성숙합니다.

### 5. GPU kernel을 직접 만들려는 경우

GPU kernel은 Rust보다 CUDA/C++ 생태계가 현실적으로 강합니다.

Rust는 GPU kernel 위의 scheduler, gateway, metadata engine, stream runtime에 쓰는 것이 더 실용적입니다.

---

## 학습 로드맵

### Step 1: Rust 기본기

목표:

- ownership
- borrowing
- lifetime
- Result/Option
- trait
- iterator
- module
- cargo

추천 실습:

- log parser
- JSON normalizer
- text chunker
- simple tokenizer

### Step 2: Rust 성능 감각

목표:

- allocation 줄이기
- Vec/String 처리
- slice 사용
- iterator vs loop 비교
- benchmark
- flamegraph

추천 실습:

- 대량 텍스트에서 문장 split
- byte 단위 tokenizer
- top-k merge
- deduplication

### Step 3: 병렬 처리

목표:

- std::thread
- rayon
- channel
- Send/Sync
- lock 최소화

추천 실습:

- batch tokenizer
- parallel document parser
- embedding metadata transform

### Step 4: async Rust

목표:

- async/await
- Future
- Tokio
- stream
- timeout
- cancellation
- backpressure

추천 실습:

- SSE server
- websocket echo server
- streaming proxy
- token stream multiplexer

### Step 5: Python binding

목표:

- PyO3
- maturin
- Python wheel
- error conversion
- batch API 설계
- GIL 처리

추천 실습:

- Rust tokenizer를 Python에서 import
- Rust parser를 FastAPI에서 호출
- Rust vector transform 모듈 만들기

### Step 6: AI core project

추천 프로젝트:

1. `rust_ai_token_counter`
   - 여러 모델 tokenizer별 token count
   - batch API
   - Python binding

2. `rust_rag_chunker`
   - Markdown/HTML 문서 chunking
   - heading-aware split
   - overlap
   - metadata extraction

3. `rust_stream_gateway`
   - SSE relay
   - timeout
   - cancellation
   - heartbeat

4. `rust_retrieval_merge`
   - dense/sparse score merge
   - deduplication
   - threshold filtering
   - top-k selection

5. `rust_ai_policy_engine`
   - tenant quota
   - model routing
   - cache policy
   - rate limiting

---

## 예시 프로젝트 구조

이 저장소가 앞으로 실제 코드까지 포함한다면 다음 구조가 좋습니다.

```text
rust_ai_core_stack/
  README.md
  docs/
    architecture.md
    python_rust_integration.md
    benchmarking.md
    deployment.md
  crates/
    rust_ai_tokenizer/
      Cargo.toml
      src/
        lib.rs
    rust_ai_chunker/
      Cargo.toml
      src/
        lib.rs
    rust_ai_retrieval/
      Cargo.toml
      src/
        lib.rs
    rust_ai_stream/
      Cargo.toml
      src/
        lib.rs
  python_examples/
    fastapi_app/
      app/
        main.py
        services/
          chat_service.py
      pyproject.toml
  benchmarks/
    tokenization_bench.md
    parser_bench.md
    streaming_bench.md
```

---

## Rust AI Core 후보 모듈 목록

### 우선순위 높음

| 모듈 | 설명 | Rust 도입 가치 |
|---|---|---|
| tokenizer/token counter | prompt 길이 계산, batch tokenization | 매우 높음 |
| chunker | RAG 문서 split, overlap, metadata | 높음 |
| parser | Markdown/HTML/code/log parser | 높음 |
| retrieval merge | dense/sparse/hybrid result merge | 높음 |
| streaming gateway | SSE/websocket relay | 높음 |
| policy engine | quota, routing, rate limit | 높음 |

### 우선순위 중간

| 모듈 | 설명 | Rust 도입 가치 |
|---|---|---|
| cache key generator | prompt/request cache key | 중간 |
| JSON normalizer | 대량 JSON 변환 | 중간 |
| embedding preprocessor | vector normalization | 중간 |
| evaluation metric core | 대량 metric 계산 | 중간 |

### 신중히 판단

| 모듈 | 설명 | 주의 |
|---|---|---|
| full agent runtime | 요구사항 변경이 잦음 | 너무 일찍 Rust화 비추천 |
| training framework | PyTorch 생태계 우위 | Python 추천 |
| GPU kernel | CUDA/C++ 생태계 우위 | Rust 단독 비추천 |
| business logic 전체 | 변경 많음 | Python/Java/Spring이 나을 수 있음 |

---

## 실전 의사결정 체크리스트

아래 질문에 많이 해당하면 Rust 도입을 고려합니다.

```text
□ 이 작업이 CPU를 많이 쓰는가?
□ 같은 처리를 초당 수천 번 이상 반복하는가?
□ Python profiling에서 hotspot으로 확인되었는가?
□ p95/p99 latency에 영향을 주는가?
□ 큰 문자열/byte/vector를 많이 다루는가?
□ 병렬 처리로 이득이 큰가?
□ 메모리 사용량이 운영 문제를 만들고 있는가?
□ API 경계가 안정적이고 자주 바뀌지 않는가?
□ batch API로 설계할 수 있는가?
□ 팀이 Rust 코드를 유지보수할 수 있는가?
```

반대로 아래에 해당하면 Rust 도입을 미룹니다.

```text
□ 아직 PoC 단계다.
□ 요구사항이 매일 바뀐다.
□ 병목을 측정하지 않았다.
□ Python으로도 충분히 빠르다.
□ 팀에 Rust 경험자가 없다.
□ 디버깅/배포 복잡도를 감당하기 어렵다.
□ 단순 CRUD 또는 일반 비즈니스 로직이다.
```

---

## 최종 정리

가장 좋은 전략:

```text
Python prototype
    ↓
Profiling
    ↓
Rust performance core
    ↓
Python binding or Rust service
    ↓
Production optimization
```

추천:

- Python = AI orchestration
- Rust = 핵심 엔진
- CUDA/C++ = GPU inference kernel

비추천:

- 처음부터 전부 Rust
- 병목 측정 없이 Rust 도입
- agent 실험과 training 연구를 Rust로 시작

앞으로 AI 엔지니어링 경쟁은 단순히 모델을 호출하는 능력이 아니라, 모델 주변의 시스템을 얼마나 잘 만드는가로 갈 가능성이 큽니다.

그 관점에서 Rust는 AI 인프라, AI gateway, retrieval engine, streaming runtime, inference scheduler 같은 영역에서 큰 차별화가 될 수 있습니다.

---

## 참고 자료

아래 자료를 기준으로 내용을 정리했습니다. 버전과 API는 시간이 지나면 바뀔 수 있으므로 실제 구현 전에는 각 공식 문서를 다시 확인해야 합니다.

- Rust 공식 문서: https://www.rust-lang.org/
- The Rust Programming Language: https://doc.rust-lang.org/book/
- PyO3: https://github.com/PyO3/pyo3
- PyO3 User Guide: https://pyo3.rs/
- maturin User Guide: https://www.maturin.rs/
- Tokio Tutorial: https://tokio.rs/tokio/tutorial
- Hugging Face Tokenizers: https://huggingface.co/docs/tokenizers/
- Hugging Face Transformers fast tokenizers: https://huggingface.co/docs/transformers/
- Qdrant: https://qdrant.tech/
- LanceDB: https://docs.lancedb.com/
- LanceDB Rust crate: https://docs.rs/lancedb
- Polars User Guide: https://docs.pola.rs/
- Polars Rust crate: https://docs.rs/polars/

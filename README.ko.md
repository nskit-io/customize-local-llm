[🇬🇧 English](README.md) | [🇯🇵 日本語](README.ja.md)

# Hybrid LLM Architecture

**성격은 로컬 모델, 팩트는 클라우드 모델 — 프라이버시 우선 AI 캐릭터 아키텍처.**

> [NVatar](https://github.com/nskit-io/nvatar) 프로젝트에서 추출 — 로컬 우선(성격은 로컬 모델, 사실은 클라우드 층)으로 설계된 AI 아바타 채팅 시스템.

---

## 왜 만들었나

LLM 라우팅은 존재합니다 (RouteLLM, FrugalGPT, Semantic Router). 하지만 기존 연구는 **비용**이나 **난이도** 최적화에 집중합니다 — 이 패턴은 대신 **기능별 분리**를 다룹니다:

| 레이어 | 위치 | 이유 |
|--------|------|------|
| 성격 & 대화 | **로컬** (Gemma 4 26B MoE) | 프라이버시, 지연시간, 캐릭터 일관성 |
| 팩트 & 검색 | **클라우드** (Claude via CSW) | 정확성, 도구 접근, 웹 검색 |

## 문제

로컬 LLM은 성격은 좋지만 **사실을 날조**합니다. 클라우드 LLM은 정확하지만 잡담에 **비용이 과다**하고 사적인 대화가 제3자에게 노출됩니다.

**해결: 난이도가 아닌 기능별 분리.**

## 아키텍처

```
사용자 메시지
    │
    ▼
컨텍스트 라우터 (Gemma, 로컬)
    │
    ├─ T1-T4, T8-T10 ──→ Gemma (로컬)
    │   잡담, 감정, 조언,       성격 레이어
    │   의견, 기억, 인사         저지연, 프라이빗
    │
    ├─ T5-T6 ──────────→ CSW → Claude (클라우드)
    │   사실, 뉴스              지식 레이어
    │                           정확 + 웹검색
    │
    └─ T7 ─────────────→ Gemma (로컬, 제한)
        추천                    한계를 인정
                                (위치/검색 인프라 없음)
```

## 10가지 대화 유형

| 코드 | 유형 | 라우팅 | Thinking |
|------|------|--------|----------|
| T1 | 잡담 | Gemma | OFF |
| T2 | 감정 공유 | Gemma | OFF |
| T3 | 조언 구하기 | Gemma | OFF |
| T4 | 심층 토론 | Gemma | **ON** |
| T5 | 사실 질문 | **클라우드** | - |
| T6 | 뉴스/시사 | **클라우드** | - |
| T7 | 추천 | Gemma (제한 — 위치/검색 인프라 없음) | OFF |
| T8 | 기억 참조 | Gemma | OFF |
| T9 | 아바타 질문 | Gemma | OFF |
| T10 | 인사/작별 | Gemma | OFF |

**핵심 인사이트:** 우리 경험상 캐릭터 대화의 대부분은 성격 레이어(T1-T4, T7-T10)이고, 클라우드가 필요한 것은 사실·뉴스 질문(T5-T6)뿐입니다. 측정한 비율을 공개하지 않았으므로 고정 비율이 아니라 정성적 관찰로 보십시오.

## "찾아볼까?" 패턴

몰래 클라우드로 라우팅하지 않고, 캐릭터가 **허락을 구합니다**:

```
사용자: "서울 날씨 어때?"
아바타: "찾아볼까?"
→ 사용자가 자연스러운 긍정 응답으로 확인
→ CSW WebSearch 실행
아바타: "서울 지금 18도래! 오후에 비 온다는데 우산 챙겨~"
```

팩트 조회 중에도 **친구 페르소나**를 유지합니다.

## 속도 비교

로컬 분류는 클라우드 왕복을 피합니다. 대부분의 대화가 로컬에서 약 2초 이내에 완료됩니다(우리 환경의 대략적 수치이며 정식 벤치마크가 아님).

## 왜 그냥 큰 클라우드 모델을 안 쓰나?

| 관심사 | 로컬 우선 하이브리드 | 클라우드 전용 |
|--------|---------------------|--------------|
| 프라이버시 | 대화가 로컬에 유지 | 모든 데이터 클라우드 전송 |
| 지연시간 | 대부분의 메시지가 로컬에서 ~2초 이내 | 모든 메시지에 클라우드 왕복이 추가 |
| 비용 | 사실·뉴스 메시지(T5-T6)만 클라우드 | 100% 클라우드 |
| 캐릭터 일관성 | 로컬 모델 = 하나의 성격 | API 변경이 동작 변화 유발 |
| 오프라인 | 인터넷 없이 채팅 가능 | 오프라인 불가 |

## 관련 프로젝트

- [**라이브 데모**](https://nskit-io.github.io/nvatar-demo/) — 데모 서버 점검으로 라이브 데모가 일시 중단되었습니다. 소스([nvatar-demo](https://github.com/nskit-io/nvatar-demo))에서 직접 실행할 수 있습니다.

- [**avatar-chat**](https://github.com/nskit-io/avatar-chat) — Gemma 캐릭터 AI 프롬프트 엔지니어링 패턴
- [**chat-like-human-memory**](https://github.com/nskit-io/chat-like-human-memory) — 9D 감정 추적 + 성격 진화 + 3단계 기억
- [**vrm-studio**](https://github.com/nskit-io/vrm-studio) — Three.js + WebSocket 3D VRM 아바타 채팅방
- [**CSW**](https://github.com/nskit-io/csw) — Claude Subscription Worker, 이 R&D 프로토타입의 클라우드 레이어에 쓰인 개인 개발자 도구
- [**portable-ai-companion**](https://github.com/nskit-io/portable-ai-companion) — 크로스앱 프랜차이즈 아키텍처

## 라이선스

CC BY-NC-SA 4.0 — [LICENSE](LICENSE) 참조

> 이 리포는 설계 패턴과 레퍼런스를 MIT 라이선스로 공개한 것입니다. NSKit 프레임워크 본체(런타임·스캐폴딩·검사기·운영 표준)는 Neoulsoft Inc. 의 비공개 자산이며 별도 상용 라이선스로 제공됩니다.

---

## 이 연구를 지원해주세요

NVatar는 하이브리드 AI 아키텍처의 최전선을 탐구하는 독립 R&D 프로젝트입니다. 1인 대표가 외부 투자 없이 진행하고 있습니다.

**후원 및 투자 문의**
- Email: [nskit@nskit.io](mailto:nskit@nskit.io)
- Organization: [NSKit](https://nskit.io) by Neoulsoft Inc.

<p align="center">
  <a href="https://github.com/nskit-io">github.com/nskit-io</a>
</p>

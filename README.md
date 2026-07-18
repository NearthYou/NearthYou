# 이시원 | AI Native Systems Developer

> I design environments where AI can execute scoped work safely, while architecture, verification, deployment authority, and final accountability remain human-owned.

저에게 AI Native는 문제를 **명세·역할·검증 단위로 분해해 AI에 위임하되, 결과의 정확성과 최종 책임은 사람이 소유하는 개발 방식**입니다. AI는 구현 속도를 높이는 실행 주체로 활용하고, 아키텍처·품질 기준·배포 권한은 직접 통제합니다.

## 검증의 소유권

AI의 설명을 결론으로 사용하지 않습니다. Source of Truth를 먼저 정하고 code path, state transition, test output을 직접 추적합니다. 문서와 구현이 다르면 구현을 기준으로 claim을 낮추고, 근거가 없는 성공 수치는 제거합니다.

대표 사례는 C 기반 Mini DBMS의 MVCC 검토입니다.

- 실제 구현은 row version MVCC가 아니라 **table-snapshot COW MVCC**입니다.
- `mv_gc`는 오래된 row version을 회수하지 않고 종료된 snapshot을 active snapshot 목록에서 해제합니다.
- `gc_tabs`는 더 이상 active snapshot이 참조하지 않는 table-version chain을 정리합니다.
- autocommit write는 1,024개 hashed mutex shard로 직렬화되며 true row-level lock이 아닙니다.
- persistence는 temp CSV 작성 후 rename이며 full WAL·fsync·replay recovery를 구현하지 않았습니다.

근거: [정정 README](https://github.com/NearthYou/SQL3_WednsdayCodingClub/blob/docs/ai-native-evidence/README.md), [`mvcc.c`](https://github.com/NearthYou/SQL3_WednsdayCodingClub/blob/docs/ai-native-evidence/src/db/mvcc.c), [`test_mvcc.c`](https://github.com/NearthYou/SQL3_WednsdayCodingClub/blob/docs/ai-native-evidence/tests/test_mvcc.c), [검토 PR #4](https://github.com/NearthYou/SQL3_WednsdayCodingClub/pull/4)

## AI 작업 운영 방식

Codex와 Claude Code에 막연히 구현을 요청하지 않고 다음 계약을 먼저 고정합니다.

1. 명세와 문서를 Source of Truth로 선언합니다.
2. `AGENTS.md`와 작업 문서에 file ownership, 변경 금지 영역, 실행 명령을 기록합니다.
3. test·CI·harness를 완료 조건으로 두고 결과를 자동 판정합니다.
4. Agent의 완료 보고에 changed files, commands, pass/fail, remaining risk를 포함시킵니다.
5. 사람이 diff, 실행 결과, claim의 근거를 검토한 뒤에만 통합·배포합니다.

실제 규칙: [SketchCatch `AGENTS.md`](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/AGENTS.md), [docs contract](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/docs/README.md), [development workflow](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/docs/development.md)

## 핵심 증거 3개

### 1. Mini DBMS — 설명이 아니라 data flow로 검증

POSIX socket과 pthread 기반 API/DB 이중 thread pool, table-snapshot COW, private working copy rollback, table-head/base conflict detection을 구현했습니다. 문서의 row lock·WAL 표현은 실제 코드와 맞지 않아 현재 review PR에서 정확한 보장 범위로 정정했습니다.

### 2. Agent collaboration rules — AI가 일할 수 있는 환경

명세, 담당 file, 금지 계약, test/CI, 완료 보고 형식을 repository contract로 만들었습니다. Agent는 제한된 범위를 실행하고, 최종 설계 판단과 integration은 사람이 소유합니다.

### 3. SketchCatch — AI 추천과 결정론적 승인 경계

자연어 요구사항을 architecture draft와 Terraform preview로 연결하되, AI는 추천·설명·검토를 담당하고 구조 correctness와 배포 경계는 graph validation, rule, explicit user approval이 담당합니다. 현재 **High detection과 finding 기록은 implemented**, severity만으로 approval을 막는 block은 **planned**입니다. 승인되지 않은 Terraform apply는 허용하지 않습니다.

근거: [ADR 0001](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/docs/adr/0001-ai-assists-deterministic-architecture-flow.md), [deployment runbook](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/docs/deployment.md), [상태 정정 PR #473](https://github.com/NearthYou/SketchCatch/pull/473)

## 기반 구현 프로젝트

- **LLM from scratch:** BPE → embedding → causal multi-head attention → GPT → pretraining/fine-tuning을 팀 원본 history, 개인 contribution map, standalone reproduction으로 분리했습니다. [standalone repository](https://github.com/NearthYou/llm-from-scratch-lab), [PR #1](https://github.com/NearthYou/llm-from-scratch-lab/pull/1), [원본 개선 PR](https://github.com/Soldbone/gpt-lab/pull/40)
- **MiniRedis:** API/server/TTL과 single-process k6 atomicity scenario를 구현했습니다. 단일 Uvicorn event loop 밖의 thread/process/distributed 원자성은 주장하지 않습니다. [evidence PR #2](https://github.com/NearthYou/MyMiniRedis_WednsdayCodingClub/pull/2)
- **mini-react:** VDOM/diff/patch/history는 팀·페어 프로그래밍 공동 구현입니다. 제 repository-visible 개인 작업은 demo integration, UI, example, documentation입니다. [collaboration evidence PR #1](https://github.com/NearthYou/mini-react/pull/1)
- **StudyTube:** 저장 분석 RAG, JSON-RPC `youtube.lookup`, 최대 4 iterations의 tool Agent, 명시적인 외부 서비스 fallback을 구현했습니다. query-time pgvector search는 아직 연결되지 않았습니다. [StudyTube path](https://github.com/NearthYou/agentic-board/tree/docs/siwon/ai-native-evidence/siwon), [evidence PR #110](https://github.com/NearthYou/agentic-board/pull/110)

## Claim → Evidence → Verification

| Claim | Evidence | Verification |
| --- | --- | --- |
| table-snapshot COW MVCC와 conflict detection | [`mvcc.c`](https://github.com/NearthYou/SQL3_WednsdayCodingClub/blob/docs/ai-native-evidence/src/db/mvcc.c), [`dbapi.c`](https://github.com/NearthYou/SQL3_WednsdayCodingClub/blob/docs/ai-native-evidence/src/db/dbapi.c) | [`test_mvcc.c`](https://github.com/NearthYou/SQL3_WednsdayCodingClub/blob/docs/ai-native-evidence/tests/test_mvcc.c), [`test_tx.c`](https://github.com/NearthYou/SQL3_WednsdayCodingClub/blob/docs/ai-native-evidence/tests/test_tx.c) |
| Agent file ownership·SSOT·completion contract | [`AGENTS.md`](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/AGENTS.md), [`docs/README.md`](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/docs/README.md) | [`docs/development.md`](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/docs/development.md), repository harness |
| High detection과 approval/apply 경계 | [deployment safety gate](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/apps/api/src/deployments/deployment-safety-gate.ts) | [safety-gate tests](https://github.com/NearthYou/SketchCatch/blob/docs/sw/ai-native-evidence/apps/api/src/deployments/deployment-safety-gate.test.ts) |
| GPT 학습 흐름과 개인 기여 분리 | [architecture](https://github.com/NearthYou/llm-from-scratch-lab/blob/build/evidence/docs/architecture.md), [contribution map](https://github.com/NearthYou/llm-from-scratch-lab/blob/build/evidence/docs/contribution-map.md) | [54-test suite와 CPU smoke](https://github.com/NearthYou/llm-from-scratch-lab/blob/build/evidence/artifacts/current/smoke-result.json) |
| MiniRedis TTL와 server-side INCR 경계 | [`core.py`](https://github.com/NearthYou/MyMiniRedis_WednsdayCodingClub/blob/docs/ai-native-evidence/miniredis/core.py), [`incr-concurrency.js`](https://github.com/NearthYou/MyMiniRedis_WednsdayCodingClub/blob/docs/ai-native-evidence/k6/miniredis/incr-concurrency.js) | [`test_core.py`](https://github.com/NearthYou/MyMiniRedis_WednsdayCodingClub/blob/docs/ai-native-evidence/miniredis/tests/test_core.py), [`test_server.py`](https://github.com/NearthYou/MyMiniRedis_WednsdayCodingClub/blob/docs/ai-native-evidence/miniredis/tests/test_server.py) |
| mini-react 공동 VDOM/runtime 구현 | [`diff.js`](https://github.com/NearthYou/mini-react/blob/docs/ai-native-evidence/src/lib/diff.js), [`rootRuntime.js`](https://github.com/NearthYou/mini-react/blob/docs/ai-native-evidence/src/rootRuntime.js) | [`diff.test.js`](https://github.com/NearthYou/mini-react/blob/docs/ai-native-evidence/tests/lib/diff.test.js), [`rootRuntime.test.js`](https://github.com/NearthYou/mini-react/blob/docs/ai-native-evidence/tests/rootRuntime.test.js) |
| StudyTube RAG·MCP·bounded Agent·fallback | [`ai/main.py`](https://github.com/NearthYou/agentic-board/blob/docs/siwon/ai-native-evidence/siwon/ai/main.py), [`ai-proxy.service.ts`](https://github.com/NearthYou/agentic-board/blob/docs/siwon/ai-native-evidence/siwon/api/src/ai-proxy.service.ts) | [`ai/test_main.py`](https://github.com/NearthYou/agentic-board/blob/docs/siwon/ai-native-evidence/siwon/ai/test_main.py), [`web/tests`](https://github.com/NearthYou/agentic-board/tree/docs/siwon/ai-native-evidence/siwon/web/tests) |

## 기술 스택

`C` · `POSIX socket` · `pthread` · `JavaScript` · `TypeScript` · `Python` · `NumPy` · `PyTorch` · `FastAPI` · `NestJS` · `React` · `PostgreSQL` · `pgvector` · `Docker` · `Terraform` · `AWS` · `k6` · `GitHub Actions`

저장소의 목적은 기술 목록을 늘리는 것이 아니라, 각 claim을 code·commit·test로 되돌아가 검증할 수 있게 만드는 것입니다.

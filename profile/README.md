# jyoliar-lab

개인 홈랩과 사이드 프로젝트를 AI agent와 함께 운영한다. \
코드에는 "무엇"만 남고 결정의 이유가 없어서, agent는 지난 결정을 모른 채 같은 논의를 반복했다. \
agent가 내놓는 코드 · 테스트 · 문서를 사람이 전부 읽어야 해서 검수가 병목이었다. \
계획 · 집행 · 규칙 정비가 한 세션에 섞여 맥락이 오염됐다. \
그래서 결정의 이유를 ADR로 남기고, 사람은 산출물 대신 시나리오와 대조표를 검수하며, 계획 · 집행 · 거버넌스를 서로 다른 저장소의 agent로 나눴다.

## 저장소

하네스는 CLAUDE.md · skill · hook을 묶은 agent 설정이다. \
역할 하나가 하네스 하나다.

| 저장소 | 역할 | 담는 것 |
|---|---|---|
| [ops-workflow](https://github.com/jyoliar-lab/ops-workflow) | 계획과 기록 | 작업의 상태는 [Issues](https://github.com/jyoliar-lab/ops-workflow/issues) · [Projects](https://github.com/orgs/jyoliar-lab/projects/1)에, 결정의 이유는 [docs/](https://github.com/jyoliar-lab/ops-workflow/tree/main/docs)에. 논의하고 ADR 초안을 쓰고 티켓을 발행하는 **design-partner**, ADR을 심사하는 **adr-judge**의 하네스 |
| [ticket-engineer](https://github.com/jyoliar-lab/ticket-engineer) | 집행 | 티켓 하나를 받아 코드 저장소에서 일하고 PR과 보고를 남기는 **ticket-engineer**의 하네스 |
| [ops-workflow-steward](https://github.com/jyoliar-lab/ops-workflow-steward) | 거버넌스 | 프로세스의 규칙을 ADR로 쓰고 두 하네스에 반영하는 **steward**의 하네스. 자기 하네스는 고치지 못한다 |
| [obs-toolkit](https://github.com/jyoliar-lab/obs-toolkit) | 산출물 | 이 흐름으로 만든 관측 분석 도구 |
| 코드 저장소 | 티켓이 가리키는 곳 | 인프라 [jyoliar-k8s](https://github.com/jyoliar-careerhub/jyoliar-k8s) · [onpremise-k8s](https://github.com/jyoliar-careerhub/onpremise-k8s) · [jyoliar-platform-terraform](https://github.com/jyoliar-careerhub/jyoliar-platform-terraform)은 공개, careerhub 서비스는 비공개(요청 시 열람) |

## 일의 흐름

한 세션은 논의 하나 또는 티켓 하나를 다루고 닫힌다. \
각 단계 끝의 **사람:** 줄이 사람이 승인하거나 직접 하는 지점이다.

### ops-workflow

#### 1. 논의 — discussion

문제는 design-partner가 사람을 인터뷰해서 정의한다. \
질문은 채팅에서만 하고, 문서는 옵션까지 갖춘 뒤 한 번만 쓴다. \
정해야 할 것은 축으로 쪼개고, 사람의 의도로 닫히는 축과 실측 · 조사가 필요한 축을 가른다. \
열린 축마다 "안 한다"를 포함한 옵션 셋 이상을 벌리고, 결정은 문서에 판정 한 줄로만 남는다. \
상태는 `active → done`(ADR 파생) 또는 `rejected`다. \
조사에는 티켓이 없다 — 탐색은 기록의 일이지 작업이 아니다.

**사람:** 열린 축마다 판정을 채팅으로 내린다. 문서의 판정 줄이 그 기록이다.

#### 2. digest

같은 목표의 논의들이 수렴하면 결론을 digest 한 건으로 증류한다. \
digest는 규범이 아니다 — 각 결론이 어디에 사는지(ADR · 티켓 · 코드)를 가리키는 지도이고, 읽기 순서는 ADR → 티켓 → digest다. \
결론이 바뀌면 고쳐 쓰고, 목표 자체가 바뀌면 새 digest로 교체한다. \
상태는 `current → superseded`다.

**사람:** docs/에 쓰는 것은 전부 사람 승인 뒤다. 승인 없는 지식 쓰기는 없다.

#### 3. ADR — 결정의 영수증

되돌리기 어려운 결정 하나만 ADR로 남긴다. \
기준은 둘이다 — 되돌리는 데 한 시간이면 ADR이 아니라 티켓이고, 한 ADR에는 함께 무효가 되는 결정만 담는다. \
"왜"만 적는다. 수치 · 리소스명 · 필드값은 코드의 것이다. \
상태는 `proposed → accepted · rejected · superseded`다. \
`proposed`에서만 본문을 고칠 수 있고, 벗어나면 본문이 통째로 얼어붙는다. 동결은 공표(push) 시점에 발효한다. \
얼어붙은 결정을 바꾸는 길은 새 ADR로 supersede하는 것뿐이고, 부분 폐기는 없다. \
심사는 adr-judge가 한다 — 초안을 쓴 agent는 심사하지 않는다. \
프로세스 자체의 규칙도 ADR로 쓰되, 정책 ADR은 심사를 통과해도 `proposed`로 남긴다 — 운영 중에 자주 고쳐지기 때문이다.

**사람:** accept를 승인한다. adr-judge 세션을 여는 것도 사람이다.

#### 4. 티켓

작업은 전부 ops-workflow의 GitHub Issues다. 코드 저장소에는 이슈가 없고 PR · 커밋이 티켓 번호를 참조만 한다. \
Issue Types는 넷이다 — 묶는 **Epic · Feature**, 수행하는 **Task · Bug**. Epic → Feature → 리프 3층을 권장한다. \
Epic은 공표된 근거 ADR을 전제한다. Feature와 리프의 근거는 선택이다. \
리프는 고쳐도 되는 저장소(target)와 참조 저장소(ref), 그리고 선행 의존을 명시한다. 선행의 완료 기준은 close다. \
Status는 여섯이다 — `Draft`(진행해도 좋은지 확신이 없다) · `Todo` · `In Progress` · `In Review` · `Done` · `Cancelled`(이행하지 않기로 판정. 티켓은 지우지 않는다). \
긴급도 · 보류 · 조사의 진척은 상태가 아니다. \
이슈가 문서를 가리키고, 문서는 이슈를 가리키지 않는다.

**사람:** Status는 사람의 것이다. `Draft → Todo`, 착수 지시, close 전부 사람이 한다. 도구는 타입별 초기값만 넣는다.

### ticket-engineer

#### 5. 착수

사람이 티켓 번호를 지정하면 ticket-engineer가 티켓과 근거 ADR을 읽고 Acceptance Criteria(검수 기준)와 Requirements(제약과 범위 밖 축)를 그대로 추출한다. \
범위 밖 축에 닿으면 임의로 닫지 않고 멈춰서 묻는다. \
착수 게이트는 스크립트가 판정한다 — 계층 전체의 근거 ADR이 accepted인가, Status가 Draft가 아닌가, 열린 선행이 없는가, 저장소 절이 있는가. \
exit 코드가 다음 행동을 정하고, 거부 사유가 할 일을 말한다. agent는 우회하지 않는다. \
작업은 티켓별 worktree에서 한다. 참조 저장소는 읽기만 한다.

**사람:** 티켓 번호를 지정한다. 게이트 거부의 해소(ADR accept · Draft 해제 · 저장소 절 기입)도 사람이 한다.

#### 6. 작업

##### 6-1. 애플리케이션 — tdd-loop를 트리거하는 AC가 있을 때

시나리오를 Given / When / Then 표로 티켓 코멘트에 올린다. \
승인된 단위부터 red(실패하는 테스트) → red 검토(subagent) → green과 refactor(subagent) → verify(subagent)가 흐른다. \
red 뒤에 소스를 쓰지 않고 green 뒤에 테스트를 고치지 않는다. 단계마다 커밋을 닫아 diff로 읽힌다. \
verify는 AC 항목별 대조표를 코멘트로 게시한다. \
사람은 코드가 아니라 시나리오 표와 대조표를 본다.

**사람:** 시나리오 표를 승인한다. verify 뒤 다음 단위 · ready · 되감기를 지시한다.

##### 6-2. 인프라

ArgoCD GitOps 저장소(jyoliar-k8s)와 Terraform(jyoliar-platform-terraform)은 PR로 반영한다. \
Ansible(onpremise-k8s)은 로컬에서 반영한 뒤 PR을 만든다. \
agent의 kubectl은 전용 ServiceAccount로 클러스터 전역 읽기 전용이고 Secret 값은 읽지 못한다 — 클러스터를 고치는 길은 PR뿐이다. \
검증은 테스트가 아니라 배포 뒤 실측이다. 명령 한 번과 그 출력을 코멘트에 그대로 남긴다. \
머지 뒤 수렴을 확인할 때까지 티켓은 열려 있다.

**사람:** PR을 머지한다.

##### 6-3. PR과 완료 보고

PR은 관문 스크립트로만 연다 — 본문 규약과 티켓 연결을 스크립트가 넣는다. 머지 게이트는 우회하지 않는다. \
완료 보고 전에 preflight가 티켓의 target 전부에 PR · 참조 · 체크가 갖춰졌는지 확인한다. \
보고는 AC 항목별 대조표 코멘트다. 충족하지 못한 항목이 있으면 보고가 아니라 논의가 먼저다.

**사람:** 게시된 코멘트로 검수하고 수정을 지시한다.

##### 6-4. 검수와 close

PR 머지, `In Review → Done`, close는 전부 사람이 한다. PR 머지는 티켓 종료가 아니다. \
agent의 관찰은 티켓 코멘트에 쌓이고, 사람이 증류해 문서로 옮길 때만 지식이 된다.

**사람:** 전부.

## 실제 사례

인프라 한 체인과 애플리케이션 한 체인이다. digest 한 줄은 다른 작업에서 가져왔다.

**인프라 — 인증을 Zitadel로 위임**

[논의 — Epic의 일 쪼갬](https://github.com/jyoliar-lab/ops-workflow/blob/main/docs/careerhub/discussion/260807_0158_zitadel-epic-work-breakdown.md) → [ADR-0030 — IdP로 Zitadel을 채택한다](https://github.com/jyoliar-lab/ops-workflow/blob/main/docs/careerhub/adr/adr-0030-adopt-zitadel-as-idp.md) (accepted) → [Epic #43](https://github.com/jyoliar-lab/ops-workflow/issues/43) → [Feature #44](https://github.com/jyoliar-lab/ops-workflow/issues/44) → [Task #50 — masterkey 보존](https://github.com/jyoliar-lab/ops-workflow/issues/50) → [jyoliar-platform-terraform#2](https://github.com/jyoliar-careerhub/jyoliar-platform-terraform/pull/2) · [jyoliar-k8s#20](https://github.com/jyoliar-careerhub/jyoliar-k8s/pull/20)

**애플리케이션 — obs-toolkit의 R·E·D 환원 연산**

[논의 — 만들 것인가 빌릴 것인가](https://github.com/jyoliar-lab/ops-workflow/blob/main/docs/ai-agent/discussion/260917_1726_obs-toolkit-build-vs-borrow.md) → [Feature #92](https://github.com/jyoliar-lab/ops-workflow/issues/92) → [Task #102](https://github.com/jyoliar-lab/ops-workflow/issues/102) → [시나리오 코멘트](https://github.com/jyoliar-lab/ops-workflow/issues/102#issuecomment-6232081604) → [verify 대조표](https://github.com/jyoliar-lab/ops-workflow/issues/102#issuecomment-6232420944) → [obs-toolkit#17](https://github.com/jyoliar-lab/obs-toolkit/pull/17)

**digest** — [레거시 서비스 공개 노출 청사진](https://github.com/jyoliar-lab/ops-workflow/blob/main/docs/careerhub/digest/261026_1715_legacy-public-exposure-blueprint.md): 논의 [261026_1610](https://github.com/jyoliar-lab/ops-workflow/blob/main/docs/careerhub/discussion/261026_1610_legacy-public-exposure-auth-disposition.md)의 결론을 증류해 [ADR-0034](https://github.com/jyoliar-lab/ops-workflow/blob/main/docs/homelab/adr/adr-0034-terraform-gitops-dependency-direction.md)와 [Feature #133](https://github.com/jyoliar-lab/ops-workflow/issues/133)으로 나갔다.

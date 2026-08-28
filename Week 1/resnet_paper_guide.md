# ResNet 논문 가이드 (허브)

- 논문: He, Zhang, Ren, Sun, *Deep Residual Learning for Image Recognition*, arXiv:1512.03385v1
  [source: sources/resnet-study/arXiv-1512.03385v1.tar.gz]
- 스터디 자료 4종: 치트시트 / 쿡북 / 빈칸 노트북 / 퀴즈
  [source: sources/resnet-study/]

## 이 폴더의 문서

시작 전 배경은 [선수자료](resnet_prerequisites.md). 개념 요약은 [치트시트](A.ing_resnet_cheat_sheet.md), 구현 API는 [쿡북](../Week%202/resnet_cookbook.md),
확인 문제는 [퀴즈](resnet_questions.md)와 [퀴즈_모범답안](resnet_questions_sample_answer.md). 노트북 두 개는 `자료/` 폴더에 있습니다.

## 참조 문법

| 표기 | 가리키는 것 |
| --- | --- |
| `[R05]` | 이 허브의 R 항목 |
| `[CS§n]` | 치트시트 n번 절 |
| `[Cn-m]` | 쿡북 n절 m번 항목 |
| `[N-k]` | 빈칸 노트북 k번 빈칸 |
| `[Qn]` | 퀴즈 n번 문항 |
| `[An]` | 퀴즈 모범답안 n번 문항 |

허브의 각 R은 자기를 참조하는 자료 좌표를 `자료 좌표` 칸에 전부 나열하고,
각 자료는 자기 절이 어느 R인지 밝힙니다 (**양방향**).

## 각 R 항목의 6칸 템플릿

| 칸 | 내용 |
| --- | --- |
| `논문 위치` | §번호, 식 번호, Table/Fig 번호 |
| `읽기 전 질문` | 이 대목을 읽기 **전에** 품고 갈 질문 1~2개 |
| `원문 핵심` | 영어 원문 발췌 1~2문장 (3줄 이내) |
| `확인 질문` | **답이 논문에만 있는** 질문 2~3개 + 어디를 볼지 |
| `해석 노트` | 논문이 **명시하지 않은 것**만 — 오해 교정·코드 연결·자료 간 모순 (3줄) |
| `자료 좌표` | `[CS§n] [Cn-m] [N-k] [Qn] [An]` |

읽는 법: `읽기 전 질문`을 품고 논문의 `논문 위치`를 편 다음, 읽고 나서 `확인 질문`에 답합니다.
`확인 질문`의 답이 허브만 보고 나온다면 그건 이 허브의 버그입니다 — 답은 논문에만 있어야 합니다.

## 범위

이 허브가 덮는 범위는 스터디 4주가 다루는 **§1, §3.1~§3.4, §4.1~§4.2**뿐입니다.

**범위 밖 (의도적으로 다루지 않음)**

- §2 Related Work
- §4.3 Object Detection on PASCAL and MS COCO
- Appendix 전부 (§A Object Detection Baselines, §B Object Detection Improvements, §C ImageNet Localization)

## 주차별 읽기 경로

R 번호는 **논문 순서**라 학습 순서와 다릅니다. 주차별로는 이 순서를 따릅니다.

| 주차             | R 범위                            | 비고                                  |
| -------------- | ------------------------------- | ----------------------------------- |
| Week 1 (논문 정독) | R01 R02 R03 R04 R05 R08 R10 R11 | 개념과 논증                              |
| Week 2 (구현)    | R05 R06 R07 R15 R16 R17 R12     | 노트북 빈칸 순서(cell 9→13→17→25→27)대로     |
| Week 3 (튜닝)    | R13 R15 R14                     | R13은 baseline, 리그전은 여기서 의도적으로 벗어납니다 |
| Week 4 (분석)    | R14 R02 R10                     | 1202층 overfitting이 R02·R10 쌍을 닫는 논증 |

논문 순서대로만 읽으면 R15·R16이 맨 뒤에 오는데, 노트북 cell 25·27은 그 둘 없이는 채울 수 없습니다.
이 표가 그 역전을 막습니다.

## 목차

### §1 Introduction

| ID | 제목 |
| --- | --- |
| [R01](#r01-깊이는-왜-중요한가) | 깊이는 왜 중요한가 |
| [R02](#r02-degradation--overfitting) | Degradation ≠ Overfitting |
| [R03](#r03-구성적-해constructed-solution-논증) | 구성적 해(constructed solution) 논증 |

### §3.1~§3.2 잔차 학습의 정의와 shortcut

| ID | 제목 |
| --- | --- |
| [R04](#r04-hx-대신-fxhx−x) | H(x) 대신 F(x)=H(x)−x |
| [R05](#r05-덧셈이-성립하려면-차원이-같아야-한다-식1식2) | 덧셈이 성립하려면 차원이 같아야 한다 (식1·식2) |
| [R06](#r06-post-activation-왜-relu가-덧셈-뒤인가) | Post-activation: 왜 ReLU가 덧셈 *뒤*인가 |

### §3.3~§3.4 아키텍처와 구현

| ID | 제목 |
| --- | --- |
| [R07](#r07-plain-net과-residual-net을-왜-그렇게-짝지었나) | Plain net과 Residual net을 왜 그렇게 짝지었나 |
| [R08](#r08-option-abc--세-가지-차원-맞추기와-그-비용) | Option A/B/C — 세 가지 차원 맞추기와 그 비용 |
| [R09](#r09-구현-설정--bn-위치he-initimagenet-스케줄) | 구현 설정 — BN 위치·He init·ImageNet 스케줄 |
| [R16](#r16-stage-구조와-downsampling-좌표) | Stage 구조와 downsampling 좌표 |
| [R17](#r17-출력단-gap--fc) | 출력단: GAP + FC |

### §4.1 ImageNet 실험

| ID | 제목 |
| --- | --- |
| [R10](#r10-degradation은-vanishing-gradient가-아니다) | degradation은 vanishing gradient가 **아니다** |
| [R11](#r11-residual-net의-결과) | Residual net의 결과 |
| [R12](#r12-bottleneck과-더-깊은-모델) | Bottleneck과 더 깊은 모델 |

### §4.2 CIFAR-10 실험 (스터디 구현의 직접 근거)

| ID | 제목 |
| --- | --- |
| [R13](#r13-cifar-10-학습-설정-논문-baseline) | CIFAR-10 학습 설정 (논문 baseline) |
| [R14](#r14-실험-결과-읽는-법) | 실험 결과 읽는 법 |
| [R15](#r15-6n2-규칙--option-a-고정--small-stem) | 6n+2 규칙 · Option A 고정 · small stem |

> R16·R17은 논문 위치가 §3.3이라 §3.3~3.4 묶음에 두었습니다. 자료(치트시트 §4·§6, 쿡북 `[5]`, 노트북 Step 16·17)가
> 실제로 쓰는 개념인데 담당 R이 없어 신설한 항목입니다.

---

## R01 깊이는 왜 중요한가

- **논문 위치**: §1 첫 2문단 (`sec:intro`)
- **읽기 전 질문**
    - 층을 쌓으면 정확히 **무엇이** 좋아진다는 주장입니까?
    - 그 주장의 근거로 논문이 든 것은 이론입니까, 실험 결과입니까?
- **원문 핵심**
    - "the ``levels'' of features can be enriched by the number of stacked layers (depth)"
    - "network depth is of crucial importance"
- **확인 질문**
    - 1) 논문이 예로 든 "very deep" 모델의 깊이 범위는 `___`층 ~ `___`층입니까? (§1 2문단에서 두 인용 모델의 깊이를 찾아 적으세요)
    - 2) 저자는 깊이의 중요성을 **자기 실험**으로 논증합니까, 아니면 선행 연구를 인용해 전제로 깔고 넘어갑니까? 근거가 되는 문장 하나를 찾으세요.
    - 3) 2문단 끝에서 "vanishing/exploding gradients"가 **이미 해결된 문제**로 처리되는데, 무엇으로 해결됐다고 적혀 있습니까? (두 가지)
- **해석 노트**
    - 이 항목은 동기 전용이라 스터디 자료 어디도 참조하지 않습니다 — 코드로 옮길 것이 없습니다.
    - 하지만 여기서 gradient 문제를 미리 치워둔 것이 [R10]의 배제 논증을 가능하게 하는 사전 작업입니다. 즉 R01은 R10의 전제입니다.
- **자료 좌표**: [NB 학습목표]

---

## R02 Degradation ≠ Overfitting

- **논문 위치**: §1 4문단, `fig:teaser`(Fig.1) — CIFAR-10, plain 20층 vs 56층
- **읽기 전 질문**
    - "깊은 모델이 더 나쁘다"를 봤을 때 보통 overfitting을 의심합니다.
    - 그 의심을 **한 장의 그림으로** 기각하려면 무엇을 보여야 합니까?
- **원문 핵심**: "such degradation is *not caused by overfitting*, and adding more layers to a suitably deep model leads to *higher training error*"
- **확인 질문**
    - 1) Fig.1의 두 패널(left/right)은 각각 무슨 error입니까? 56층이 20층보다 나쁜 것이 **양쪽 다**인가, 한쪽뿐입니까?
    - 2) Fig.1에서 20층과 56층의 최종값을 읽어 채우세요: train error 20층=___% 56층=___%, test error 20층=___% 56층=___%.
    - 3) 이 그림만 놓고 "56층이 overfit 됐다"고 말할 수 있습니까? 그 판정에 필요한 정보가 Fig.1의 어느 패널에 있는지 짚으세요.
- **해석 노트**
    - 학습자 최대 오해 지점: "깊으면 overfit"은 **train error가 낮을 때만** 성립하는 진단입니다.
    - Fig.1은 train부터 지고 있으므로 일반화 문제가 아니라 **최적화 문제**입니다.
    - 진짜 overfitting 사례는 논문 안에 딱 하나 있습니다 — 1202층([R14]).
    - 그 둘을 나란히 놓는 것이 Week 4의 논증입니다.
- **자료 좌표**: [Q1] [A1] [A2] [NB 학습목표]

---

## R03 구성적 해(constructed solution) 논증

- **논문 위치**: §1 5문단 "There exists a solution *by construction*…"
- **읽기 전 질문**: 깊은 모델이 얕은 모델보다 **원리상** 나쁠 수 없다는 것을 학습 없이 증명하려면 어떤 모델을 손으로 지어 보이면 됩니까?
- **원문 핵심**: "There exists a solution *by construction* to the deeper model: the added layers are *identity* mapping, and the other layers are copied from the learned shallower model." / "our current solvers on hand are unable to find solutions that are comparably good or better than the constructed solution"
- **확인 질문**
    - 1) 구성적 해에서 추가된 층이 맡는 mapping은 무엇이며, 나머지 층의 가중치는 어디서 옵니까?
    - 2) 이 구성이 성립하면 깊은 모델의 training error에 대해 무엇이 따라 나옵니까? (5문단에서 그 결론 문장을 찾으세요)
    - 3) 논문은 실패의 책임을 **모델 표현력**에 두는가 **solver**에 둡니까? 그 판정이 드러나는 단어를 원문에서 집으세요.
- **해석 노트**
    - 이 논증은 존재 증명일 뿐 도달 가능성 증명이 아니다 — "있다"와 "SGD가 찾는다"는 다릅니다.
    - 그 간극이 곧 논문의 문제 정의이고, [R04]의 F(x)=H(x)−x는 그 간극을 **좁히는 재매개화**이지 새 표현력을 주는 게 아닙니다.
    - identity를 학습으로 맞추는 대신 기본값으로 깔아 준다는 뜻.
- **자료 좌표**: [Q2] [A2]

---

## R04 H(x) 대신 F(x)=H(x)−x

- **논문 위치**: §3.1 Residual Learning (`sec:motivation`) 전체 3문단
- **읽기 전 질문**
    - 두 형태가 **표현력은 같다**면, 왜 굳이 바꿉니까?
    - 논문이 기대하는 이득은 표현력인가 다른 무엇입니까?
- **원문 핵심**: "we explicitly let these layers approximate a residual function $\mathcal{F}(\ve{x}):=\mathcal{H}(\ve{x})-\ve{x}$" / "Although both forms should be able to asymptotically approximate the desired functions (as hypothesized), the ease of learning might be different."
- **확인 질문**
    - 1) identity가 최적일 때, 잔차 형태에서 solver가 해야 할 일은 무엇입니까? 원래 형태에서는? (2문단 마지막 문장)
    - 2) 3문단에서 저자가 identity mapping의 최적성에 대해 취하는 입장은 무엇입니까? 그 입장을 담은 핵심 동사 하나를 원문에서 찾아 적으세요.
    - 3) 이 재정식화가 옳다는 **실험적 증거**로 논문이 가리키는 그림 번호는 무엇입니까? (그 그림은 §4.2에 있습니다 → [R14])
- **해석 노트**
    - §1의 구성적 해 논증([R03])이 남긴 간극을 메우는 자리입니다 — 논증은 §1, 처방은 §3.1, 증거는 §4.2 `fig:std`로 세 곳에 흩어져 있습니다.
    - 코드로는 `out = F(x) + x` 한 줄이지만 그 한 줄이 사는 이유는 이 셋을 이어야 보입니다.
    - 각주가 근사 가설 자체를 open question으로 인정하는 점도 놓치지 말 것.
- **자료 좌표**: [CS§0] [Q2] [A2] [N-11] [N-12] [N-13] [N-14] [N-15] [N-16] [NB 학습목표] [NB 0) 도입]

---

## R05 덧셈이 성립하려면 차원이 같아야 한다 (식1·식2)

- **논문 위치**: §3.2 Identity Mapping by Shortcuts — 식(1) `eq:identity`, 식(2) `eq:transform`
- **읽기 전 질문**
    - `F(x) + x`라는 덧셈이 성립하려면 두 항이 무엇을 공유해야 합니까?
    - 그 조건이 깨지는 순간은 네트워크의 **어디**입니까?
- **원문 핵심**: "The dimensions of $\ve{x}$ and $\mathcal{F}$ must be equal in Eqn.(1). If this is not the case (\eg, when changing the input/output channels), we can perform a linear projection $W_{s}$…"
- **확인 질문**
    - 1) 식(1)의 shortcut이 파라미터·계산량에 미치는 영향을 논문은 어떻게 서술합니까? 그 서술이 **비교 실험**에서 왜 중요합니까?
    - 2) 식(2)의 $W_s$는 언제 쓰이는가 — 항상인가, 특정 조건에서만입니까? 근거 문장을 찾으세요.
    - 3) $\mathcal{F}$가 **1층뿐**이면 식(1)은 무엇과 비슷해지고, 저자는 거기서 이득을 관찰했습니까?
- **해석 노트**
    - 조건이 깨지는 자리는 **stage 경계 딱 하나**다 — 채널이 2배, 맵이 절반 되는 지점([R16]).
    - 그 외 블록은 전부 identity로 충분합니다.
    - 그리고 conv에서 "차원이 같다"는 (C,H,W) 셋 다 같다는 뜻이지 채널만이 아니다 — 노트북 Option A 빈칸이 stride 슬라이싱과 zero-pad를 **둘 다** 요구하는 이유가 이것.
- **자료 좌표**: [C1-2] [C2] [C3-4] [CS§0] [CS§1] [A2] [A3] [NB 2) Option B] [N-5] [N-6] [N-10] [N-16] [N-7] [N-8] [N-9] [NB Option A/B 용어] [NB 0) 도입] [N-2] [N-3] [N-4]

---

## R06 Post-activation: 왜 ReLU가 덧셈 *뒤*인가

- **논문 위치**: §3.2 "We adopt the second nonlinearity after the addition", `fig:block`(Fig.2)
- **읽기 전 질문**
    - 블록 안에 ReLU가 두 번 나옵니다.
    - 두 번째 ReLU를 덧셈 **앞**에 두면 무엇이 달라집니까?
- **원문 핵심**: "The operation $\mathcal{F}+\ve{x}$ is performed by a shortcut connection and element-wise addition. We adopt the second nonlinearity after the addition (\ie, $\sigma(\ve{y})$, see Fig.2)."
- **확인 질문**
    - 1) Fig.2에서 ReLU는 총 몇 번 나오고, 각각 덧셈 기호 기준으로 앞인가 뒤입니까?
    - 2) §3.2는 Fig.2의 2층 블록에서 $\mathcal{F}$를 어떤 식으로 적습니까? 그 식 안에 $\sigma$가 몇 번 나오며, 그것은 Fig.2의 어느 ReLU에 해당합니까?
    - 3) 논문은 이 배치를 "adopt"한다고만 하는데, **왜** 그래야 하는지 §3.2에서 근거를 댑니까? (대지 않는다면 그 사실 자체가 답입니다)
- **해석 노트**
    - 논문은 근거 없이 채택만 합니다 — 이 선택이 논쟁이 되는 건 후속 v2 논문(He et al.
    - 2016, *Identity Mappings in Deep Residual Networks*)에서입니다.
    - **이 논문(v1)의 내용이 아닙니다.** 덧셈 뒤 ReLU를 빠뜨리면 학습은 돌아가고 성능만 조용히 떨어져 디버깅이 어렵습니다.
    - 블록 내 첫 ReLU는 conv1-BN1 뒤라 shortcut 가지에는 ReLU가 없습니다 — identity 경로가 선형 통로로 남는 이유.
- **자료 좌표**: [C3-3] [C3-4] [C3] [CS§2] [NB 3) BasicBlock] [N-14] [N-15] [N-17]

---

## R07 Plain net과 Residual net을 왜 그렇게 짝지었나

- **논문 위치**: §3.3 'Plain Network'·'Residual Network' 두 소제목, `fig:arch`(Fig.3 middle/right)
- **읽기 전 질문**: 두 네트워크를 비교해 "shortcut 덕분"이라 결론 내리려면, 두 모델이 shortcut 말고 무엇무엇을 공유해야 합니까?
- **원문 핵심**: "Based on the above plain network, we insert shortcut connections which turn the network into its counterpart residual version." / "The identity shortcuts can be directly used when the input and output are of the same dimensions (solid line shortcuts in Fig.3)."
- **확인 질문**
    - 1) Fig.3 middle과 right는 **몇 개의 weighted layer**를 갖습니까? FLOPs는 각각 얼마입니까? (caption에서 읽으세요)
    - 2) 'Residual Network' 소제목 첫 문장은 residual net이 plain net으로부터 어떻게 만들어졌다고 적습니까? 그 문장을 옮기세요.
    - 3) Fig.3 right에서 실선 shortcut과 점선 shortcut의 차이는 무엇입니까? 점선은 그림의 어느 위치에 몰려 있습니까?
- **해석 노트**
    - 대조군 설계가 이 실험의 전부입니다 — 파라미터·깊이·너비·연산량이 같아야 [R11]의 역전을 shortcut 하나로 돌릴 수 있다([R05]의 "no extra parameter"가 여기서 값을 합니다).
    - 설계 규칙 (i)(ii)는 이 항목이 아니라 [R16] 소유입니다.
    - 코드로는 plain/residual이 `use_shortcut` 플래그 하나 차이여야 공정한 비교가 됩니다.
- **자료 좌표**: [C3] [CS§2] [NB 3) BasicBlock] [N-7] [N-8] [N-9]

---

## R08 Option A/B/C — 세 가지 차원 맞추기와 그 비용

- **논문 위치**: §3.3 끝 (A·B **둘만** 정의) + §4.1 'Identity vs Projection Shortcuts' (**C는 여기서만 정의**), `tab:10crop`(Table 3)
- **읽기 전 질문**: 차원이 늘어나는 자리에서 shortcut을 맞추는 방법이 여럿이라면, 각각의 **비용**(파라미터·연산·학습 가능성)은 어떻게 다릅니까?
- **원문 핵심**: "(A) The shortcut still performs identity mapping, with extra zero entries padded for increasing dimensions. This option introduces no extra parameter; (B) The projection shortcut in Eqn.(2) is used to match dimensions (done by 1×1 convolutions)."
- **확인 질문**
    - 1) §3.3에서 저자가 고려한 옵션은 **몇 개**입니까? C는 어느 절에서 처음 정의됩니까? A/B/C 각각의 정의를 §4.1 문장 그대로 옮겨 적으세요.
    - 2) Table 3에서 세 옵션의 top-1 error를 읽어 채우세요: A=___% B=___% C=___%. B가 A보다 나은 이유로 저자가 든 문장은 무엇입니까?
    - 3) C가 가장 좋은데도 논문이 C를 **쓰지 않는** 이유 두 가지를 §4.1 마지막 두 문장에서 찾으세요. 그 이유가 [R12] bottleneck과 어떻게 연결됩니까?
- **해석 노트**
    - 함정: 논문의 결론은 "A/B/C 중 뭐가 낫냐"가 아니라 **차이가 작다**이고, 거기서 degradation의 원인이 shortcut 종류가 아님이 따라 나옵니다. A의 비용은 두 겹입니다. ① **논문이 말하는 것**(§4.1): zero-pad된 차원은 residual 학습을 하지 않습니다. ② **논문에 없는 것**: stride 슬라이싱 `x[:, :, ::2, ::2]`가 픽셀의 3/4를 버립니다 — 구현을 봐야 보이는 비용입니다.
    - CIFAR는 A로 고정([R15]).
- **자료 좌표**: [C1-1] [C1-2] [C1] [C2] [CS§1] [Q3] [A3] [A4] [A5] [NB 2) Option B] [N-5] [N-6] [NB Option A/B 용어] [NB 1) Option A] [N-1] [N-2] [N-3] [N-4]

---

## R09 구현 설정 — BN 위치·He init·ImageNet 스케줄

- **논문 위치**: §3.4 Implementation (`sec:impl`)
- **읽기 전 질문**
    - BN을 conv **뒤·activation 앞**에 두면 conv의 bias 항은 어떻게 됩니까?
    - 이 배치는 코드 한 줄에 어떻게 나타납니까?
- **원문 핵심**: "We adopt batch normalization (BN) right after each convolution and before activation" / "We initialize the weights as in [13] and train all plain/residual nets from scratch."
- **확인 질문**
    - 1) BN이 들어가는 위치를 §3.4의 원문 표현 그대로 적으세요. 그 표현대로면 conv·BN·activation 셋의 순서는 어떻게 됩니까?
    - 2) ImageNet 학습 설정을 채우세요: batch=___, 초기 lr=___, lr을 ÷10 하는 **조건**은 무엇인가(고정 iteration인가 다른 기준입니까?), 총 ___×10⁴ iterations, weight decay=___, momentum=___.
    - 3) 저자가 **쓰지 않는다**고 명시한 정규화 기법은 무엇이며, 왜입니까?
- **해석 노트**
    - BN이 conv 바로 뒤라 `nn.Conv2d(..., bias=False)`가 정답입니다 — BN의 β가 bias를 대신하므로 conv bias는 학습되되 효과 없이 낭비됩니다(논문은 이 코드 함의를 말하지 않습니다).
    - ÷10 조건이 **plateau 기준**인 점이 [R13] CIFAR의 32k/48k **고정 스텝**과 다릅니다.
    - "as in [13]"이 He 초기화(PyTorch `kaiming_normal_`)이고, 가중치를 **0 근처**로 두는 이 관행이 [R04]의 "residual을 0으로 미는 게 쉽다"는 논증의 숨은 전제입니다 — 그 연결은 논문이 말하지 않습니다(모범답안 `(2-a)` 부연).
- **자료 좌표**: [C3-1] [C3-2] [C3] [N-11] [N-12] [N-13] [N-14] [N-15] [NB 7) Train/Eval] [LG cell11]

---

## R10 degradation은 vanishing gradient가 *아니다*

- **논문 위치**: §4.1 'Plain Networks' 마지막 2문단 + 각주, `fig:imagenet`(Fig.4 left) — ImageNet 18층 vs 34층
- **읽기 전 질문**
    - "gradient가 사라져서 그렇다"는 가설을 **배제**하려면 어떤 관찰을 제시해야 합니까?
    - 배제 논증은 원인 규명과 무엇이 다릅니까?
- **원문 핵심**: "We argue that this optimization difficulty is *unlikely* to be caused by vanishing gradients. These plain networks are trained with BN, which ensures forward propagated signals to have non-zero variances."
- **확인 질문**
    - 1) 저자가 vanishing gradient를 배제하며 든 근거는 **몇 가지**이고 각각 forward인가 backward입니까? (해당 문장을 그대로 옮기세요)
    - 2) 34층 plain이 "solver가 어느 정도 작동한다"는 증거로 제시된 것은 무엇입니까?
    - 3) 각주는 어떤 추가 실험을 했고 결과가 어땠습니까? 그 실험이 배제하는 가설은 무엇입니까?
    - 4) 이 문단이 원인 규명으로 끝나는지 확인하세요 — 저자가 자기 설명에 붙인 동사를 그대로 적고, 이 절이 어떤 문장으로 끝나는지 보세요.
- **해석 노트**
    - [R02]와 짝이다: R02는 overfitting을, R10은 vanishing gradient를 배제합니다.
    - 둘 다 배제일 뿐 원인 규명이 아니고 논문은 future work로 남깁니다 — "ResNet이 gradient 소실을 해결했다"는 요약은 논문을 거꾸로 읽은 것입니다.
    - [R01]에서 gradient 문제를 치워둔 것이 이 배제의 전제.
- **자료 좌표**: [Q1] [A1]

---

## R11 Residual net의 결과

- **논문 위치**: §4.1 'Residual Networks' 3개 관찰, `tab:plain_vs_shortcut`(Table 2), `fig:imagenet`(Fig.4 right)
- **읽기 전 질문**: shortcut을 넣기 전후로 "18층 vs 34층"의 대소 관계가 어떻게 바뀌습니까? 파라미터는 얼마나 늘었습니까?
- **원문 핵심**: "the situation is reversed with residual learning -- the 34-layer ResNet is better than the 18-layer ResNet" / "So they have *no extra parameter* compared to the plain counterparts."
- **확인 질문**
    - 1) Table 2의 네 칸을 채우세요: plain-18=___ plain-34=___ ResNet-18=___ ResNet-34=___ (top-1 err %). 각 열에서 18층과 34층의 대소를 비교하고, 두 열의 결과를 나란히 적으세요.
    - 2) 이 실험에서 쓴 shortcut 옵션은 A/B/C 중 무엇입니까? 그 선택이 "no extra parameter" 주장에 왜 필수입니까?
    - 3) 18층에서는 plain과 ResNet의 최종 정확도가 비슷한데, 그럼 18층에서 ResNet의 이득은 무엇으로 나타납니까? (Fig.4 left vs right를 비교해 답하세요)
- **해석 노트**
    - 세 번째 관찰이 가장 자주 잘려 나간다: **얕으면 이득이 정확도가 아니라 수렴 속도로만 나타납니다.** 이것이 스터디 리그전(작은 모델·짧은 예산)에서 ResNet 효과가 기대만큼 안 보일 수 있는 이유이기도 합니다.
    - 그리고 "역전"이 성립하는 건 [R07]의 대조군 설계가 다른 변수를 모두 묶어 뒀기 때문입니다.
- **자료 좌표**: [NB 8) 실습]

---

## R12 Bottleneck과 더 깊은 모델

- **논문 위치**: §4.1 'Deeper Bottleneck Architectures' + 각주, `fig:block_deeper`(Fig.5), `tab:arch`(Table 1)
- **읽기 전 질문**
    - 층을 3배로 늘리면서 연산량을 비슷하게 유지하려면 무엇을 좁혀야 합니까?
    - 그 좁힘이 shortcut 쪽에는 어떤 제약을 만드습니까?
- **원문 핵심**: "The three layers are 1×1, 3×3, and 1×1 convolutions, where the 1×1 layers are responsible for reducing and then increasing (restoring) dimensions, leaving the 3×3 layer a bottleneck with smaller input/output dimensions."
- **확인 질문**
    - 1) Fig.5 right에서 세 conv의 채널 수를 읽어 채우세요: 1×1 ___ → 3×3 ___ → 1×1 ___. 이 세 값 사이에 어떤 관계가 보이습니까?
    - 2) bottleneck에서 identity shortcut을 projection으로 바꾸면 어떻게 되는지 논문이 정량적으로 말하는 문장을 찾아 그대로 옮기세요.
    - 3) Table 1에서 50층의 conv2_x~conv5_x 블록 반복 수를 읽으세요: [___, ___, ___, ___]. 101층·152층은 어느 stage에서 늘어나습니까?
    - 4) 각주는 bottleneck을 쓰는 이유를 무엇이라고 밝히습니까? 해당 표현을 그대로 적으세요.
- **해석 노트**
    - expansion=4가 논문 본문에 숫자로 적혀 있지 않습니다 — Fig.5의 64/64/256에서 **읽어내야** 나옵니다.
    - 구현에서 `planes`와 `planes*expansion`을 혼동하면 shortcut 채널이 어긋나 shape 에러가 나는데, 이 클래스의 버그는 "in_channels가 이전 블록의 **출력**(=planes*4)"이라는 계약을 놓쳐서 생긴다([R16]).
    - 또 identity가 여기서는 취향이 아니라 **비용 문제**입니다: 양 끝이 고차원이라 projection 하나가 블록 전체만큼 비싸집니다 — [R08]에서 C를 버린 결정이 여기서 값을 합니다.
    - 각주가 말하듯 bottleneck은 정확도가 아니라 **예산** 때문에 씁니다.
    - CIFAR 구현([R15])은 bottleneck을 쓰지 않으므로 이 항목은 Week 2 후반 확장 과제에 해당합니다.
- **자료 좌표**: [CS§3] [Q4] [A3] [A4] [NB 4) Bottleneck] [NB Appendix ImageNet] [LG cell9] [N-23] [N-25]

---

## R13 CIFAR-10 학습 설정 (논문 baseline)

- **논문 위치**: §4.2, 아키텍처 표 아래 학습 설정 문단 + 110층 warm-up 문단
- **읽기 전 질문**
    - ImageNet 설정([R09])과 CIFAR 설정 중 **무엇이 같고 무엇이 다른가**?
    - lr을 낮추는 기준이 두 곳에서 같습니까?
- **원문 핵심**: "We start with a learning rate of 0.1, divide it by 10 at 32k and 48k iterations, and terminate training at 64k iterations, which is determined on a 45k/5k train/val split."
- **확인 질문**
    - 1) 다음을 채우세요: batch=___, 초기 lr=___, ÷10 시점 ___k·___k iter, 종료 ___k iter, weight decay=___, momentum=___. 이 중 [R09] ImageNet과 **다른** 항목에 표시하세요.
    - 2) data augmentation은 무엇 무엇입니까? 테스트 시에는 몇 개의 view를 쓰습니까?
    - 3) 110층 문단을 읽으세요 — 이 깊이에서 학습 절차가 달라집니까? 달라진다면 무엇이 어떻게 바뀌고, 원래 스케줄로 돌아가는 기준은 무엇입니까?
- **해석 노트**
    - 이 설정은 **리그전의 근거가 아니라 리그전이 의도적으로 벗어난 baseline**입니다 — 아래 대조표를 볼 것.
    - 그리고 32k/48k는 CIFAR 값이고 ImageNet은 plateau 기준이라, 둘을 섞어 적는 것이 흔한 오답이다([R09]).
    - 110층 warm-up은 "깊으면 초반이 불안정하다"는 신호이지 residual이 안 통한다는 뜻이 아닙니다.
- **자료 좌표**: [NB 7) Train/Eval] [NB 8) 실습] [LG cell0] [LG cell11] [LG cell5]

**논문 baseline vs 3주차 리그전** — 리그전은 이 표에서 **의도적으로 벗어납니다**. *왜 벗어나도 되는지*가 3주차의 진짜 교육 포인트입니다.

| | 논문 §4.2 (R13) | Week 3 리그전 (cell 6, 12) |
| --- | --- | --- |
| 데이터 | 50k 전체, 45k/5k split | `TRAIN_SAMPLES_FIXED = 10_000` / `VALID_SAMPLES_FIXED = 2_000` |
| 예산 | 64k iterations | `EPOCHS_FIXED = 25` (epoch 기반) |
| 스케줄 | MultiStep, 32k/48k에서 ÷10 | `CosineAnnealingLR(T_max=EPOCHS_FIXED)` |
| batch | 128 | `BATCH_SIZE_FIXED = 128` (일치) |

생각할 거리: 25 epoch·10k 샘플이라는 예산 안에서 32k/48k MultiStep을 그대로 쓰면 **첫 감쇠 시점에 도달하기도 전에** 학습이 끝납니다.
리그전 TUNE 항목(`LR`, `WEIGHT_DECAY`, `MOMENTUM`, `GRAD_ACCUM`, optimizer/scheduler)이 논문의 어느 선택에 대응하는지 하나씩 짚어 보세요.

---

## R14 실험 결과 읽는 법

- **논문 위치**: §4.2 — `tab:cifar`(Table 6), `fig:cifar`(Fig.6), `fig:std`(Fig.7), 'Analysis of Layer Responses' + 'Exploring Over 1000 layers'
- **읽기 전 질문**
    - 표 한 줄에서 읽어야 할 축은 몇 개인가(깊이·파라미터·error)?
    - 어느 지점에서 "깊을수록 좋다"가 **깨지는가**?
- **원문 핵심**: "The testing result of this 1202-layer network is worse than that of our 110-layer network, although both have similar training error. We argue that this is because of overfitting."
- **확인 질문**
    - 1) Table 6에서 ResNet 계열만 뽑아 채우세요: 20층 ___M/___% · 56층 ___M/___% · 110층 ___M/___% · 1202층 ___M/___%. 깊이-error 곡선이 꺾이는 지점은 어디입니까?
    - 2) §4.2 마지막 문단에서 1202층의 training error와 test error를 각각 읽으세요: train=___ test=___%. 110층과 비교했을 때 두 지표가 각각 어떻게 다릅니까? 저자는 이 결과를 무엇으로 설명하습니까?
    - 3) Fig.7에서 ResNet과 plain 중 어느 쪽 response std가 더 작습니까? 20/56/110층을 비교하면 깊어질수록 std는 어떻게 됩니까? 이 관찰이 §3.1의 어떤 주장을 뒷받침하습니까?
    - 4) Fig.6 left에서 plain-110이 그래프에 없는 이유는 무엇입니까? (caption을 읽으세요)
- **해석 노트**
    - 1202층은 논문 **유일의 진짜 overfitting** 사례라 [R02]의 대조군이 됩니다 — "train은 잘 되는데 test만 나쁘다"가 있어야 Fig.1이 왜 overfitting이 아닌지 대조로 확정됩니다.
    - Fig.7은 §3.1([R04])의 "residual은 0에 가깝다" 가설을 닫는 고리입니다.
    - Week 4 로그 분석은 이 세 그림 읽는 법이 전부.
- **자료 좌표**: [NB 8) 실습] [LG cell13] [LG cell15]

---

## R15 6n+2 규칙 · Option A 고정 · small stem

- **논문 위치**: §4.2 아키텍처 문단 + 그 아래 요약 표(output map size / #layers / #filters)
- **읽기 전 질문**
    - 32×32 입력에 ImageNet stem(7×7 stride 2 + maxpool)을 그대로 쓰면 맵 크기가 어떻게 됩니까?
    - 왜 CIFAR는 다른 stem을 써야 합니까?
- **원문 핵심**: "The first layer is 3×3 convolutions. Then we use a stack of $6n$ layers with 3×3 convolutions on the feature maps of sizes {32,16,8} respectively, with $2n$ layers for each feature map size." / "There are totally $6n$+2 stacked weighted layers."
- **확인 질문**
    - 1) 6n+2에서 `6n`과 `+2`는 각각 무엇을 세습니까? 논문이 실험한 n 값 집합을 §4.2에서 모두 찾아 적고, 각각의 총 층수를 계산하세요.
    - 2) 표에서 읽어 채우세요 — feature map 크기 {___, ___, ___}, 필터 수 {___, ___, ___}, 각 크기의 층 수 ___. 세 stage의 층 수가 서로 같습니까? 다르다면 왜인지 stem을 세는 방식과 연결해 설명하세요.
    - 3) shortcut은 총 몇 개이며, 어느 옵션(A/B/C)으로 **고정**됩니까? 그 고정이 "plain과 파라미터 수가 정확히 같다"는 주장에 왜 필요합니까?
    - 4) 네트워크의 마지막 두 층은 무엇입니까? ([R17])
- **해석 노트**
    - 구현 시 자주 틀리는 곳 셋.
    - (a) `+2`는 stem conv 1층과 최종 FC 1층입니다 — GAP은 학습 파라미터가 없어 세지 않습니다.
    - 그래서 `(depth-2) % 6 == 0`이 유효성 검사가 되고, 노트북 cell 27이 정확히 이걸 묻습니다.
    - (b) stem이 3×3 stride 1 **단일 conv**이고 maxpool이 없습니다 — ImageNet stem을 복붙하면 32×32가 8×8로 시작해 stage 구조가 무너집니다.
    - (c) 각 stage는 2n층 = **n개 블록**(블록당 conv 2개)이라 `[n,n,n]`이 나옵니다 — 층 수와 블록 수를 혼동하면 정확히 2배가 어긋납니다.
    - 그리고 CIFAR가 Option A로 고정된 탓에 [R08]의 A/B/C 비교는 여기서 재현되지 않습니다 — 스터디 코드에 projection 경로가 있어도 CIFAR 기본 경로는 A입니다.
    - 논문 baseline([R13])의 shortcut 개수 3n도 이 구조에서 곧바로 나옵니다.
- **자료 좌표**: [C1] [C5-1] [C5] [CS§5] [Q5] [A5] [NB 5) ResNetV1] [N-18] [NB 6) CIFAR Factory] [NB 1) Option A] [N-1] [LG cell0] [LG cell7] [LG cell9] [N-28] [N-29] [N-30]

---

## R16 Stage 구조와 downsampling 좌표

- **논문 위치**: §3.3 'Plain Network'의 설계 규칙 (i)(ii) + §3.3 끝 "when the shortcuts go across feature maps of two sizes…" + `tab:arch`(Table 1) caption + §4.2 아키텍처 표
- **읽기 전 질문**
    - 맵 크기가 절반이 될 때 필터 수를 2배로 하면 층당 연산량은 어떻게 됩니까?
    - 그 규칙이 **어디서** 적용되는가 — 모든 블록인가 특정 블록입니까?
- **원문 핵심**: "(i) for the same output feature map size, the layers have the same number of filters; and (ii) if the feature map size is halved, the number of filters is doubled so as to preserve the time complexity per layer. We perform downsampling directly by convolutional layers that have a stride of 2."
- **확인 질문**
    - 1) §3.3의 설계 규칙 (i)과 (ii)를 각각 원문 그대로 옮기세요. (ii)에 붙은 목적절("so as to …")이 주장하는 바를 conv FLOPs 계산으로 직접 검산하세요.
    - 2) Table 1 caption은 downsampling이 어느 층에서 일어난다고 적습니까? 그 층 이름을 모두 옮기고, 이름의 숫자 부분이 stage 안에서 몇 번째 블록을 뜻하는지 판단하세요.
    - 3) shortcut이 두 맵 크기를 가로지를 때 stride는 얼마입니까? (§3.3 마지막 문장)
    - 4) CIFAR(§4.2)에서 stage는 몇 개이고 각 맵 크기·필터 수는 무엇입니까? ([R15])
- **해석 노트**
    - 규칙 (ii)는 곧 **stage 경계 계약**이다: `stride=2`와 `channels×2`가 항상 같은 블록에서 함께 일어나고 그 블록이 stage의 **첫** 블록이다(나머지는 stride 1·채널 유지).
    - 함정 둘 — 모든 블록에 stride 2를 주면 맵이 GAP 전에 무너지고, 루프에서 `in_channels`를 갱신하지 않으면 다음 블록에서 shape mismatch가 납니다.
- **자료 좌표**: [C5-2] [C5] [CS§4] [A5] [NB 5) ResNetV1] [N-19] [N-20] [N-21] [NB Appendix ImageNet] [LG cell7] [N-23] [N-24] [N-25] [N-26] [N-27]

---

## R17 출력단: GAP + FC

- **논문 위치**: §3.3 "The network ends with a global average pooling layer and a 1000-way fully-connected layer with softmax." + §4.2 "ends with a global average pooling, a 10-way fully-connected layer, and softmax" + `tab:arch`(Table 1) 마지막 두 행
- **읽기 전 질문**
    - VGG는 마지막에 FC를 여러 층 쌓습니다.
    - 그 자리를 GAP 하나로 바꾸면 파라미터 수와 입력 크기 의존성은 어떻게 달라집니까?
- **원문 핵심**: "The network ends with a global average pooling layer and a 1000-way fully-connected layer with softmax."
- **확인 질문**
    - 1) Table 1 마지막 두 행에서 GAP 직전 output size와 GAP 직후 output size를 읽으세요: ___×___ → ___×___. FC의 출력 차원은 ImageNet에서 ___, CIFAR에서 ___입니까?
    - 2) GAP은 학습 파라미터를 몇 개 갖습니까? 이 답이 [R15]의 `6n+2`에서 GAP이 세어지지 않는 이유와 어떻게 연결됩니까?
    - 3) §3.3은 34층 baseline과 VGG-19의 FLOPs를 각각 얼마로 적습니까? ___ vs ___ (billion). 두 모델의 마지막 단(FC 구성) 차이가 이 격차에 어떻게 기여하습니까?
- **해석 노트**
    - 코드로는 `nn.AdaptiveAvgPool2d(1)` → `flatten` → `nn.Linear(C, num_classes)`.
    - GAP의 값은 파라미터 절감이 아니라 **위치 불변성을 구조로 강제**하는 데 있습니다 — 공간 위치를 평균으로 뭉개 버리므로 FC가 위치에 과적합할 여지 자체가 사라진다(그래서 `view(-1, C*H*W)`로 바꾸면 입력 크기 변화에 깨질 뿐 아니라 이 규제도 함께 잃습니다).
    - `Linear`의 in_features는 마지막 stage 채널(CIFAR 64, bottleneck은 planes×4 → [R12]), softmax는 `CrossEntropyLoss`가 처리하니 모델에 넣지 않습니다.
- **자료 좌표**: [C5-3] [C5] [CS§6] [A5] [NB 5) ResNetV1] [N-22] [LG cell7]

---

## 부록 A — 논문 → R 커버리지 (V3)

| 논문 절 | 덮는 R |
| --- | --- |
| §1 Introduction | R01 R02 R03 |
| §3.1 Residual Learning | R04 |
| §3.2 Identity Mapping by Shortcuts | R05 R06 |
| §3.3 Network Architectures | R07 R08 R16 R17 |
| §3.4 Implementation | R09 |
| §4.1 ImageNet Classification | R08 R10 R11 R12 |
| §4.2 CIFAR-10 and Analysis | R13 R14 R15 |
| §2 / §4.3 / Appendix | **범위 밖** |

## 부록 B — 자료 → R 매핑 (V7)

각 자료의 모든 절이 최소 1개 R에 매핑됩니다. Phase 2에서 이 표의 역방향(`자료 좌표` 칸)을 채웁니다.

| 자료 절 | 담당 R |
| --- | --- |
| 치트시트 §0 Residual Learning | R04 R05 |
| 치트시트 §1 Shortcut 연결 방식 (Option A/B) | R05 R08 |
| 치트시트 §2 Basic Residual Block | R07 R06 |
| 치트시트 §3 Bottleneck Block | R12 |
| 치트시트 §4 Stage 구조와 Downsampling | R16 |
| 치트시트 §5 CIFAR-10 코드에서 다르게 구현한 부분 | R15 |
| 치트시트 §6 Global Average Pooling | R17 |
| 쿡북 `[C0]` 먼저 읽기 — nn.Module | (구현 준비, R 없음) |
| 쿡북 `[C1]` Shortcut Option A (Slicing / Zero Padding) | R08 R15 |
| 쿡북 `[C2]` Shortcut Option B (Projection) | R05 R08 |
| 쿡북 `[C3]` BasicBlockV1 (Conv / BN / ReLU / ShortCut) | R06 R07 R09 |
| 쿡북 `[C4]` BottleneckV1 | R12 |
| 쿡북 `[C5-1]` Model Stem | R15 |
| 쿡북 `[C5-2]` Layer Stacking | R16 |
| 쿡북 `[C5-3]` Final Classification | R17 |
| 쿡북 `[C6]` CIFAR Factory (depth = 6n+2) | R15 |
| 퀴즈 `[Q1]` degradation vs overfitting | R02 R10 |
| 퀴즈 `[Q2]` H(x) → F(x) | R03 R04 |
| 퀴즈 `[Q3]` Option A/B/C, C를 버린 이유 | R08 |
| 퀴즈 `[Q4]` Bottleneck, identity가 중요한 이유 | R12 |
| 퀴즈 `[Q5]` 6n+2, identity 고정 | R15 |
| 모범답안 `[A1]`~`[A5]` | 위 Q1~Q5와 동일 |
| 노트북 `[N-1]`~`[N-4]` ShortcutZeroPad | R05 R08 R15 |
| 노트북 `[N-5]` `[N-6]` ShortcutProjection | R05 R08 R09 |
| 노트북 `[N-7]`~`[N-17]` BasicBlockV1 | R04 R05 R06 R07 |
| 노트북 `[N-18]`~`[N-21]` stage 채널·stride | R16 |
| 노트북 `[N-22]` GAP+FC | R17 |
| 노트북 `[N-23]`~`[N-27]` _make_layer | R12 R16 |
| 노트북 `[N-28]`~`[N-30]` resnet_cifar (6n+2) | R15 |
| 노트북 train/eval + degradation 재현 셀 | R02 R09 R10 R13 |
| 리그전 FIXED/TUNE | R13 R14 R15 |

> R01은 유일하게 자료 참조가 없습니다 — 동기 전용 항목이라 정상입니다.

> 이 표는 **현재 번호 기준**입니다 (2026-08-27 갱신). 쿡북은 `[C0]`~`[C6]`, 노트북은 `[N-1]`~`[N-30]`,
> 퀴즈는 `[Q1]`~`[Q5]`, 모범답안은 `[A1]`~`[A5]`가 실제로 문서에 박혀 있는 앵커입니다.

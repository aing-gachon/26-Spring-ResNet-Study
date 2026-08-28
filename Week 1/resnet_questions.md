# ResNet (He et al., 2016) 질문 정리 (Questions Only)

각 문항의 `> 허브:` 표기는 [논문 가이드](resnet_paper_guide.md)의 R 항목을 가리킵니다. 답은 [퀴즈_모범답안](resnet_questions_sample_answer.md).

각 문항 머리에 `> 근거:`로 **개념은 [치트시트](A.ing_resnet_cheat_sheet.md) `[CS§n]`, 구현은 [쿡북](../Week%202/resnet_cookbook.md) `[Cn-m]`, 코드는 노트북 `[N-k]`** 좌표를 함께 답니다. 막히면 그 좌표부터 다시 읽으시면 됩니다.


[source: sources/resnet-study/Week 1/resnet_questions.md]

> *Deep Residual Learning for Image Recognition* 기반 문제 정리  
> (모범답안 제외 버전)

**난이도 표기** — 소문항마다 붙어 있습니다.

| 태그 | 뜻 | 필요한 것 |
| --- | --- | --- |
| `[기본]` | 논문을 읽었으면 답할 수 있습니다 | 해당 절·그림 |
| `[응용]` | 논문 + 코드 연결이 필요합니다 | 허브 R + 쿡북/노트북 좌표 |
| `[심화]` | 논문에 직접 안 적힌 추론 | 논증을 스스로 구성 |

`[심화]`는 못 풀어도 진도에 지장 없습니다. 먼저 `[기본]`을 전부 채우고 넘어갈 것.

---

## 1) Degradation 문제는 Overfitting과 어떻게 다릅니까?  `[Q1]`

> 허브: [R02] [R10]
> 근거: [CS§0] · 노트북 마지막 degradation 재현 셀

논문은 '깊어질수록 성능이 나빠지는 현상(degradation)'이 overfitting이 아니라 **최적화(학습) 문제**라고 주장합니다.

- (1) `[기본]` Figure 1을 근거로 degradation과 overfitting을 구분하시오.
- (2) `[응용]` 왜 degradation이 '학습이 안 되는 문제(optimization issue)'인지 설명하시오.

---

## 2) H(x) 대신 F(x)=H(x)−x를 학습하면 왜 더 쉽습니까?  `[Q2]`

> 허브: [R03] [R04]
> 근거: [CS§0] (특히 "왜 쉬운 해인가" 항목) · [C3-2]

Section 3.1에서 저자들은 원하는 mapping을 H(x)라 두고, 잔차를 F(x)=H(x)−x로 재정의하여  
$y = F(x) + x$ 형태로 학습합니다.

(1) `[기본]` 최적의 해가 identity mapping(H(x)=x)일 때, 왜 F(x)=0으로 학습하는 것이 더 쉬운지 설명하시오.  

(2) `[심화]` 이 논증이 degradation 문제와 어떻게 연결되는지 설명하시오.

> `[심화]` 더 읽을거리: 모범답안의 `(2-a)` 부연 — 왜 F=0이 H=I보다 가까운가(He 초기화 [R09]와 연결).

---

## 3) Shortcut Option A/B/C는 무엇이며, 논문은 왜 Option C를 버렸습니까?  `[Q3]`

> 허브: [R08]
> 근거: [CS§1] · [C1-1] [C1-2] [C2-1] · [N-1]~[N-6]

차원이 유지될 때는 identity shortcut $y = F(x) + x$를 쓸 수 있지만,  
차원이 증가하거나 spatial downsampling이 발생하면 shortcut 설계가 달라집니다.

(1) `[응용]` 논문이 말하는 Option A/B/C를 각각 설명하시오.  
또한 Option B에서 $y = F(x) + W_s x$가 의미하는 바를 설명하시오.

(2) `[응용]` Table 3 결과와 저자 논의를 근거로,  
왜 논문은 Option C를 이후 실험에서 사용하지 않겠다고 결론 내렸는지 설명하시오.

---

## 4) Bottleneck block은 왜 필요하며, 왜 identity shortcut이 특히 중요해집니까?  `[Q4]`

> 허브: [R12]
> 근거: [CS§3] · [C4-4]

ResNet-34는 basic block(3×3, 3×3)을 쓰지만,  
ResNet-50/101/152는 bottleneck block(1×1, 3×3, 1×1)을 사용합니다.

(1) `[응용]` bottleneck에서 두 개의 1×1 convolution은 각각 어떤 역할을 합니까?  

(2) `[심화]` 논문은 bottleneck에서 identity shortcut을 projection으로 바꾸면  
time complexity와 model size가 크게 증가한다고 주장합니다.  
그 이유를 설명하시오.

---

## 5) CIFAR-10에서 6n+2 규칙은 어떻게 나오며, 왜 shortcut을 identity로 고정했습니까?  `[Q5]`

> 허브: [R15]
> 근거: [CS§5] · [C5-1] [C5-2] · [N-18]~[N-23]

논문 4.2(CIFAR-10)에서는 '6n+2 weighted layers' 규칙의 단순 구조를 사용합니다.

(1) `[기본]` 네트워크 구성을 기반으로 6n+2가 되는 과정을 유도하시오.  

(2) `[응용]` shortcut이 3n개가 되는 이유를 설명하시오.  

(3) `[응용]` CIFAR-10 실험에서 왜 모든 shortcut을 identity(option A)로 고정했는지  
논문 의도를 설명하시오.

---

## 참고 문헌

- Kaiming He, Xiangyu Zhang, Shaoqing Ren, Jian Sun.  
  *Deep Residual Learning for Image Recognition.* CVPR 2016.

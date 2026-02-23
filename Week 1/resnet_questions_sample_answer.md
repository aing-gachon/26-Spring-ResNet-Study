# ResNet (He et al., 2016) 핵심 Q&A 정리



## 1) Degradation 문제는 Overfitting과 어떻게 다른가?

### 핵심 구분

- **Overfitting(과적합)**: 보통 **training error는 계속 내려가는데**, **test error(또는 generalization)가 악화**되는 현상.
- **Degradation(성능 저하)**: 깊이를 늘릴수록 **training error 자체가 더 높아지는** 현상.  
  즉, 데이터에 과적합되기 전에 **학습(최적화) 자체가 잘 안 되는 최적화 실패**를 의미한다.

### 왜 “학습이 안 되는 문제(optimization issue)”인가?

- 깊은 네트워크는 **추가된 층이 최소한 identity mapping을 학습**할 수 있다면,
  얕은 네트워크가 구현 가능한 함수를 **그대로 포함**할 수 있어야 한다.
- 그럼에도 불구하고 더 깊은 *plain network*가 더 높은 training error를 보인다면,
  이는 **모델의 표현력 부족**이 아니라 **optimizer가 그 ‘쉬운 해(= identity 포함)’를 못 찾는 문제**로 해석하는 것이 자연스럽다.
- 논문 Figure 1에서 **56-layer plain**이 **20-layer plain**보다 **training error가 더 높게 유지**되는 것이
  degradation이 overfitting이 아니라 **최적화 문제**라는 핵심 근거다.

---

## 2) H(x) 대신 F(x)=H(x)−x를 학습하면 왜 더 쉬운가?

논문은 원하는 mapping을 `H(x)`라 두고, residual을 `F(x)=H(x)−x`로 재정의하여:

- **출력**: `y = F(x) + x`

로 학습한다.

### (1) 최적의 해가 identity mapping(H(x)=x)일 때, 왜 F(x)=0이 더 쉬운가?

- `H(x)=x`가 최적이라면, plain net은 여러 층(비선형 포함)이 **정교하게 항등함수(identity)**를 구현해야 한다.
- 반면 ResNet에서는 목표가 **“정답을 새로 만들기”가 아니라**  
  **“변화분(residual)이 0이면 된다”**, 즉 `F(x)=0`으로 바뀐다.
- 최적화 관점에서 이는 **문제의 재파라미터화(re-parameterization)**로 볼 수 있다.
  - identity에 가까운 해를 찾기 위해 굳이 복잡한 층들을 맞추기보다,
  - **잔차를 0으로 수렴시키는 방향**(가중치를 0 근처로 몰아가는 등)이 더 “쉽게” 작동할 수 있다.

### (2) 이 논증이 degradation 문제와 어떻게 연결되는가?

- degradation은 “깊은 plain net이 (적어도 일부 구간에서) 사실상 identity를 구현하면 되는 상황에서도
  그 해를 잘 못 찾는 최적화 실패”로 해석할 수 있다.
- residual 재정의(`H→F`)는 이 상황에서 해 공간을 **identity 근방으로 정렬**해 주는 효과가 있어,
  optimizer가 **‘학습하기 쉬운 형태’**로 문제를 풀게 만들며 degradation을 완화한다.

---

## 3) Shortcut Option A/B/C는 무엇이며, 논문은 왜 Option C를 버렸는가?

차원이 유지될 때는 identity shortcut:

- `y = F(x) + x`

를 그대로 쓸 수 있다.  
하지만 **채널 수 증가 또는 spatial downsampling(stride=2 등)**이 발생하면 shortcut 설계가 달라진다.

### (1) Option A/B/C 정의

- **Option A (parameter-free)**
  - shortcut은 **identity 계열**을 유지한다.
  - 차원(채널)이 증가할 때는 **zero-padding으로 채널을 맞춘 뒤 더한다**(추가 파라미터 없음).
  - spatial downsampling이 필요하면 identity branch에서도 **간단한 downsampling(예: stride에 맞춘 subsampling)**으로 해상도를 맞춘다.
- **Option B (projection only when needed)**
  - 차원이 바뀌는 구간에서만 shortcut에 projection을 적용한다.
  - 보통 `W_s`는 **1×1 conv**이며, 필요하면 **stride**로 해상도도 맞춘다.
  - 수식: `y = F(x) + W_s x`
- **Option C (projection everywhere)**
  - 모든 shortcut을 projection(`W_s`)으로 만든다.
  - 파라미터/연산량이 증가한다.

### (2) 왜 Option C를 이후 실험에서 쓰지 않는가?

- Table 3에서 A/B/C 모두 plain 대비 큰 개선을 보이며, B가 A보다 약간 낫고 C가 B보다 아주 약간 더 낫다.
- 하지만 논문은 C의 추가 이득이
  - “projection shortcut이 많아져서 생긴 **추가 파라미터(용량) 효과**”로 해석 가능하며,
  - degradation 해결에 projection이 “항상 필수”라고 보기 어렵고,
  - 개선 폭도 크지 않다고 정리한다.
- 따라서 이후 실험에서는 **불필요한 복잡도(시간/메모리/파라미터)를 줄이기 위해**
  **Option C를 사용하지 않겠다**는 결론으로 이어진다.

---

## 4) Bottleneck block은 왜 필요하며, 왜 identity shortcut이 특히 중요해지는가?

ResNet-34는 basic block(3×3, 3×3)을 쓰지만,
ResNet-50/101/152는 bottleneck block(1×1, 3×3, 1×1)을 사용한다.

### (1) bottleneck에서 1×1 conv 두 개의 역할

- 첫 번째 1×1 conv: **채널 차원 축소(reduce)**  
  → 비싼 3×3 conv를 “얇은 채널”에서 수행하게 하여 연산량을 줄인다.
- 마지막 1×1 conv: **채널 차원 복원/확장(restore/expand)**  
  → 블록 출력의 채널 수를 다음 stage 설계(표현력 요구)에 맞게 되돌린다.

### (2) “identity shortcut이 projection으로 바뀌면 복잡도가 2배”가 되는 이유

- bottleneck의 핵심 의도는 “가운데(3×3)를 얇게 만들어 비용을 절감”하는 것이다.
- 그런데 shortcut까지 projection(학습 파라미터가 있는 1×1 conv)이 되면,
  shortcut 경로에서도 **고차원(high-dimensional ends)**에서 추가 연산/파라미터가 들어간다.
- 즉, bottleneck이 절감하려던 비용과 비슷한 급의 비용이 shortcut 경로에 추가되어
  time complexity와 model size가 크게 증가(논문 표현: doubled)할 수 있다.
- 반대로 identity shortcut은 **파라미터 0**, 연산도 최소이므로 bottleneck의 효율성을 해치지 않는다.

---

## 5) CIFAR-10에서 6n+2 규칙은 어떻게 나오며, 왜 shortcut을 identity로 고정했는가?

논문 4.2(CIFAR-10)는 intentionally simple한 구조로 “6n+2 weighted layers” 규칙을 사용한다.

### (1) 6n+2 유도

CIFAR-10용 ResNet은 다음으로 구성된다.

- **초기 3×3 conv 1개**
- 해상도(stage) 3개: `{32×32, 16×16, 8×8}`
  - 각 stage에 residual unit이 **n개**
  - unit 하나당 3×3 conv가 **2개**
  - 따라서 stage당 **2n개**, 전체 **6n개**의 3×3 conv
- 마지막에 **global average pooling + FC 1개(분류기)**

→ weighted layers 기준으로

- `1 (초기 conv) + 6n (3×3 conv들) + 1 (FC)`  
= **6n + 2**

### (2) shortcut이 3n개가 되는 이유

- shortcut은 residual unit마다 1개씩 붙는다.
- residual unit은 “3×3 conv 두 개가 한 쌍”인 구조이며,
- stage가 3개이고 stage마다 unit이 n개이므로:

- `3 stages × n units = 3n shortcuts`

### (3) CIFAR-10에서는 왜 모든 shortcut을 identity(option A)로 고정했는가?

- 논문은 CIFAR-10에서 **extremely deep network의 behavior를 분석**하는 목적이 강하다.
- residual vs plain 비교에서 변수를 최소화하려면,
  shortcut에 projection을 넣어 **파라미터를 추가**하는 요인을 가능한 한 배제하는 것이 유리하다.
- 따라서 **추가 파라미터가 없는 identity shortcut(Option A)**을 고정하여
  “구조적 잔차학습(residual learning)의 효과” 자체를 더 순수하게 관찰하려는 의도가 있다.

---

- Kaiming He, Xiangyu Zhang, Shaoqing Ren, Jian Sun. *Deep Residual Learning for Image Recognition.* CVPR 2016.

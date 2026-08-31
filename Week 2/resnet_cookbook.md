# [A.ing](http://A.ing) ResNet CookBook

각 절의 `> 허브:` 표기는 [논문 가이드](../Week%201/resnet_paper_guide.md)의 R 항목을 가리킵니다. 개념 요약은 [치트시트](../Week%201/A.ing_resnet_cheat_sheet.md), 자기점검은 [퀴즈](../Week%201/resnet_questions.md) 참고.

각 절 머리의 `> 대응:` 표기는 그 절이 담당하는 **치트시트 절 `[CS§n]`과 노트북 빈칸 `[N-k]`**입니다. 노트북에서 막히면 그 빈칸 번호로 여기 절을 찾으시면 됩니다.


[source: sources/resnet-study/Week 2/resnet_cookbook.md]

---

## [0] 먼저 읽기 — `nn.Module`  `[C0]`

빈칸 30개는 전부 아래와 같은 **클래스 안**에 있습니다. ResNet이 아닌 가장 단순한 블록으로 틀만 봅시다. 지금은 Conv2d, BatchNorm2d, ReLU와 같은 내용보다는 큰 구조에 집중해 주시면 됩니다. 세 모듈 모두 아래에서 더 자세히 다룹니다.

```python
import torch, torch.nn as nn

class TinyBlock(nn.Module):
    def __init__(self, c):          # ① 부품을 "만들어 두는" 곳
        super().__init__()          #    반드시 첫 줄. 빠뜨리면 에러가 납니다
        self.mix  = nn.Conv2d(c, c, kernel_size=3, padding=1, bias=False)
        self.norm = nn.BatchNorm2d(c)
        self.act  = nn.ReLU(inplace=True)

    def forward(self, x):           # ② 만들어 둔 부품을 "쓰는" 곳
        h = self.mix(x)             #    self.mix 가 함수처럼 호출됩니다
        h = self.norm(h)
        h = self.act(h)
        return h
```

실행하면:

```python
blk = TinyBlock(16)
y = blk(torch.randn(2, 16, 32, 32))   # blk(...) 는 forward 를 부릅니다
print(y.shape)                        # torch.Size([2, 16, 32, 32])
```

### 여기서 가져갈 것 셋

**(1) `__init__`은 만들고, `forward`는 씁니다.**
`self.mix = nn.Conv2d(...)`로 **저장해 두면**, `forward`에서 `self.mix(x)`로 **부릅니다**.
저장할 때는 `= nn.Conv2d(...)`(괄호에 설정값), 부를 때는 `self.mix(x)`(괄호에 데이터). 이 둘이 다릅니다.

**(2) 저장한 이름 그대로 부릅니다.**
`self.bn1`로 저장했으면 그 이름 그대로 불러야 합니다 — 다른 이름으로 부르면 `AttributeError`가 납니다. 노트북에는 `conv1`/`conv2`,
`bn1`/`bn2`처럼 **번호가 붙은** 이름이 있으니, 빈칸 위쪽 `__init__`에서 **실제로 뭐라고 저장했는지 확인**하고 씁니다.

**(3) 결과 변수를 계속 덮어씁니다.**
`h = self.mix(x)` 다음 `h = self.norm(h)` — 앞 결과를 다음 입력으로 넘깁니다.
이 연결이 끊기면(둘째 줄에 `h` 대신 `x`를 넣으면) 조용히 틀린 값이 나오고 **에러는 안 납니다**. 결과는 이상한데 명시적인 오류가 없어 디버깅이 더 힘들어질 수 있습니다.
노트북에서는 이 변수 이름이 `out`입니다.

### 자주 나오는 모양 셋

| 모양                           | 언제                | 왜                                                |
| ---------------------------- | ----------------- | ------------------------------------------------ |
| `nn.Conv2d(..., bias=False)` | BN이 바로 뒤에 올 때     | BN이 평균을 빼므로 bias가 무의미해집니다 → `[C3-2]`              |
| `nn.Identity()`              | "아무것도 안 함"이 필요할 때 | `if`로 분기하지 않고 **똑같이 호출**하려고 → `[C3-5]`           |
| `nn.Sequential(*layers)`     | 층 리스트를 하나로 묶을 때   | `*`는 리스트를 인자들로 펼칩니다(Python Unpacking) → `[C5-2]` |

> **`nn.Identity()`가 왜 필요한가**: shortcut이 어떤 때는 그냥 통과, 어떤 때는 변환입니다.
> 통과일 때 `None`을 넣으면 `forward`에서 매번 `if`를 써야 합니다. 대신 "아무것도 안 하는 모듈"을
> 넣어 두면 **두 경우 모두 한 줄로 끝납니다.** 분기를 `__init__` 쪽에 몰아넣는 것입니다.

---

## **[1) Shortcut Option A (Identity + Zero Padding)]**  `[C1]`

> 허브: [R08] [R15]
> 대응: [CS§1] · [N-1]~[N-4] · [Q3]

### 1. Slicing  `[C1-1]`

> 허브: [R08]

- **사용 함수**: Python Slicing Syntax (`::step`)
- (내 추가) 전제1: ResNet에서 다룰 Tensor의 형태는 (개수, 채널, 행, 열) 입니다. 개수는 한 배치당 학습하는 이미지의 개수, 채널은 이미지의 채널 수(컬러 이미지는 RGB로 채널이 3, 흑백이미지는 채널이 1입니다.), 행은 이미지의 세로 해상도, 열은 이미지의 가로 해상도입니다. 예를 들어 (5, 3, 32, 32)는 32x32 컬러 이미지 5개를 한 배치에서 학습한다는 의미입니다.
- **패턴**: `x[:, :, ::stride, ::stride]` (Height(행)와 Width(열) 차원만 샘플링)
- **설명**: ResNet Identity Mapping 시, 메인 경로에서 Stride가 적용되어 이미지의 행과 열이 줄어듭니다. 이렇게 Feature Map의 크기가 줄어든 경우(=이미지의 행과 열이 줄어든 경우) F(x) + x 연산이 불가능해집니다. 이를 해결하기 위해 입력값 x 에서도 동일한 간격으로 데이터를 추출하여 파라미터 없이 해상도를 맞춥니다.
- **주의 사항**
    - 높이와 너비 차원에 대해 Stride 값만큼 건너뛰며 데이터를 추출하세요.
    - 배치와 채널 차원은 건드리지 않아야 합니다.
- **Shape 흐름**
    - **Input:** `[128, 16, 32, 32]`
    - **Slicing (::2):** 가로/세로를 stride 간격으로 점프하며 선택
    - **Output:** `[128, 32, 16, 16]` 행과 열만 정확히 절반

### 2. Zero Padding  `[C1-2]`

> 허브: [R08] [R05]

다운샘플링된 텐서의 채널 수가 목표 출력 채널 수보다 적을 때, 0을 채워 차원을 확장하는 과정입니다.

- **코드**
    - `torch.zeros()` : 0으로 채워진 텐서 생성
    - `torch.cat()` : 텐서 연결
    - `tensor.size()` : 텐서 크기 확인
- **패턴**
    - `torch.zeros(batch, ch, h, w)` : 지정된 크기의 0 텐서 생성 (device, dtype 일치 필수)
    - `torch.cat([A, B], dim=1)` : 채널 축(dim=1)을 기준으로 연결
- 설명: ResNet Identity Mapping 시, 메인 경로에서 Convolution 연산 적용되어 Feature Map의 채널 수가 늘어난 경우 F(x) + x 연산이 불가능합니다. 이를 해결하기 위해(=부족한 채널 수를 늘리기 위해) 입력값 x 에서 부족한 채널 수만큼 0 행렬을 만들어 채웁니다.
- **코드 사용법**
    - **필요 채널 계산:** (목표 출력 채널) - (현재 입력 채널)을 구하세요.
    - **0 생성:** 입력 텐서와 동일한 공간 크기를 가지면서, 위에서 계산한 부족한 채널만큼의 깊이를 가진 0 텐서를 만듭니다.
    - **결합:** 입력 텐서 뒤에 0 텐서를 이어 붙입니다.
- **Shape 흐름**
    - **Input:** `[128, 32, 16, 16]` (Step 1 결과)
    - **Zeros:** `[128, 32, 16, 16]` (새로 만든 0)
    - **Output:** `[128, 32, 16, 16]`

---

## **[2) Shortcut Option B  (Projection)]**  `[C2]`

> 대응: [CS§1] · [N-5] [N-6] · [Q3]

> 허브: [R05] [R08]

### 1. Projection Shortcut  `[C2-1]`

1x1 합성곱을 사용하여 해상도 감소와 채널 확장을 동시에 수행하는 학습 가능한 숏컷 방식입니다.

- **코드**
    - `nn.Conv2d()` : 합성곱 연산
    - `nn.BatchNorm2d()` : 배치 정규화
- **패턴**
    - `nn.Conv2d(in, out, kernel_size=1, ...)` : 픽셀 간 정보 교환 없이 채널만 변경
- 설명: Option A가 단순히 빈자리를 0으로 채우는 수동적인 방식이라면, Option B는 1×1 Convolution의 가중치를 통해 어떤 정보를 Shortcut로 보낼지 모델이 스스로 결정하는 능동적인 방식입니다. 이는 차원이 불일치하는 지점에서도 정보의 손실을 최소화하고 학습의 유연성을 높여줍니다.
- **코드 사용법**
    - 커널 크기는 1로 고정하여 채널 간 연산만 수행합니다.
    - 공간 해상도를 줄이기 위해 메인 경로와 동일한 `stride`를 적용하세요.
    - 합성곱 뒤에는 반드시 `BatchNorm`을 연결해야 분포가 깨지지 않습니다.
- **Shape 흐름**
    - **Input:** `[128, 16, 32, 32]`
    - **Conv 1x1 (s=2):** 해상도 ½, 채널 2배
    - **Output:** `[128, 32, 16, 16]`

---

## **[3) BasicBlockV1]**  `[C3]`

> 대응: [CS§0] [CS§2] · [N-7]~[N-17] · [Q2]

> 허브: [R07] [R09] [R06]

블록 안에 Conv2d · BatchNorm2d · ReLU 세 개가 늘 붙어 다닙니다. 개별 API는 아래 절에서 보고, 여기서는 **왜 이 셋이 세트인지**를 ReLU → BatchNorm → Conv2d 순으로 먼저 잡고 갑니다.

**① ReLU — 비선형성을 얻는 대가로 정보를 버린다**

`ReLU(x) = max(0, x)`

```
        y
        ^
      3 |                    /
      2 |                  /
      1 |                /
      0 |______________/
        +------------------------> x
       -3   -2   -1   0    1   2   3

  x <  0 : y = 0   (기울기 0, 평평)
  x >= 0 : y = x   (기울기 1, 45도 직선)
```

- **음수 구간(x < 0)**: 전부 0으로 눌립니다. 0에서 꺾이기 때문에 **비선형성은 여기서 나오지만**, -0.1과 -50이 똑같은 0이 되므로 **정보 손실**이 생깁니다. gradient도 0이라 그 샘플에서는 그 뉴런이 학습되지 않습니다.
- **양수 구간(x >= 0)**: `y = x`, 즉 항등함수입니다. 값이 그대로 통과하니 **정보는 온전히 보존되지만**, 이 구간만 쓰이면 선형 함수와 다를 게 없어 **비선형성을 학습하지 못합니다**.
- 즉 ReLU는 **정보 보존 ↔ 비선형성**의 trade-off이고, 그 trade-off가 어디에 놓이는지는 "입력이 0을 기준으로 어디에 분포하는가"가 정합니다.

**② BatchNorm — 그 trade-off를 돌리는 다이얼**

ReLU의 위 단점을 보완하려면 결국 **입력 분포의 위치를 조절**해야 합니다. 그게 BatchNorm이 앞에 붙는 이유입니다. 입력의 평균과 분산을 조절해서 분포를 왼쪽으로 밀면 음수 구간 비중이 커져 비선형성이 강해지고(대신 정보 손실↑), 오른쪽으로 밀면 양수 구간 비중이 커져 정보는 보존되지만 거의 선형이 됩니다. **선형성 ↔ 비선형성 trade-off의 조절 손잡이**입니다 [C3-2].

**③ Conv2d — 필터와 얼마나 닮았는지를 재는 연산**

필터와 **같은 크기로 이미지의 일부**를 잘라와 원소별로 곱해 더합니다(행렬곱 형태의 내적). 이 값이 **클수록 그 위치의 이미지 조각이 그 필터가 찾는 특징을 강하게 가진다**는 뜻입니다. 그리고 필터 값은 고정된 상수가 아니라 **파라미터**라서, 모델은 이미지를 잘 인식하는 방향으로 **필터를 스스로 바꿔갑니다** [C3-1].

### **1. Convolution**  `[C3-1]`

> 허브: [R09]

- **코드**
    - `nn.Conv2d()`
- **패턴**
    - `nn.Conv2d(in_channels, out_channels, kernel_size=3, stride=stride, padding=1, bias=False)`
- 설명: ResNet의 기본 블록 내에서 이미지의 특징을 추출하는 가장 핵심적인 연산입니다. 3x3 크기의 커널을 사용하여 공간적인 특징을 훑으며, `stride` 설정에 따라 해상도를 유지하거나 줄이면서 채널 수를 확장합니다.
    - **연산의 의미**: 필터와 **같은 크기(3x3)로 입력의 일부**를 잘라와 원소별로 곱해 더합니다. 이 값이 **클수록 그 위치가 필터가 찾는 특징을 강하게 가진다**는 뜻이고, 필터를 전체 이미지에 미끄러뜨려 얻은 결과가 곧 "그 특징이 어디에 얼마나 있는가"의 feature map입니다.
    - **필터는 학습된다**: 커널 값은 사람이 설계해 넣는 상수가 아니라 **파라미터**입니다. 모델은 loss를 줄이는 방향으로 **필터를 스스로 바꿔가며**, 그 데이터셋을 잘 인식하는 데 필요한 특징 검출기를 직접 만들어냅니다.
- **코드 사용법**
    - **`bias=False`**: 합성곱 층 바로 뒤에 `BatchNorm`이 배치될 경우, `BatchNorm` 내부의 학습 가능한 파라미터가 편향(bias)의 역할을 대신하므로 메모리와 연산 효율을 위해 반드시 꺼주어야 합니다.
    - **`padding=1`**: 3x3 커널을 사용할 때, `stride=1`인 상황에서 입력과 출력의 해상도를 동일하게 유지하기 위해 필수적으로 설정해야 합니다.
- **Shape 흐름**
    - **Input**: `[128, 16, 32, 32]`
    - **Conv 3x3 (s=2)**: `stride=2`가 적용되면 가로와 세로 해상도가 각각 절반으로 줄어듭니다.
    - **Output**: `[128, 32, 16, 16]`(채널은 늘어나고 해상도는 압축된 형태)

### 2. **BatchNorm2d**  `[C3-2]`

> 허브: [R09]

- 코드
    - `nn.BatchNorm2d()`
- **패턴**
    - `nn.BatchNorm2d(out_channels)`
- 설명: 채널별로 입력의 **평균과 분산을 조절**해 학습 속도를 높이고 수렴을 안정화합니다. 네트워크가 깊어져도 하위 층의 파라미터 변화가 상위 층에 미치는 영향을 줄여줍니다.
    - **왜 ReLU 앞에 두는가**: ReLU는 음수를 버리고 양수를 그대로 통과시키므로, 입력 분포가 0을 기준으로 어디 있느냐가 **선형성 ↔ 비선형성 trade-off**를 정합니다 [C3-3]. BatchNorm은 그 분포의 평균·분산을 조절해 이 trade-off를 원하는 지점에 놓는 다이얼 역할을 합니다.
    - **주의 — 평균 0·분산 1은 목표가 아니라 출발점입니다**: 정규화 뒤에 학습 가능한 `γ`(scale)·`β`(shift)를 곱하고 더하는데, `γ=1, β=0`으로 **시작**하기 때문에 초기 상태가 평균 0·분산 1일 뿐입니다. 학습이 진행되면 모델이 채널별로 유리한 평균·분산을 스스로 찾아갑니다. **평균 0·분산 1이 되도록 학습시키는 것이 아닙니다.**
- **코드 사용법**
    - **위치 선정**: 일반적으로 `Conv2d` 연산 바로 뒤, `ReLU` 활성화 함수 앞에 위치합니다.
    - **모드 전환**: 학습 시와 평가 시 동작이 다르므로, 평가 시에는 반드시 `model.eval()`을 호출해 이동 평균을 사용하도록 해야 합니다.
- **Shape 흐름**
    - **Input**: `[128, 32, 16, 16]`
    - **BN**: 채널별 통계량을 계산하여 데이터의 스케일을 조정하지만 차원 자체는 변하지 않습니다.
    - **Output**: `[128, 32, 16, 16]`(채널은 늘어나고 해상도는 압축된 형태)

### 3. **ReLU**  `[C3-3]`

> 허브: [R06]

- 코드
    - `nn.ReLU()`
- **패턴**
    - `nn.ReLU(inplace=True)`
- 설명: 음수 값을 0으로 만드는 간단한 연산을 통해 네트워크에 비선형성을 부여합니다. 층이 깊어져도 기울기가 소실되지 않고 잘 전달되도록 돕는 일등 공신입니다. 그래프와 두 구간의 성질은 [C3] 앞머리 참고.
    - **음수 구간**: 0으로 눌리며 여기서 **비선형성이 나오지만**, 서로 다른 음수가 모두 0이 되므로 **정보 손실**이 발생하고 gradient도 0이 됩니다.
    - **양수 구간**: `y = x` 항등함수라 **정보는 보존되지만**, 이 구간만 쓰이면 선형과 같아 **비선형성을 학습하지 못합니다**.
    - 이 trade-off의 위치를 조절하는 것이 앞단의 BatchNorm입니다 [C3-2].
- **코드 사용법**
    - **`inplace=True`**: 새로운 메모리를 할당하지 않고 기존 텐서의 값을 직접 수정하여 메모리 효율을 높이지만, 원본 데이터가 필요할 경우 값이 덮어씌워질 수 있으므로 주의가 필요합니다.
    - **최종 활성화**: 블록의 마지막 ReLU는 반드시 Shortcut 합산 연산이 끝난 후에 적용해야 합니다.
- **Shape 흐름**
    - **Input**: `[128, 32, 16, 16]`
    - **ReLU**: 값의 범위만 조정(0 이상)할 뿐, 텐서의 크기에는 영향을 주지 않습니다.
    - **Output**: `[128, 32, 16, 16]`(채널은 늘어나고 해상도는 압축된 형태)

### 4. **ShortCut**  `[C3-4]`

> 허브: [R05] [R06]

- 코드
    - `nn.Identity()` or `x`
- **패턴**
    - `out = 메인경로 결과 + 숏컷경로 결과`
- 설명:  메인 경로의 결과 F(x)에 입력값 x를 그대로 더해주는 과정입니다. 모델이 입력과 출력의 차이인 잔차만을 학습하게 하여 최적화를 훨씬 쉽게 만들어줍니다.
- **코드 사용법**
    - **Shape 일치**: 두 텐서의 가로, 세로, 채널 크기가 1이라도 다르면 런타임 에러가 발생합니다. 차원이 다를 때는 앞서 배운 `Option A`나 `Option B`를 통해 반드시 형태를 맞춰야 합니다.
    - **합산 시점**: 두 번째 `BatchNorm`을 통과한 직후, 그리고 마지막 `ReLU`를 통과하기 직전에 수행합니다.
- **Shape 흐름**
    - **Main Path Output (F(x))**: `[128, 32, 16, 16]`
    - **Shortcut Output (x)**: `[128, 32, 16, 16]`
    - **Addition (F(x) + x)**: 두 텐서가 원소별로 합쳐지며 동일한 차원을 유지합니다.
    - **Final Output**: `[128, 32, 16, 16]`

---

### 5. shortcut 분기 로직  `[C3-5]`

- **코드**
    - `nn.Identity()` : 입력을 그대로 통과시키는 무연산 모듈
    - `ShortcutZeroPad` / `ShortcutProjection` : Option A / Option B 구현 [C1-2] [C2-1]
- **패턴**
    - `if <입력과 출력의 형태가 같은가>: 항등 경로  else: 차원을 맞추는 경로`
- 설명: shortcut을 무엇으로 할지는 취향이 아니라 **형태 일치 여부**가 정합니다. 더하기가 성립하는 상황이면 아무것도 하지 않는 것이 가장 싸고(파라미터 0), 성립하지 않는 상황에서만 A 또는 B를 고릅니다. A/B 중 어느 쪽인지는 형태가 아니라 **호출자가 준 정책 인자**로 결정됩니다.
- **코드 사용법**
    - 판정 조건은 두 가지를 **함께** 봐야 합니다 — 공간 크기가 그대로인가(stride), 채널이 그대로인가. 하나만 보면 stage 경계에서 틀립니다 [C5-4].
    - Bottleneck에서는 비교 대상이 `planes`가 아니라 `planes * expansion`입니다 [C4-4].
    - 모르는 정책 문자열이 오면 기본값으로 넘어가지 말고 `ValueError`로 끊습니다 — 오타가 조용한 성능 저하로 바뀌는 걸 막습니다.
    - shortcut 경로에는 활성화를 넣지 않습니다. identity 통로가 선형으로 남아야 residual의 전제가 유지됩니다 [C3-3].
- **Shape 흐름**
    - **stride 1 · 채널 동일:** `[128, 16, 32, 32]` → 항등 → `[128, 16, 32, 32]`
    - **stride 2 · 채널 2배:** `[128, 16, 32, 32]` → A 또는 B → `[128, 32, 16, 16]`

---

## **[4) BottleneckV1]**  `[C4]`

> 대응: [CS§3] · [N-23] [N-25] · [Q4]

> 허브: [R12]

깊은 ResNet(50/101/152층)에서 층당 연산 예산을 지키기 위해 쓰는 3층 블록입니다. **이 절의 Shape 흐름만 ImageNet 기준**입니다 — bottleneck은 CIFAR 실험에 쓰이지 않습니다 [C6-3]. 1×1로 채널을 줄여 3×3을 좁은 폭에서 돌리고, 다시 1×1로 되돌립니다.

### 1. 1x1 압축 (reduce)  `[C4-1]`

- **코드**
    - `nn.Conv2d()` : 채널 수 변경
    - `nn.BatchNorm2d()` : 배치 정규화
- **패턴**
    - `nn.Conv2d(in_channels, planes, kernel_size=1, stride=1, padding=0, bias=False)`
- 설명: 3×3 conv의 비용은 입력 채널 × 출력 채널에 비례하므로, 비싼 3×3 앞에서 채널을 미리 좁혀 두는 것이 bottleneck의 요점입니다. 여기서 공간 크기는 건드리지 않습니다(`stride=1`).
- **코드 사용법**
    - `kernel_size=1`이라 `padding=0`이면 H·W가 그대로 유지됩니다.
    - downsampling은 이 층이 아니라 다음 3×3이 담당합니다 — 여기에 stride를 주면 정보가 두 번 줄어듭니다.
    - 뒤에 BN이 붙으므로 `bias=False`입니다 [C3-1].
- **Shape 흐름**
    - **Input:** `[32, 256, 56, 56]`
    - **Conv 1x1 (s=1):** 채널만 축소
    - **Output:** `[32, 64, 56, 56]`

### 2. 3x3 처리  `[C4-2]`

- **코드**
    - `nn.Conv2d()` : 공간 특징 추출 + downsampling
    - `nn.BatchNorm2d()` : 배치 정규화
- **패턴**
    - `nn.Conv2d(planes, planes, kernel_size=3, stride=stride, padding=1, bias=False)`
- 설명: 블록에서 실제로 공간 정보를 보는 유일한 층입니다. 입출력 채널이 둘 다 좁은 `planes`라서, 같은 3×3이어도 BasicBlock보다 훨씬 쌉니다. stage 경계에서의 `stride=2`도 이 층이 맡습니다 [C5-4].
- **코드 사용법**
    - `kernel_size=3`에는 `padding=1`을 짝지어야 stride 1에서 크기가 보존됩니다.
    - `stride`는 블록 인자를 그대로 넘겨 받습니다 — 하드코딩하지 마세요.
    - 입력·출력 채널이 **둘 다 `planes`**입니다. `out_channels`를 넣으면 압축의 의미가 사라집니다.
- **Shape 흐름**
    - **Input:** `[32, 64, 56, 56]`
    - **Conv 3x3 (s=2):** 해상도 ½, 채널 유지
    - **Output:** `[32, 64, 28, 28]`

### 3. 1x1 확장 (expand)  `[C4-3]`

- **코드**
    - `nn.Conv2d()` : 채널 복원
    - `nn.BatchNorm2d()` : 배치 정규화
- **패턴**
    - `nn.Conv2d(planes, planes * expansion, kernel_size=1, stride=1, padding=0, bias=False)`
- 설명: 좁혀 둔 채널을 블록의 최종 출력 폭으로 되돌립니다. 이 층 뒤에는 ReLU가 오지 않습니다 — 덧셈 뒤에 한 번만 걸리는 post-activation 규칙 때문입니다 [C3-3].
- **코드 사용법**
    - 출력 채널은 `planes`가 아니라 `planes * expansion`입니다. 이 둘을 혼동하는 것이 bottleneck 구현 최대 버그원입니다.
    - `expansion`은 클래스 속성으로 두어 `_make_layer` 쪽에서 읽어 갈 수 있게 합니다 [C5-2].
    - BN까지만 하고 활성화는 덧셈 뒤로 넘깁니다.
- **Shape 흐름**
    - **Input:** `[32, 64, 28, 28]`
    - **Conv 1x1 (s=1):** 채널 4배 복원
    - **Output:** `[32, 256, 28, 28]`

### 4. expansion=4와 shortcut  `[C4-4]`

- **코드**
    - `nn.Identity()` : 차원이 맞을 때의 무연산 shortcut
    - `ShortcutProjection` : 차원이 어긋날 때의 1×1 conv shortcut [C2-1]
- **패턴**
    - stage의 출력 채널은 `planes` 자체가 아니라 **`planes`에 expansion을 곱한 값**입니다 → 이 값으로 shortcut 분기를 판정
- 설명: bottleneck은 블록 양 끝이 모두 고차원(`planes*4`)이라, shortcut을 projection으로 바꾸면 그 1×1 하나가 블록 본체만큼 비싸집니다. 그래서 조건이 맞는 한 identity를 유지하는 것이 성능이 아니라 **비용** 문제입니다.
- **코드 사용법**
    - 분기 판정은 `stride == 1 and in_channels == out_channels` 두 조건을 **함께** 봅니다 [C3-5].
    - 여기서 `out_channels`는 `planes`가 아니라 `planes * expansion`입니다 — 이걸 틀리면 shape은 맞는데 채널이 4배 어긋납니다.
    - 다음 블록의 `in_channels`도 `planes * expansion`으로 갱신해야 합니다 [C5-2].
- **Shape 흐름**
    - **Input:** `[32, 256, 56, 56]`
    - **shortcut (s=2, 채널 불일치):** projection 경로로 맞춤
    - **Output:** `[32, 256, 28, 28]` — 메인 경로와 더할 수 있는 형태

---

## **[5) ResNetV1]**  `[C5]`

> 대응: [CS§4] [CS§5] [CS§6] · [N-18]~[N-27] · [Q5]

> 허브: [R15] [R16] [R17]

### 1. **Model Stem**  `[C5-1]`

> 허브: [R15]

- **코드**
    - `nn.Sequential()`
    - `nn.MaxPool2d()`
- **패턴**
    - `nn.Sequential(nn.Conv2d(...), nn.BatchNorm2d(...), nn.ReLU(...))`
- **설명:**  데이터셋의 크기에 따라 전략이 달라집니다. 큰 이미지(ImageNet)는 정보를 빠르게 요약하고, 작은 이미지(CIFAR)는 정보 손실을 막기 위해 조심스럽게 시작합니다.
- **코드 사용법**
    - **CIFAR (32x32)**: 이미지가 작으므로 `MaxPool`을 생략하고 3x3 Conv로 시작하여 해상도를 최대한 보존합니다.
    - **ImageNet (224x224)**: 7 x7 Conv와 `MaxPool`을 사용하여 해상도를 초기 단계에서 1/4로 과감하게 줄입니다.
- **Shape 흐름**
    - **Input**: `[128, 3, 32, 32]`
    - **Out**: `[128, 16, 32, 32]`

### 2. **Layer Stacking**  `[C5-2]`

> 허브: [R16]

- **코드**
    - `list.append()`
    - `block.expansion`
- **패턴**
    - `layers.append(item)` : 리스트에 블록 요소 추가
    - `self.in_channels = new_value` : 다음 블록을 위해 입력 채널 수 갱신
- **설명:** 스테이지가 바뀔 때 채널이 늘어나면, 모델은 "이제부터 들어올 데이터의 채널은 이만큼이다"라고 기억해둬야 합니다. 이 업데이트를 누락하면 다음 블록을 만들 때 차원 불일치 에러가 발생합니다.
- **코드 사용법**
    - **First Block**: 레이어의 첫 번째 블록에서만 `stride`를 적용해 해상도를 줄이고 채널을 확장합니다.
    - **State Update (Trap 주의)**: 첫 블록을 만든 직후, `self.in_channels`를 **(출력 채널 × expansion)** 값으로 반드시 갱신해야 합니다.
    - **Remaining Blocks**: 나머지 블록들은 `stride=1`로 고정하여 동일한 크기를 유지하며 쌓습니다.
- **Shape 흐름**
    - **Layer 1 Start**: `[128, 16, 32, 32]`
    - **Layer 2 Start**: `[128, 32, 16, 16]`
    - **Layer 3 Start**: `[128, 64, 8, 8]`

### 3. **Final Classification**  `[C5-3]`

> 허브: [R17]

- **코드**
    - `nn.AdaptiveAvgPool2d()`
    - `torch.flatten()`
    - `nn.Linear()`
- **패턴**
    - `pool((1, 1))`
    - `flatten(x, 1)`
- **설명:** 수많은 특징 값들을 전역 평균 풀링으로 요약하여 위치 정보에 유연하게 대응하게 만든 뒤, 분류기에 전달하여 정답을 맞힙니다.
- **코드 사용법**
    - **Global Pooling**: 가로/세로 크기를 1x1로 만들어 특징 맵 전체의 평균적인 경향성을 추출합니다.
    - **Flatten**: 2D 형태의 이미지를 1D 벡터로 펴서 `Linear` 레이어에 입력 가능한 형태로 만듭니다.
    - **Classifier**: 클래스 개수만큼의 출력 노드를 설정합니다.
- **Shape 흐름**
    - **Last Layer Out**: `[128, 64, 8, 8]`
    - **AvgPool**: `[128, 64, 1, 1]`
    - **Flatten**: `[128, 64]`
    - **Linear**: `[128, 10]`

---

### 4. stage 채널·stride 스케줄  `[C5-4]`

- **코드**
    - (torch API 없음 — stage별 채널·stride를 정하는 규칙)
- **패턴**
    - `stage_planes = [base, base * ?, base * ?]` / `stride = <stage 시작에서만 2, 그 외 1>`
- 설명: 논문 §3.3의 설계 규칙은 "맵 크기가 절반이면 필터 수는 2배"입니다. 이 둘은 **같은 지점에서 동시에** 일어나야 층당 연산량이 유지됩니다. 그 지점이 곧 stage의 첫 블록이고, 나머지 블록은 크기·채널을 그대로 둡니다.
- **코드 사용법**
    - 첫 stage는 stem이 이미 크기를 정해 놓았으므로 downsample하지 않습니다.
    - stride를 stage의 **모든** 블록에 주면 맵이 기하급수로 줄어 GAP 직전에 1×1 밑으로 무너집니다 [C5-3].
    - 블록 루프에서 다음 블록의 `in_channels`를 이번 블록의 출력으로 갱신해야 합니다. 빠뜨리면 두 번째 블록에서 shape mismatch가 납니다 [C5-2].
    - CIFAR의 base 채널은 생성자 인자(`cifar_base_channels`)로 들어옵니다 — 숫자를 직접 박지 말고 그 인자에 배수를 적용하세요.
- **Shape 흐름**
    - **stage 1:** `[4, base, 32, 32]` — stride 1, 채널 유지
    - **stage 2:** `[4, base×?, 16, 16]` — 시작 블록만 stride 2, 채널 배증
    - **stage 3:** `[4, base×?, 8, 8]` — 동일 규칙 반복

---

## **[6) CIFAR Factory (depth = 6n+2)]**  `[C6]`

> 대응: [CS§5] · [N-28]~[N-30] · [Q5]

> 허브: [R15]

`depth` 하나만 받아 CIFAR용 ResNet을 조립하는 팩토리 함수입니다. 여기서 다루는 것은 torch API가 아니라 **깊이 규칙과 호출 규약**이라, 다른 절과 달리 "어떤 함수를 쓰나"보다 "무엇이 계약인가"가 본론입니다.

### 1. 6n+2 유도  `[C6-1]`

- **코드**
    - (torch API 없음 — 층을 세는 산술)
- **패턴**
    - `총 weighted layer = stem conv + 블록들의 conv + 최종 FC`
- 설명: CIFAR-ResNet은 stage가 3개, 각 stage에 같은 수의 블록, 블록마다 3×3 conv 2개입니다. 여기에 학습 파라미터를 갖는 층은 앞의 stem conv 하나와 맨 뒤 FC 하나가 더 있습니다. GAP은 파라미터가 없어 이 셈에 들어가지 않습니다 [C5-3].
- **코드 사용법**
    - 세는 대상은 "층"이지 "블록"이 아닙니다. 블록 수 × 2 = conv 수라는 환산을 빠뜨리면 정확히 2배가 어긋납니다.
    - stage 3개는 feature map {32, 16, 8}에 대응합니다 [C5-4].
    - 논문이 실험한 depth(20/32/44/56/110)를 이 셈에 대입해 n이 정수로 떨어지는지 직접 확인해 보세요.
- **Shape 흐름**
    - **Input:** `depth` (정수 하나)
    - **유도:** stem 1 + conv 2×(stage 3 × 블록 n) + FC 1
    - **Output:** 그 depth가 표현 가능한지 여부와, 가능하다면 stage당 블록 수

### 2. 검증 계약  `[C6-2]`

- **코드**
    - `raise ValueError(...)` : 규칙 위반 시 조기 실패
- **패턴**
    - `if <depth가 규칙을 만족하지 않으면>: raise ValueError("CIFAR depth must be 6n+2")`
- 설명: 잘못된 depth를 조용히 반올림하지 않고 예외로 끊는 것이 이 함수의 계약입니다. 모델을 다 만든 뒤 shape 에러로 터지는 것보다, 인자를 받은 즉시 실패하는 편이 디버깅이 훨씬 쉽습니다.
- **코드 사용법**
    - Check 5가 요구하는 스펙은 두 줄입니다 — `depth=20`은 통과, `depth=21`은 `ValueError`.
    - 잡는 쪽이 `except ValueError`이므로 `AssertionError`나 `Exception`을 던지면 테스트가 실패합니다.
    - 검사는 n을 계산하기 **전에** 둡니다. 나눗셈이 먼저 돌면 잘못된 depth에서도 어중간한 n이 나옵니다.
- **Shape 흐름**
    - **Input:** `depth=20` → 통과
    - **Input:** `depth=21` → `ValueError`
    - **Output:** 통과한 경우에만 다음 단계로 진행

### 3. ResNetV1 호출 규약  `[C6-3]`

- **코드**
    - `ResNetV1(block, layers, num_classes, stem=..., shortcut=...)` : 공용 생성자
- **패턴**
    - `ResNetV1(BasicBlockV1, <stage별 블록 수 리스트>, num_classes, stem="cifar", shortcut=shortcut)`
- 설명: CIFAR과 ImageNet은 같은 `ResNetV1`을 쓰고 인자로만 갈립니다. `stem="cifar"`는 3×3 stride 1 단일 conv에 maxpool 없음을 뜻하고 [C5-1], `shortcut="zero_pad"`는 논문 §4.2가 CIFAR 실험을 Option A로 고정했기 때문입니다 [C1-2].
- **코드 사용법**
    - block은 `BasicBlockV1`입니다 — CIFAR 실험에는 bottleneck을 쓰지 않습니다 [C4-4].
    - stage 블록 수 리스트는 길이 3이고 세 값이 모두 같습니다(각 stage에 n개씩).
    - `shortcut`은 하드코딩하지 말고 인자를 그대로 전달해, 학습자가 projection으로 바꿔 비교할 수 있게 둡니다.
    - ImageNet stem을 쓰면 32×32 입력이 시작부터 8×8로 줄어 stage 구조가 무너집니다.
- **Shape 흐름**
    - **Input:** `[4, 3, 32, 32]`
    - **stage 1/2/3:** 32×32 → 16×16 → 8×8, 채널 16 → 32 → 64
    - **Output:** `[4, num_classes]`

---

## **[부록) 에러 메시지별 디버깅 표]**

> 허브: [R05] [R16] [R09]

빈칸을 채우다 터지는 에러는 종류가 정해져 있습니다. 메시지를 보고 **어느 절로 돌아갈지** 바로 찾는 표.

| 에러 메시지 (요지) | 실제 원인 | 돌아갈 절 |
| --- | --- | --- |
| `RuntimeError: The size of tensor a (X) must match the size of tensor b (Y) at non-singleton dimension 1` | 덧셈 직전 **채널**이 안 맞음. shortcut이 identity인데 stage 경계라 채널이 2배가 됐습니다 | [C3-5] [C5-4] |
| 같은 메시지인데 `dimension 2` 또는 `3` | 채널이 아니라 **H·W**가 안 맞음. 메인 경로만 stride 2를 먹고 shortcut은 안 먹었습니다 | [C1-1] [C2-1] |
| `RuntimeError: Given groups=1, weight of size [A,B,..], expected input[..,C,..] to have B channels, but got C channels` | conv의 `in_channels` 선언과 실제 입력이 다름. 블록 루프에서 `in_channels` 갱신 누락 | [C5-2] [C4-4] |
| `RuntimeError: Expected all tensors to be on the same device, but found at least two devices` | 모델은 `.to(device)` 했는데 **새로 만든 텐서**(zero-pad의 `zeros`)가 CPU에 남았습니다 | [C1-2] |
| `RuntimeError: mat1 and mat2 shapes cannot be multiplied (AxB and CxD)` | FC의 `in_features`가 마지막 stage 채널과 다름. bottleneck이면 `planes*expansion`을 안 곱했습니다 | [C5-3] [C4-3] |
| `TypeError: linear(): argument 'input' (position 1) must be Tensor, not tuple` / 4D 입력 오류 | GAP 뒤 **flatten 누락** | [C5-3] |
| `ValueError: CIFAR depth must be 6n+2` | 의도된 동작. depth가 규칙을 안 지켰습니다 | [C6-2] |
| 에러 없이 **정확도만** 이상하게 낮음 | (a) 덧셈 뒤 ReLU 누락 (b) `model.eval()` 안 하고 평가 — BN이 배치 통계를 계속 갱신 (c) shortcut 경로에 ReLU를 넣음 | [C3-3] [C3-5] |
| 학습이 아예 안 내려감 (loss 발산) | lr이 너무 큼. 논문 CIFAR 기본은 0.1이고, 110층은 warm-up이 따로 있습니다 | [R13] |

> **읽는 법**: 앞의 다섯 개는 *shape 계약*이 깨진 것이고, 마지막 두 개는 shape이 맞는데도 **의미**가 틀린 경우입니다.
> 후자가 잡기 어려우니 `✅ Check n` 셀을 건너뛰지 말 것.

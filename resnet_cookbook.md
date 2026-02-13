A.ing ResNet 쿡북

### **\[Step 1\. Shortcut Option A \- Slicing\]**

* **사용 함수**: Python Slicing Syntax (`::step`)  
* **패턴 예시**:  `x[:, :, ::stride, ::stride]` (Height와 Width 차원만 샘플링)  
* **설명**: ResNet Identity Mapping 시, 메인 경로에서 Stride가 적용되어 Feature Map의 크기가 줄어든 경우 F(x) \+ x 연산이 불가능해집니다. 이를 해결하기 위해 입력값 x 에서도 동일한 간격으로 데이터를 추출하여 파라미터 없이 해상도를 맞춥니다.  
* **주의 사항**  
  * 높이와 너비 차원에 대해 Stride 값만큼 건너뛰며 데이터를 추출하세요.  
  * 배치와 채널 차원은 건드리지 않아야 합니다.  
* **Shape 흐름**  
  * **Input:** `[32, 64, 56, 56]`  
  * **Slicing (::2):** 가로/세로를 stride 간격으로 점프하며 선택  
  * **Output:** `[32, 64, 28, 28]`

---

### **\[Step 2\. Shortcut Option A \- Zero Padding\]**

다운샘플링된 텐서의 채널 수가 목표 출력 채널 수보다 적을 때, 0을 채워 차원을 확장하는 과정입니다.

* **코드**  
  * `torch.zeros()` : 0으로 채워진 텐서 생성  
  * `torch.cat()` : 텐서 연결  
  * `tensor.size()` : 텐서 크기 확인  
* **패턴**  
  * `torch.zeros(batch, ch, h, w)` : 지정된 크기의 0 텐서 생성 (device, dtype 일치 필수)  
  * `torch.cat([A, B], dim=1)` : 채널 축(dim=1)을 기준으로 연결  
* **코드 사용법**  
  * **필요 채널 계산:** (목표 출력 채널) \- (현재 입력 채널)을 구하세요.  
  * **0 생성:** 입력 텐서와 동일한 공간 크기를 가지면서, 위에서 계산한 부족한 채널만큼의 깊이를 가진 0 텐서를 만듭니다.  
  * **결합:** 입력 텐서 뒤에 0 텐서를 이어 붙입니다.  
* **Shape 흐름**  
  * **Input:** `[32, 64, 28, 28]` (Step 1 결과)  
  * **Zeros:** `[32, 64, 28, 28]` (새로 만든 0\)  
  * **Output:** `[32, 128, 28, 28]`

---

### **\[Step 3\. Shortcut Option B \- Projection\]**

1x1 합성곱을 사용하여 해상도 감소와 채널 확장을 동시에 수행하는 학습 가능한 숏컷 방식입니다.

* **코드**  
  * `nn.Conv2d()` : 합성곱 연산  
  * `nn.BatchNorm2d()` : 배치 정규화  
* **패턴**  
  * `nn.Conv2d(in, out, kernel_size=1, ...)` : 픽셀 간 정보 교환 없이 채널만 변경  
* **코드 사용법**  
  * 커널 크기는 1로 고정하여 채널 간 연산만 수행합니다.  
  * 공간 해상도를 줄이기 위해 메인 경로와 동일한 `stride`를 적용하세요.  
  * 합성곱 뒤에는 반드시 `BatchNorm`을 연결해야 분포가 깨지지 않습니다.  
* **Shape 흐름**  
  * **Input:** `[32, 64, 56, 56]`  
  * **Conv 1x1 (s=2):** 해상도 ½, 채널 2배  
  * **Output:** `[32, 128, 28, 28]`

---

### **\[Step 4\. Basic Block \- Main Convolution\]**

ResNet의 기본 블록 내에서 특징을 추출하는 3x3 합성곱 층을 정의합니다.

* **코드**  
  * `nn.Conv2d()`  
* **패턴**  
  * `bias=False` : BatchNorm 사용 시 편향(bias) 제거  
  * `padding=1` : 3x3 커널에서 크기 유지를 위해 필수  
* **코드 사용법**  
  * 커널 크기 3, 패딩 1을 사용하여 입력과 출력의 해상도가 유지되도록 설정합니다. (Stride가 1일 경우)  
  * 바로 뒤에 `BatchNorm`이 올 것이므로, `Conv2d`의 편향(bias) 파라미터는 메모리 낭비이므로 끄세요.  
* **Shape 흐름**  
  * **Input:** `[32, 64, 56, 56]`  
  * **Conv 3x3 (s=2):**  
  * **Output:** `[32, 128, 28, 28]`

---

### **\[Step 5\. Basic Block \- Shortcut Logic (Branching)\]**

블록의 초기화 단계에서, 입력 데이터가 변형(Identity) 없이 통과할지, 혹은 차원을 맞춰야 할지 결정하는 로직입니다.

* **코드**  
  * `nn.Identity()` : 입력을 그대로 반환하는 레이어  
  * `if/else` 조건문  
* **패턴**  
  * `layer = nn.Identity()` : 아무 연산도 하지 않음  
  * `if condition:` : 조건에 따른 분기 처리  
* **코드 사용법**  
  * 두 가지 조건을 검사하세요.  
    * 공간 크기가 줄어드는가? (`stride != 1`)  
    * 채널 수가 변하는가? (`in_channels != out_channels`)  
  * 두 조건 모두 해당하지 않는다면(변화 없음), 연산 비용이 없는 `Identity`를 할당합니다.  
  * 하나라도 해당한다면, 앞서 구현한 \*\*Shortcut 모듈(Step 1\~3)\*\*을 할당하여 차원을 맞춰줍니다.  
* **Shape 흐름**  
  * **Main Path Output:** `[B, C_out, H', W']`  
  * **Shortcut Output:** `[B, C_out, H', W']` (반드시 위와 같아야 함)

---

### **\[Step 6\. Basic Block \- Forward Pass (Residual Connection)\]**

정의된 레이어들을 통과시키고, 원본 정보(Identity)와 추출된 정보(Convolution)를 더하는 단계입니다.

* **코드**  
  * `+` 연산자 : Element-wise Sum  
  * `F.relu()` 또는 `nn.ReLU()`  
* **패턴**  
  * `out = layer(x)`  
  * `out = out + residual`  
* **코드 사용법**  
  * **Shortcut 계산:** 입력 x를 Step 5에서 결정된 shortcut 레이어에 통과시켜 잔차(residual)를 준비합니다.  
  * **Main Path 계산:** Conv \-\> BN \-\> ReLU \-\> Conv \-\> BN 순서로 연산합니다. (마지막 ReLU는 아직 적용하지 마세요.)  
  * **Add:** Main Path의 결과와 Shortcut의 결과를 더합니다. 두 텐서의 Shape이 다르면 여기서 에러가 발생합니다.  
  * **Final Activation:** 더한 값에 마지막으로 ReLU를 적용합니다.  
* **Shape 흐름**  
  * **Main Path:** `[32, 128, 28, 28]`  
  * **Shortcut:** `[32, 128, 28, 28]`  
  * **Result:** `[32, 128, 28, 28]`

---

### **\[Step 7\. Model Stem \- Dataset Constraints\]**

데이터셋(ImageNet vs CIFAR-10)의 이미지 크기에 따라 초기 진입부(Stem) 구조를 다르게 설계합니다.

* **코드**  
  * `nn.Sequential()`  
  * `nn.MaxPool2d()`  
* **패턴**  
  * `nn.Sequential(layer1, layer2, ...)` : 레이어를 순차적으로 묶음  
* **코드 사용법**  
  * **ImageNet (224x224):** 이미지가 크므로 정보를 빠르게 압축해야 합니다. 7x7 Conv와 MaxPool을 사용하여 해상도를 1/4로 줄이세요.  
  * **CIFAR (32x32):** 이미지가 매우 작습니다. 초반에 MaxPool을 쓰거나 stride를 크게 주면 정보가 소실됩니다. 3x3 Conv만 사용하여 해상도를 유지하세요.  
  * **공통:** Conv 뒤엔 반드시 BN과 ReLU가 따라와야 학습이 진행됩니다.  
* **Shape 흐름**  
  * **Input Image:** `[32, 3, 224, 224]`  
  * **Conv 7x7 (s=2):** `[32, 64, 112, 112]` (절반 감소)  
  * **MaxPool 3x3 (s=2):** `[32, 64, 56, 56]` (다시 절반 감소)  
  * **Final Stem Out:** `[32, 64, 56, 56]`

---

### **\[Step 8\. Layer Stacking \- State Management\]**

여러 개의 블록을 쌓으면서 채널 수가 변할 때, 모델의 내부 상태(`self.in_channels`)를 관리하는 로직입니다.

* **코드**  
  * `list.append()`  
  * `block.expansion` : 블록 타입별 채널 확장 비율 (BasicBlock=1, Bottleneck=4)  
* **패턴**  
  * `layers.append(item)` : 리스트에 요소 추가  
  * `self.var = new_value` : 클래스 멤버 변수 갱신  
* **코드 사용법**  
  * **First Block:** 레이어의 첫 번째 블록은 stride를 적용하고 채널을 변경할 수 있습니다.  
  * **State Update (Trap 주의):** 첫 블록을 생성한 직후, 다음 블록들이 사용할 입력 채널 수(`self.in_channels`)를 (출력 채널 × 확장 비율)로 갱신해야 합니다. 이 갱신을 누락하면 다음 레이어 생성 시 차원 불일치가 발생합니다.  
  * **Remaining Blocks:** 나머지 블록들은 stride=1로 고정하여 쌓습니다.  
* **Shape 흐름**  
  * **Layer 1 Start:** `[B, 64, H, W]`  
  * **Layer 2 Start:** `[B, 128, H/2, W/2]` (채널 2배, 크기 1/2)

---

### **\[Step 9\. Final Classification\]**

공간 정보를 압축하고 클래스 확률을 출력하는 마지막 단계입니다.

* **코드**  
  * `nn.AdaptiveAvgPool2d()` : 출력 크기 강제 고정  
  * `torch.flatten()` : 1차원으로 펼치기  
  * `nn.Linear()` : 분류기  
* **패턴**  
  * `pool((1, 1))` : 어떤 크기가 들어와도 1x1로 만듦  
  * `flatten(x, 1)` : 배치(0번) 차원을 제외하고 모두 핌  
* **코드 사용법**  
  * **Global Pooling:** 특징 맵의 가로/세로 크기에 상관없이 1x1 크기의 특징 벡터 하나로 압축합니다.  
  * **Flatten:** 완전 연결 층(Linear)에 넣기 위해 2D 이미지를 1D 벡터로 폅니다.  
  * **Classifier:** 최종 클래스 개수만큼의 출력을 가지는 선형 레이어를 통과시킵니다.  
* **Shape 흐름**  
  * **Last Layer Out:** `[32, 512, 7, 7]` (가정)  
  * **AvgPool:** `[32, 512, 1, 1]` (공간 정보 삭제)  
  * **Flatten:** `[32, 512]` (벡터화)  
  * **Linear:** `[32, 1000]`


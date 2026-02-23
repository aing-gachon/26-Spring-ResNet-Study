# ResNet (He et al., 2016) 질문 정리 (Questions Only)

> *Deep Residual Learning for Image Recognition* 기반 문제 정리  
> (모범답안 제외 버전)

---

## 1) Degradation 문제는 Overfitting과 어떻게 다른가?

논문은 '깊어질수록 성능이 나빠지는 현상(degradation)'이 overfitting이 아니라 **최적화(학습) 문제**라고 주장한다.

- Figure 1을 근거로 degradation과 overfitting을 구분하시오.
- 왜 degradation이 '학습이 안 되는 문제(optimization issue)'인지 설명하시오.

---

## 2) H(x) 대신 F(x)=H(x)−x를 학습하면 왜 더 쉬운가?

Section 3.1에서 저자들은 원하는 mapping을 H(x)라 두고, 잔차를 F(x)=H(x)−x로 재정의하여  
`y = F(x) + x` 형태로 학습한다.

(1) 최적의 해가 identity mapping(H(x)=x)일 때, 왜 F(x)=0으로 학습하는 것이 더 쉬운지 설명하시오.  

(2) 이 논증이 degradation 문제와 어떻게 연결되는지 설명하시오.

---

## 3) Shortcut Option A/B/C는 무엇이며, 논문은 왜 Option C를 버렸는가?

차원이 유지될 때는 identity shortcut `y = F(x) + x`를 쓸 수 있지만,  
차원이 증가하거나 spatial downsampling이 발생하면 shortcut 설계가 달라진다.

(1) 논문이 말하는 Option A/B/C를 각각 설명하시오.  
또한 Option B에서 `y = F(x) + W_s x`가 의미하는 바를 설명하시오.

(2) Table 3 결과와 저자 논의를 근거로,  
왜 논문은 Option C를 이후 실험에서 사용하지 않겠다고 결론 내렸는지 설명하시오.

---

## 4) Bottleneck block은 왜 필요하며, 왜 identity shortcut이 특히 중요해지는가?

ResNet-34는 basic block(3×3, 3×3)을 쓰지만,  
ResNet-50/101/152는 bottleneck block(1×1, 3×3, 1×1)을 사용한다.

(1) bottleneck에서 두 개의 1×1 convolution은 각각 어떤 역할을 하는가?  

(2) 논문은 bottleneck에서 identity shortcut을 projection으로 바꾸면  
time complexity와 model size가 크게 증가한다고 주장한다.  
그 이유를 설명하시오.

---

## 5) CIFAR-10에서 6n+2 규칙은 어떻게 나오며, 왜 shortcut을 identity로 고정했는가?

논문 4.2(CIFAR-10)에서는 '6n+2 weighted layers' 규칙의 단순 구조를 사용한다.

(1) 네트워크 구성을 기반으로 6n+2가 되는 과정을 유도하시오.  

(2) shortcut이 3n개가 되는 이유를 설명하시오.  

(3) CIFAR-10 실험에서 왜 모든 shortcut을 identity(option A)로 고정했는지  
논문 의도를 설명하시오.

---

## 참고 문헌

- Kaiming He, Xiangyu Zhang, Shaoqing Ren, Jian Sun.  
  *Deep Residual Learning for Image Recognition.* CVPR 2016.

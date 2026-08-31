[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC_BY--NC_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)

## Introduction
A.ing 주니어 트랙 ResNet 세션은 ResNet에 대한 근본적인 이해를 목표로 합니다.<br>
단순히 모델을 가져다 쓰는 것이 아니라, 그 내부 구조와 작동 원리를 이해하고, 자신의 데이터에 맞춰 수정하고 개선할 수 있는 능력을 기르는 것을 목표로 합니다.<br>

## Eligibility
### Required
- 기초 파이썬 프로그래밍 능력
- 로컬 Jupyter Notebook 또는 Python 스크립트 실행 환경을 설정할 수 있는 능력
- 팀 프로젝트에 적극적으로 참여하고 지속할 수 있는 의지
### Recommended
- 머신러닝/딥러닝 기본 개념에 대한 이해
- PyTorch 또는 TensorFlow와 같은 딥러닝 프레임워크 경험
- CNN 아키텍처에 대한 기본적인 이해 
- 선형대수에 대한 기본 이해 (행렬 연산, 벡터 공간 등)
### Expected Outcomes
- ResNet의 아키텍처와 핵심 아이디어에 대한 깊은 이해
- 논문의 수식을 직접 코드로 구현할 수 있는 능력
- 자신의 데이터에 맞게 모델을 수정하고 개선할 수 있는 능력
- 팀 프로젝트를 통해 협업 능력과 문제 해결 능력 향상

## 자료가 서로 연결되는 방식

자료 4종(치트시트·쿡북·빈칸 노트북·퀴즈)은 각자 영어 논문만 참조하기 때문에, 논문을 못 읽으면 4개가 전부 파편이 됩니다.
그래서 **논문 좌표 허브** [`Week 1/resnet_paper_guide.md`](Week%201/resnet_paper_guide.md)를 하나 두고 모든 자료가 그쪽을 가리킵니다.

```text
            [허브] resnet_paper_guide.md
                     R01 ~ R17
        ┌────────┬───────┴────────┬────────┐
     치트시트     쿡북          노트북      퀴즈
      [CS§n]     [Cn-m]         [N-k]    [Qn]/[An]
```

허브의 각 R 항목은 자기를 참조하는 자료 좌표를 전부 나열하고, 각 자료는 자기 절이 어느 R인지 밝힙니다 (양방향).

| 표기 | 가리키는 것 |
| --- | --- |
| `[R05]` | 허브의 R 항목 |
| `[CS§n]` | 치트시트 n번 절 |
| `[Cn-m]` | 쿡북 n절 m번 항목 |
| `[N-k]` | 빈칸 노트북 k번 빈칸 |
| `[Qn]` / `[An]` | 퀴즈 문항 / 모범답안 |

대괄호 한 겹이라 링크가 아니라 **좌표**입니다. 막히면 그 좌표를 따라가면 됩니다.

## Weekly Plan

| 주차 | 단계 | 자료 | 세부 사항 |
| --- | --- | --- | --- |
| 0 | 준비 | [선수자료](Week%201/resnet_prerequisites.md) | MLP·최적화·활성화·BN·CNN 기초 점검 |
| 1 | 스터디 | 논문, [허브](Week%201/resnet_paper_guide.md), [치트시트](Week%201/A.ing_resnet_cheat_sheet.md), [퀴즈](Week%201/resnet_questions.md) | 허브 R01~R05·R08·R10·R11을 논문과 나란히 읽고 퀴즈로 확인 |
| 2 | 구현 | [쿡북](Week%202/resnet_cookbook.md), [빈칸 노트북](Week%202/Aing_resnet_from_scratch_blank.ipynb) | 쿡북 가이드를 따라 빈칸 N-1~N-30을 직접 코딩 |
| 3 | 실험 | [리그전 노트북](Week%203/Aing_league_ResNet_CIFAR10.ipynb) | 하이퍼파라미터 튜닝 및 리그전 |
| 4 | 정리 | 실험 데이터 기록, 허브 R14·R02·R10 | 1등 팀의 전략 발표 및 로그 분석 |

## Paper Link

https://arxiv.org/abs/1512.03385

## Folder Structure
```text
26-Spring-ResNet-Study/
├─ README.md
├─ rules.md                                   # 출석·결석·공결 규칙
├─ Week 1
│    ├─ resnet_paper_guide.md                 # 허브. 논문 §1~§4.2의 R01~R17 좌표
│    ├─ resnet_prerequisites.md               # 시작 전 딥러닝 배경
│    ├─ A.ing_resnet_cheat_sheet.md           # 개념 정리 (§0~§6)
│    ├─ A.ing_resnet_week1_appendix.md        # 1주차 부록
│    ├─ resnet_questions.md                   # 퀴즈 Q1~Q5
│    └─ resnet_questions_sample_answer.md     # 모범답안
├─ Week 2
│    ├─ resnet_cookbook.md                    # torch API 사용법 + 에러 디버깅 표
│    ├─ Aing_resnet_from_scratch_blank.ipynb  # 빈칸 N-1~N-30
│    └─ Aing_resnet_from_scratch_answer.ipynb # 정답 + Bottleneck 참고 구현
└─ Week 3
     └─ Aing_league_ResNet_CIFAR10.ipynb      # 튜닝 리그전 (FIXED/TUNE 구분)
```

빈칸 노트북은 **먼저 스스로 채운 뒤에** 정답 노트북을 여세요. 쿡북에는 정답 코드가 없고 API 사용법만 있습니다.

## 담당자

| 이름 | 이메일 |
|------|--------|
| 문예훈 (자료 제작 총괄) | hoobhoob04@gachon.ac.kr |
| 정진용 | wlsdyd5373@gachon.ac.kr |
| 전지우 | jiwoo424@gachon.ac.kr |
질문이나 도움이 필요하면 언제든 연락 주세요!<br>
한 학기 동안 함께 열심히 공부해봅시다!

---

Copyright © 2026 A.ing. Licensed under CC BY-NC 4.0.

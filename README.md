# Kubernetes 배포 정의 방식 - Manifest, Helm, Kustomize

# ArgoCD 기반

## 1. 공통점
- 모두 **Kubernetes 리소스 배포 정의** 목적
- 최종적으로 사람이 사용할 **YAML** 파일 생성

---

## 2. 비교표

| 방식        | 특징                      | 장점                 | 단점              |
| --------- | ----------------------- | ------------------ | --------------- |
| Manifest  | 기본 YAML 작성              | 단순/직관              | 재사용 및 환경 분리 어려움 |
| Helm      | 템플릿 & 변수 활용, Chart 구조   | 변수화, 패키지 생태계       | 러닝커브, 복잡도       |
| Kustomize | YAML Patch, Overlays 지원 | 기존 YAML 재사용, 환경 분리 | 제한적 변수화         |

---
## 3. 차이점/공통점 요약

| 방식        | 쉽고 빠름 | 재사용/확장성 | 환경분리 | 템플릿/변수 |
| --------- | :---: | :-----: | :--: | :----: |
| Manifest  |   O   |    X    |  X   |   X    |
| Helm      |   △   |    O    |  O   |   O    |
| Kustomize |   △   |    O    |  O   | O(부분)  |

---
## 4. step by step
### [k8s  - Manifest](./docs/manifest.md)
### [k8s  - Helm](./docs/helm.md)
### [k8s  - Kustomize](./docs/kustomize.md)

---

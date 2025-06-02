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

## 5. k8s mcp-server 실행 방법

### 5.1 kind 클러스터 준비
- 클러스터 삭제:
  ```bash
  kind delete cluster --name mcp-cluster
  ```
- 클러스터 생성:
  ```bash
  kind create cluster --name mcp-cluster
  ```

### 5.2 필수 컴포넌트 설치 (Helm)
- ingress-nginx 설치:
  ```bash
  helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-nginx --create-namespace
  ```
- cert-manager 설치:
  ```bash
  helm install cert-manager jetstack/cert-manager \
    --namespace cert-manager --create-namespace \
    --set installCRDs=true
  ```

### 5.3 ArgoCD 설치 및 접속
- 네임스페이스 생성:
  ```bash
  kubectl create namespace argocd
  ```
- 설치:
  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```
- admin 비밀번호 확인:
  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  ```
- 포트포워딩 및 브라우저 접속:
  ```bash
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  # 브라우저: https://localhost:8080
  ```

### 5.4 CD 설정
- infra-cd 저장소 활용:
  - Repository: `https://github.com/MiddleKD/infra-cd.git`
  - kustomize 경로: `mcp-server/kustomize/overlays/dev`

---
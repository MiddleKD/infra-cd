# k8s 배포 정의 방식 - Manifest

## Manifest란?
Kubernetes에서 가장 기본적인 배포 정의 방식으로, 리소스(YAML)를 ‘있는 그대로’ 기술하여 바로 배포하는 방법입니다.

---

## 특징
- 템플릿, 별도 도구 없이 YAML만 작성
- 명확하고 단순함
- 복잡한 변수/환경별 분리에는 부적합

---

## 장점과 단점

| 항목     | 설명              |
| ------ | --------------- |
| **장점** | 매우 단순, 직관적      |
| **단점** | 재사용, 환경별 분리 어려움 |

---
## 사용 흐름

### Manifest (Raw YAML)
- 가장 직관적인 방식. '그대로' 쓴다.
- 예) deployment.yaml, service.yaml 작성 → 바로 적용
- **재귀적으로 yaml 파일 탐색 X**

---

## 파일/디렉토리 구조 예시

```plaintext
infra-gitops/
└── manifest/
    └── hello-server/
        ├── deployment.yaml
        └── service.yaml
```

---

## 실전 예시

### 1. Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-server
  template:
    metadata:
      labels:
        app: hello-server
    spec:
      containers:
        - name: hello
          image: yourdockerhub/hello-server:latest
          ports:
            - containerPort: 80
```

### 2. Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-server
spec:
  selector:
    app: hello-server
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

---

## 배포 방법

1. 리소스 YAML 파일 작성
2. kubectl 명령어로 배포

```bash
kubectl apply -f manifest/hello-server/
```
또는 파일별 개별 적용:
```bash
kubectl apply -f manifest/hello-server/deployment.yaml
kubectl apply -f manifest/hello-server/service.yaml
```

## ArgoCD
```bash
argocd app create hello-server-manifest \
  --repo https://github.com/yourname/infra-gitops \
  --path manifest/hello-server \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```

---

## 실무 활용 팁
- 단순 서비스나 파일 수 적을 때 적합
- 환경별로 분리하면 파일 복사/관리 복잡성 ↑ (개선 필요 시 Helm/Kustomize 추천)

---

## 한눈에 보는 Manifest 방식
- **입문자/소규모 서비스에 최적**
- 재사용·환경분리가 필요하다면 다른 방식 고려  
- ‘난 복잡한 것 없이 YAML만 깔끔하게 쓴다!’라면 Manifest부터 시작하세요
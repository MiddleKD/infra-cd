# k8s 배포 정의 방식 - Helm

## [k8s Helm 설치 가이드](helm_install_guide.md)
## [k8s Custom Helm Chart 의존성 관리](helm_chat_dependency.md)

## Helm이란?
Helm은 Kubernetes 리소스를 배포할 때 템플릿과 변수 활용이 가능한 패키지 관리자입니다. 복잡한 환경 변수, 재사용, 패키징에 최적화되어 있습니다.

---

## 특징
- 템플릿 엔진 사용(`.yaml`에 변수 주입)
- 다양한 환경에 맞춰 반복적 YAML 생성/관리
- Chart 단위(패키지 구조)로 배포

- **Helm = Kubernetes용 패키지/템플릿 시스템**
- 변수와 템플릿 문법을 활용, 여러 환경에 맞는 YAML 파일을 반복 생성
- 주요 구조:
  ```
  mychart/
  ├── Chart.yaml         # 설명 및 버전 정보
  ├── values.yaml        # 변수 파일
  └── templates/         # 실제 리소스 템플릿(YAML, 변수포함)
  ```
- 보통 "{{ .Values.xxx }}" 식으로 변수 치환

#### Helm Chart 생성 예시
1. 아래 구조로 파일 준비
    - Deployment/Service 등은 `templates/` 디렉터리 아래에 작성
2. 변동되는 값은 `values.yaml`에, 고정 로직은 템플릿에

---

## 장점과 단점

| 항목     | 설명                      |
| ------ | ----------------------- |
| **장점** | 변수화, 재사용, 생태계(Chart 공유) |
| **단점** | 러닝커브, 구조의 복잡성           |

---
## 사용 흐름
### Helm
1. **템플릿** 파일(`deployment.yaml` 등)에 변수를 심어둔다.
2. 실제 배포 시 `values.yaml` 값으로 치환, 다양한 환경에 맞게 생성
3. `helm install mychart .` 식으로 배포
4. **재귀적으로 yaml 파일 탐색**

---

## 디렉토리/파일 구조 예시

```plaintext
infra-gitops/
└── helm/
    └── hello-server/
        ├── Chart.yaml
        ├── values.yaml
        └── templates/
            ├── deployment.yaml
            └── service.yaml
```

---

## 주요 파일 예제

### Chart.yaml
```yaml
apiVersion: v2
name: hello-server
version: 0.1.0
```

### values.yaml
```yaml
replicaCount: 2
image:
  repository: yourdockerhub/hello-server
  tag: latest
  pullPolicy: IfNotPresent
service:
  port: 80
```

### templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
        - name: hello
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.port }}
```

### templates/service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
spec:
  selector:
    app: {{ .Chart.Name }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.port }}
  type: ClusterIP
```

---

## 배포 방법

1. Helm 설치 및 Chart 준비
2. values.yaml 등 환경에 따라 변수 수정
3. 다음 명령어로 배포

```bash
helm install hello-server ./helm/hello-server
```
- 값 커스터마이징 시:  
  `helm install hello-server ./helm/hello-server -f custom-values.yaml`

## ArgoCD

```bash
argocd app create hello-server-helm \
  --repo https://github.com/yourname/infra-gitops \
  --path helm/hello-server \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set image.tag=latest \
  --sync-policy automated
```

---

## 실무 활용 팁
- 여러 환경(개발/운영 등) 또는 서비스별 변수화가 필요할 때 이상적
- Chart 패키징/공유로 대규모 배포 운영에 편리

---

## 한눈에 보는 Helm 방식
- **복잡한 배포 자동화·재사용에 최적**
- YAML만의 한계(중복, 환경분리)를 극복  
- “반복 업무를 줄이고, 배포를 유연하게!”라면 Helm이 정답
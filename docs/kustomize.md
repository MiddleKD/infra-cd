# k8s 배포 정의 방식 - Kustomize

## Kustomize란?
Kustomize는 Kubernetes 리소스(YAML)에 ‘덧입히기(Overlay)’ 방식으로, 환경별 설정만 손쉽게 다르게 적용할 수 있는 도구입니다.  
템플릿이나 변수 문법이 아닌, “patch”와 “디렉토리 구조”만으로 배포 환경을 나눕니다.

---

## 특징
- 기본(base) 리소스를 overlay로 ‘부분만 변경’
- 환경별 patch, 기존 YAML 재사용에 특화
- 템플릿 문법 없이도 탄력적 배포 가능

> base는 "리소스 인터페이스"가 아님  
> 추상화 ❌ / 설정 변경 ✅

- **base**: 실제로 배포 가능한 **완성된 리소스 집합**  
  (base만으로도 서비스 배포가 가능, 일종의 인터페이스가 아님)
- **overlay**: base를 가져와 **필요한 설정만 수정**  
  (ex: replicas, image, env 등)

---

## 장점과 단점

| 항목     | 설명                            |
| -------- | ----------------------------- |
| **장점** | 환경 분리, 기존 YAML 재사용         |
| **단점** | 제한적인 변수화, 복잡한 구조 시 불편 |

---
## 사용 흐름
### Kustomize
1. **base**에 공통 yaml, **overlays**에 환경별 patch
2. 배포 시 `kustomize build overlays/dev | kubectl apply -f -`
3. 환경마다 변경점만 Patch로 관리
4. **재귀적으로 yaml 파일 탐색**

---
## 디렉토리/파일 구조 예시

```plaintext
infra-gitops/
└── kustomize/
    ├── base/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── kustomization.yaml
    └── overlays/
        └── dev/
            ├── deployment-patch.yaml
            └── kustomization.yaml
```
- `base/`: 모든 환경에 공통되는 리소스
- `overlays/`: 환경별로 덮어쓸 patch & kustomization.yaml

---

## 주요 파일 예시

### base/deployment.yaml
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

### base/service.yaml
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

### base/kustomization.yaml
```yaml
resources:
  - deployment.yaml
  - service.yaml
```

### overlays/dev/deployment-patch.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-server
spec:
  replicas: 2
```

### overlays/dev/kustomization.yaml
```yaml
resources:
  - ../../base
patchesStrategicMerge:
  - deployment-patch.yaml
```

---

## 배포 방법

1. base와 overlays 디렉토리 구성
2. 변경점은 overlay(예: dev)의 patch 파일에 작성
3. 아래 명령어로 배포

```bash
kubectl apply -k kustomize/overlays/dev
```

또는
```bash
kustomize build kustomize/overlays/dev | kubectl apply -f -
```

## ArgoCD
```bash
argocd app create hello-server-kustomize \
  --repo https://github.com/yourname/infra-gitops \
  --path kustomize/overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```

---

## 실무 활용 팁
- 환경별로 극단적으로 분리된 설정에 강점
- 단일 소스(base) 유지, 환경만 Patch로 관리
- 대량 리소스 수정, 변수처리가 필요하면 Helm도 고려

---

## Kustomize 한눈에 정리
- **base/overlay**로 환경별 YAML 덧씌우기
- 반복 배우기보단, 실전 디렉토리 설계와 patch 구조 고민이 핵심  
- “동일 서비스에 환경별 살짝 다른 배포”엔 Kustomize가 최고

---

## Kustomize Overlay에서 Patch 적용 원리 정리

### Overlay에서 base 리소스 patch 방식

- overlay(예: `overlays/dev`)의 `kustomization.yaml`에서 base(공통 리소스 디렉토리)를 `resources`로 지정하면,
  - base 내 `kustomization.yaml`에 정의된 모든 리소스(YAML)를 overlay에서 불러옵니다.
- `patchesStrategicMerge`에 patch 파일을 지정하면,
  - patch 파일의 `kind`(리소스 종류)와 `metadata.name`(이름)이 base의 리소스와 **일치하는 경우에만** 해당 리소스에 patch가 적용됩니다.
  - 일치하지 않으면 patch는 무시됩니다.

#### 예시

```yaml
# overlays/dev/kustomization.yaml
resources:
  - ../../base
patchesStrategicMerge:
  - deployment-patch.yaml
```
- 위 예시에서 `../../base` 아래의 Deployment, Service 등이 overlay(dev)로 포함됩니다.
- `deployment-patch.yaml`의 kind와 metadata.name이 Deployment, hello-server라면,
  - base의 hello-server Deployment만 덮어써서(dev에 맞는 값으로) 생성됩니다.
- Service 등 다른 리소스는 patch가 없다면 base 복사본 그대로 사용합니다.

---
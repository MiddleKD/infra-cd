# k8s Custom Helm Chart 의존성 관리

>**Helm Chart.yaml에서 `namespace: namespace`는 동작하지 않습니다. 다 default에 설치됨.**

## 1. NGINX Ingress Controller 설치

클러스터 외부 요청을 서비스로 라우팅하기 위해 NGINX Ingress Controller를 설치합니다.

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

---

## 2. Helm Chart 의존성 관리

차트 의존성은 `Chart.yaml`의 `dependencies` 필드에 정의합니다.

```yaml
# Chart.yaml
dependencies:
  - name: ingress-nginx
    version: "4.9.1"
    repository: "https://kubernetes.github.io/ingress-nginx"
  - name: cert-manager
    version: "1.13.2"
    repository: "https://charts.jetstack.io"
```

의존성 차트 다운로드:
```bash
helm dependency update
```

> dependencies 추가 후 반드시 `helm dependency update`를 실행해야  
> `helm install` 시 의존 차트도 함께 배포됩니다.


---

## 3. Helm 템플릿에서 하이픈(-) 키 접근 오류 및 해결
### ⛔ 문제 상황
`values.yaml`의 키에 하이픈이 있을 때, dot(`.`) 표기법을 그대로 사용하면 템플릿 파싱 오류가 발생합니다.
**오류 메시지 예시**
```bash
Error: INSTALLATION FAILED: parse error at (...): bad character U+002D '-'
```

**잘못된 예시**
```yaml
image: {{ .Values.image.mcp-context7.repository }}:{{ .Values.image.mcp-context7.tag }}

```

### ✅ 해결 방법: `index` 함수 사용

하이픈이 포함된 키는 반드시 `index` 함수로 접근해야 합니다.
**수정 예시**
```yaml
image: {{ index .Values.image "mcp-context7" "repository" }}:{{ index .Values.image "mcp-context7" "tag" }}
imagePullPolicy: {{ index .Values.image "mcp-context7" "pullPolicy" }}
```

**values.yaml 예시**
```yaml
image:
  mcp-context7:
    repository: your-repo/mcp-context7
    tag: latest
    pullPolicy: IfNotPresent
```

---

## 4. Tips

- 가능하면 하이픈 대신 언더스코어(`_`)나 카멜케이스(`camelCase`) 등 호환 표기를 고려하세요.
- 하이픈이 필요하다면 항상 `index`로 접근하세요.

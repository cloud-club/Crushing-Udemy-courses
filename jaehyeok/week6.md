## 📈 1. Configuring Application Reliability

### 1.1 Health Probes (Liveness, Readiness, Startup)

* Probe는 컨테이너의 상태를 주기적으로 확인하여 오작동, 부팅 지연 등을 감지
* **유형**

  * **Liveness**: 비정상 상태 감지 시 컨테이너 재시작
  * **Readiness**: 서비스 트래픽 수신 가능 여부 판단
  * **Startup**: 초기 부팅이 완료될 때까지 다른 프로브 연기
* **예시**

  ```yaml
  livenessProbe:
    httpGet:
      path: /healthz
      port: 8080
    initialDelaySeconds: 30
    periodSeconds: 10
  readinessProbe:
    tcpSocket:
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
  startupProbe:
    exec:
      command: ["cat", "/tmp/started"]
    failureThreshold: 30
    periodSeconds: 10
  ```

---

### 1.2 Reserve and Limit Application Compute Capacity

* Pod 수준의 CPU 및 메모리 요청(request)과 제한(limit)을 설정하여 안정성 및 스케줄링 제어
* **예시**

  ```yaml
  resources:
    requests:
      cpu: "250m"
      memory: "512Mi"
    limits:
      cpu: "500m"
      memory: "1Gi"
  ```

---

### 1.3 Scaling Applications

* **수평 자동 확장(Horizontal Pod Autoscaler)**: 메트릭에 따라 Pod 개수를 자동 조절
* **수동 스케일링**

  ```bash
  oc scale deployment/myapp --replicas=5
  ```
* **클러스터 머신 확장**: MachineSets, ClusterAutoscaler 사용 권장

---

## 🔄 2. Managing Application Updates

### 2.1 Managing Image Streams

* 이미지 태그를 관리하는 추상화된 리소스. 내부 레지스트리 및 외부 레지스트리를 연결하면서 태그, 롤백, 권한 관리 가능
* **YAML 예시**

  ```yaml
  apiVersion: image.openshift.io/v1
  kind: ImageStream
  metadata:
    name: myapp
  spec:
    lookupPolicy:
      local: false
  ```

---

### 2.2 Using Tags and Digests to Identify Images

* **Tag**: 변경 가능성이 높은 태그(예: latest) 사용 시 불확실성 존재
* **Digest (SHA)**: 고유 이미지 식별자로 안정적 롤백 가능

---

### 2.3 Using Triggers to Manage Images

* **자동 롤아웃**: imagestream 변경 시 자동으로 Build/Deployment 재실행
* **예시**

  ```bash
  oc set triggers deployment/myapp --from-image=myapp:latest -c mycontainer
  ```

---

### 2.4 Rolling Back Failed Deployments

* **롤백 기준**: 실패 시 이전 리비전을 SHA 기반으로 롤백
* **명령어**

  ```bash
  oc rollout undo deployment/myapp
  ```

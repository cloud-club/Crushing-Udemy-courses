## 1. 🚀 Manage Storage for Application Configuration and Data

### 1.1 Secrets 생성 및 사용

* 비밀번호, API 키 등 민감한 정보를 안전하게 저장하고 Pod에 주입
* **생성 방법**

  ```bash
  oc create secret generic my-secret --from-literal=username=admin --from-literal=password=s3cret
  ```
* **사용 예**

  * 환경 변수 주입

    ```yaml
    env:
      - name: DB_USER
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: username
    ```
  * 볼륨으로 마운트

    ```yaml
    volumeMounts:
      - name: secret-vol
        mountPath: /etc/secret
    volumes:
      - name: secret-vol
        secret:
          secretName: my-secret
    ```

---

### 1.2 ConfigMaps 생성 및 사용

* 설정값을 코드와 분리해서 관리하며 Pod에 주입
* **생성 방법**

  ```bash
  oc create configmap my-config --from-literal=LOG_LEVEL=DEBUG
  ```
* **사용 예**

  * 환경 변수 주입

    ```yaml
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: LOG_LEVEL
    ```
  * 볼륨으로 마운트

    ```yaml
    volumes:
    - name: config-vol
      configMap:
        name: my-config
    ```

---

### 1.3 영구 스토리지(Persistent Storage)

* Pod 재시작이나 재스케줄 시에도 데이터 유지가능한 스토리지 제공
* **구성 요소**

  * **PersistentVolume(PV)**: 실제 스토리지 자원
  * **PersistentVolumeClaim(PVC)**: Pod가 요청하는 스토리지
* **예시**

  PVC 선언

  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-pvc
  spec:
    accessModes: [ReadWriteOnce]
    resources:
      requests:
        storage: 10Gi
  ```

  Pod 내 사용

  ```yaml
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: my-pvc
  volumeMounts:
    - name: data
      mountPath: /data
  ```

---

### 1.4 Storage Classes

* 다양한 스토리지 유형(AWS EBS, Azure Disk 등)을 동적으로 프로비저닝 제공
* **예시**

  ```yaml
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: fast-ssd
  provisioner: kubernetes.io/aws-ebs
  parameters:
    type: io1
    iopsPerGB: "10"
  volumeBindingMode: WaitForFirstConsumer
  reclaimPolicy: Delete
  ```
* **PVC에서 사용**

  ```yaml
  spec:
    storageClassName: fast-ssd
    ...
  ```

---

### 1.5 StatefulSets

* 고유한 네트워크 ID와 영구저장소가 필요한 상태 유지 서비스에 사용
* **구성**

  * headless Service 및 volumeClaimTemplates 포함
* **예시**

  ```yaml
  kind: StatefulSet
  spec:
    serviceName: "nginx"
    replicas: 3
    template:
      metadata:
        labels: {app: nginx}
    volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
  ```

---

## 2. 🔐 Configure Network Security

### 2.1 네트워크 구성 요소 설정

* **Ingress Operator**는 HAProxy 기반 Ingress Controller를 배포하여 외부 접근 지원 
* **Config.openshift.io/Ingress 리소스**로 도메인·TLS 설정 정의 가능

---

### 2.2 외부 Route 생성 및 수정

* 클러스터 외부에서 서비스 접근 가능하도록 경로 생성
* **HTTP Route 예시**

  ```bash
  oc expose svc my-service
  ```

  YAML 예

  ```yaml
  apiVersion: route.openshift.io/v1
  kind: Route
  metadata:
    name: my-route
  spec:
    host: www.example.com
    to:
      kind: Service
      name: my-service
    port:
      targetPort: 8080
  ```

---

### 2.3 TLS 인증서로 트래픽 암호화

* **내부 wildcard 인증서**는 `.apps.<domain>` 도메인 자동 제공
* **커스텀 인증서 적용 예**

  1. TLS 비밀정보(secret 생성)
  2. ingresscontroller CR 배포 시 `defaultCertificate`로 참조

---

### 2.4 Ingress 제어

* **Route**와 **Ingress** 리소스를 통해 외부 트래픽 관리함
* **Ingess Controller 설정**

  * `endpointPublishingStrategy`, TLS 프로파일, shard 지정
  * `clientTLS` 설정 통한 mutual TLS 등 구성 가능 
* 서로 다른 Route를 컨트롤러에 매핑하거나 네임스페이스·노드 셰어딩 가능

---

## ✅ 요약

| 영역                            | 핵심 요약                                    |
| ----------------------------- | ---------------------------------------- |
| **Secrets/ConfigMaps**        | 민감/설정값 안전 보관 및 Pod 주입                    |
| **Persistent Storage/PVC/PV** | 데이터 영속성 확보                               |
| **StorageClass**              | 다양한 스토리지 자동 할당                           |
| **StatefulSet**               | 상태 유지 서비스 배포용 컨트롤러                       |
| **Route/Ingress/TLS**         | 외부 서비스 노출 및 암호화                          |
| **Ingress Controller 설정**     | LoadBalancer, TLS, sharding 등 고급 네트워크 제어 |

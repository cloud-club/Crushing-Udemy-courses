## Deploy Applications

### 1. Resource Manifest를 통한 애플리케이션 배포

YAML 형식의 Kubernetes 매니페스트 파일을 사용하여 OpenShift 클러스터에 애플리케이션을 배포

* **절차**

  1. 배포할 리소스(예: Deployment, Service 등)의 매니페스트 파일을 작성
  2. `oc apply -f <파일명>.yaml` 명령어를 사용하여 클러스터에 적용

* **예시**

```bash
  oc apply -f deployment.yaml
```



### 2. 이미지, OpenShift 템플릿, Helm 차트를 통한 애플리케이션 배포

* **컨테이너 이미지**: `oc new-app` 명령어를 사용하여 이미지로부터 애플리케이션을 생성

```bash
  oc new-app <이미지 경로>
```



* **OpenShift 템플릿**: 사전 정의된 템플릿을 사용하여 애플리케이션을 배포

```bash
  oc new-app --template=<템플릿 이름>
```



* **Helm 차트**: Helm을 사용하여 복잡한 애플리케이션을 배포

```bash
  helm install <릴리스 이름> <차트 경로>
```



### 3. 일회성 작업을 위한 Job 배포

Job은 특정 작업을 한 번 실행하고 완료되는 리소스입니다.

* **예시 매니페스트**

```yaml
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: example-job
  spec:
    template:
      spec:
        containers:
        - name: example
          image: busybox
          command: ["echo", "Hello, OpenShift!"]
        restartPolicy: Never
```



* **배포 명령어**

```bash
  oc apply -f job.yaml
```



### 4. 애플리케이션 배포 관리

* **Deployment**: 애플리케이션의 배포 및 롤백을 관리하는 리소스

* **주요 명령어**

  * 배포 상태 확인

    ```bash
    oc rollout status deployment/<배포 이름>
    ```

  * 롤백

    ```bash
    oc rollout undo deployment/<배포 이름>
    ```

### 5. ReplicaSet

지정된 수의 파드를 유지하도록 보장하는 리소스입니다. 일반적으로 Deployment에 의해 관리

* **명령어**

```bash
  oc get rs
```



### 6. 라벨과 셀렉터 활용

* **라벨(Label)**: 리소스에 키-값 쌍을 부여하여 식별 및 분류에 사용

* **셀렉터(Selector)**: 특정 라벨을 가진 리소스를 선택하는 데 사용

* **예시**

```bash
  oc get pods -l app=myapp
```



### 7. 서비스 구성

파드 집합에 대한 안정적인 접근을 제공하는 리소스

* **서비스 유형**

  * **ClusterIP**: 클러스터 내부에서만 접근 가능
  * **NodePort**: 노드의 특정 포트를 통해 외부에서 접근 가능
  * **LoadBalancer**: 외부 로드 밸런서를 통해 접근 가능

* **예시 매니페스트**

```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    selector:
      app: myapp
    ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
    type: ClusterIP
```



### 8. HTTP 및 비HTTP 애플리케이션 외부 노출

* **HTTP 애플리케이션**: Route를 사용하여 외부에 노출

```bash
  oc expose svc/<서비스 이름>
```



* **비HTTP 애플리케이션**: NodePort 또는 LoadBalancer 서비스를 사용하여 외부에 노출

```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    selector:
      app: myapp
    ports:
      - protocol: TCP
        port: 6379
        targetPort: 6379
    type: NodePort
```



### 9. MetalLB 및 Multus

* **MetalLB**: 베어메탈 환경에서 LoadBalancer 서비스를 구현하기 위한 솔루션

  * **설치 및 구성**

    1. MetalLB Operator 설치
    2. 주소 풀(Address Pool) 구성
    3. 서비스에 LoadBalancer 유형 지정

* **Multus**: 파드에 여러 네트워크 인터페이스를 할당할 수 있도록 지원하는 CNI 플러그인

  * **활용 예시**

    * 데이터 플레인과 컨트롤 플레인을 분리한 네트워크 구성
    * 특정 네트워크 요구사항이 있는 애플리케이션 배포
    * 참고 링크 : https://docs.redhat.com/ko/documentation/red_hat_openshift_data_foundation/4.16/html/planning_your_deployment/multi-network-plugin-multus-support_rhodf

  * 예시
    
    1. 추가 네트워크 정의 (NetworkAttachmentDefinition), NAD
    ```yaml
    apiVersion: "k8s.cni.cncf.io/v1"
    kind: NetworkAttachmentDefinition
    metadata:
    name: example-network
    namespace: your-namespace
    spec:
    config: '{
        "cniVersion": "0.3.1",
        "type": "bridge",
        "bridge": "br0",
        "ipam": {
        "type": "host-local",
        "subnet": "10.1.1.0/24",
        "rangeStart": "10.1.1.10",
        "rangeEnd": "10.1.1.100",
        "routes": [{"dst": "0.0.0.0/0"}],
        "gateway": "10.1.1.1"
        }
    }'
    ```
    * [OpenShift에서 지원하는 CNI 플러그인 종류](https://docs.redhat.com/en/documentation/openshift_container_platform/4.12/html/networking/multiple-networks)
        - bridge: 동일한 호스트에 있는 파드들이 서로 및 호스트와 통신할 수 있도록 브리지 기반의 추가 네트워크를 구성합니다.
        - host-device: 호스트 시스템의 물리적 이더넷 네트워크 장치에 파드가 접근할 수 있도록 host-device 기반의 추가 네트워크를 구성합니다.
        - ipvlan: ipvlan 기반의 추가 네트워크를 구성하여, 호스트에 있는 파드들이 다른 호스트 및 해당 호스트의 파드들과 통신할 수 있도록 합니다. 이는 macvlan 기반 네트워크와 유사하지만, macvlan과 달리 각 파드는 부모 물리 네트워크 인터페이스와 동일한 MAC 주소를 공유합니다.
        - macvlan: macvlan 기반의 추가 네트워크를 구성하여, 파드가 물리 네트워크 인터페이스를 통해 다른 호스트 및 해당 호스트의 파드들과 통신할 수 있도록 합니다. macvlan에 연결된 각 파드에는 고유한 MAC 주소가 부여됩니다.
        - SR-IOV: 호스트 시스템에 SR-IOV를 지원하는 하드웨어가 있을 경우, 파드가 가상 기능(VF, Virtual Function) 인터페이스에 직접 연결되도록 하는 SR-IOV 기반의 추가 네트워크를 구성합니다.

    2. 파드에 추가 네트워크 인터페이스 연결
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: example-pod
    annotations:
        k8s.v1.cni.cncf.io/networks: example-network
    spec:
    containers:
    - name: example-container
        image: your-image
    ```
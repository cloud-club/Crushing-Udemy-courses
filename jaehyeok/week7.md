## 🔐 1. Configuring Application Security

### 1.1 서비스 어카운트(Service Accounts) 생성 및 구성

* 사용자 계정이 아닌, 파드나 서비스가 API에 접근할 수 있게 하는 자동화 전용 계정

* **기본 제공 서비스 계정**
  * `builder`, `deployer`, `default` 등이 네임스페이스에 자동 생성

* **생성 및 권한 부여**
  ```bash
  oc create sa my-sa
  oc adm policy add-role-to-user edit system:serviceaccount:<project>:my-sa
  ```

  * `edit` 권한을 부여하여 리소스 생성·수정 등 수행 가능 

* **API 토큰 사용**

  Secret 형태로 자동 생성되며, 외부 앱에서도 인증용으로 사용 가능

---

### 1.2 Security Context Constraints (SCC)로 권한 관리

* 파드가 실행될 수 있는 보안 컨텍스트(UID, SELinux, 네트워크, 볼륨 등) 규칙을 정의하는 OpenShift 고유 리소스

* **기본 SCC 목록**

  * `restricted` (기본): 최소 권한
  * `nonroot`, `anyuid`, `hostnetwork`, `hostaccess`, `privileged` 등

* **사용 예**

  ```bash
  oc adm policy add-scc-to-user anyuid -z my-sa -n myproject
  ```

  * 특정 서비스 계정에 특정 SCC 권한 부여

* **SCC 직접 생성 예**

  ```yaml
  kind: SecurityContextConstraints
  apiVersion: security.openshift.io/v1
  metadata:
    name:scc-admin
  allowPrivilegedContainer: true
  runAsUser:
    type: RunAsAny
  users:
  - system:serviceaccount:myproject:my-sa
  ```

  ````bash
  oc create -f scc-admin.yaml
  ````

---

### 1.3 Privileged 애플리케이션 실행

* 호스트 네트워킹, 마운트, 루트 권한 사용 등을 가능하게 하는 고권한 SCC 사용
* **위험 요소**

  클러스터 보안에 매우 민감하므로 최소한으로 제한

* **권한 부여**

  `privileged`, `hostaccess`, `hostnetwork`와 같은 SCC를 사용하며, RBAC을 통해 자신의 서비스 계정에만 권한을 부여

---

### 1.4 K8s API 접근 구성

* 서비스 계정에 적절한 권한(RBAC Role 및 RoleBinding)을 부여함으로써,

*  파드 내부 애플리케이션이 Kubernetes API(예: pod 상태 조회, configmap CRUD 등)에 접근 가능

* **예시**

  ```bash
  oc create rolebinding sa-edit --role=edit \
    --serviceaccount=myproject:my-sa -n myproject
  ```

---

## ⚙️ 2. Managing Operators

### 2.1 모듈 소개

* Operator란 Kubernetes 클러스터에서 특정 애플리케이션의 설치·업데이트·백업·롤백 등을 자동화하는 컨트롤러 
* OpenShift에서는 OperatorHub를 통해 쉽게 사용 가능
---

### 2.2 OpenShift Operator 개요

* Kubernetes CRD 기반으로 작동하며, 특정 기능이나 애플리케이션을 자동으로 운영
* OpenShift UI에서 심플하게 설치·제거·관리 가능

---

### 2.3 Operator Ecosystem

* Red Hat 및 커뮤니티 제공 Operator 포함
* Storage, Database, Monitoring, Networking 등 광범위한 영역에서 사용 가능

---

### 2.4 Operator 유형

1. **Cluster-scoped**: 클러스터 전체를 타겟으로 기능 제공
2. **Namespace-scoped**: 특정 네임스페이스 범위에서만 동작함

* 일반적으로 권한 분리를 위해 네임스페이스 스코프 사용 권장

---

### 2.5 Operator 설치

* **UI 사용 예**: OperatorHub에서 Operator 선택 후 설치
* **CLI 예**

  ```bash
  oc create -f subscription.yaml
  oc create -f operatorgroup.yaml
  ```
* 설치 시 CRD, ClusterRole/CRB, Deployment 등이 자동 생성됨

---

### 2.6 Operator 제거

* UI에서 "Uninstall" 클릭 또는
* **CLI 예**

  ```bash
  oc delete subscription <name>
  oc delete operatorgroup <namespace>
  ```

  관련 리소스를 자동 정리
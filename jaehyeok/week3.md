[![Red Hat OpenShift – Configuring an htpasswd identity provider - vMattroman](https://tse4.mm.bing.net/th?id=OIP.p2z8qQFdw5Vcw9IGWO3dawHaEC\&pid=Api)](https://vmattroman.com/red-hat-openshift-configuring-an-htpasswd-identity-provider/)

## 🔐 Manage Authentication and Authorization

### 1. HTPasswd Identity Provider 구성

HTPasswd는 OpenShift에서 로컬 사용자 인증을 위해 사용하는 간단한 파일 기반 인증 제공자

* **설정 절차**

  1. `httpd-tools` 패키지를 설치하여 `htpasswd` 명령어를 사용할 수 있도록 함

  2. 사용자 이름과 비밀번호를 포함한 `htpasswd` 파일을 생성

     ```bash
     htpasswd -c -B -b users.htpasswd <username> <password>
     ```

  3. 생성된 `users.htpasswd` 파일을 OpenShift 클러스터에 업로드하고, OAuth 설정을 통해 HTPasswd Identity Provider를 구성

  4. 구성이 완료되면, 해당 사용자는 OpenShift Web Console이나 CLI를 통해 로그인 가능

### 2. 사용자 생성 및 삭제

* **사용자 생성**: HTPasswd 파일에 새로운 사용자를 추가

```bash
  htpasswd -B -b users.htpasswd <new_username> <new_password>
```



* **사용자 삭제**: HTPasswd 파일에서 사용자를 제거

```bash
  htpasswd -D users.htpasswd <username>
```



* 변경된 `users.htpasswd` 파일을 클러스터에 반영하기 위해 Secret을 업데이트하거나 재적용해야 함

### 3. 사용자 비밀번호 변경 

* 기존 사용자의 비밀번호를 변경하려면, `htpasswd` 명령어를 사용하여 해당 사용자의 비밀번호를 업데이트

```bash
  htpasswd -B -b users.htpasswd <username> <new_password>
```



* 변경 사항을 적용하기 위해 Secret을 업데이트하거나 재적용해야 함

### 4. 그룹 생성 및 관리

* **그룹 생성**: OpenShift CLI를 사용하여 새로운 그룹을 생성

```bash
  oc adm groups new <group_name>
```



* **사용자 그룹에 추가**: 기존 그룹에 사용자를 추가

```bash
  oc adm groups add-users <group_name> <username>
```



* **그룹 정보 확인**: 그룹의 상세 정보를 조회

```bash
  oc get group <group_name> -o yaml
```



### 5. 사용자 및 그룹 권한 수정

* **역할 바인딩 생성**: 사용자 또는 그룹에 특정 역할을 부여

```bash
  oc adm policy add-role-to-user <role> <username> -n <project>
  oc adm policy add-role-to-group <role> <group_name> -n <project>
```



* **역할 바인딩 제거**: 사용자 또는 그룹에서 특정 역할을 제거

```bash
  oc adm policy remove-role-from-user <role> <username> -n <project>
  oc adm policy remove-role-from-group <role> <group_name> -n <project>
```



* 역할(Role)과 역할 바인딩(RoleBinding)을 통해 세분화된 접근 제어를 구현할 수 있음

---

## ⚙️ Enable Developer Self-Service

### 1. 프로젝트 리소스 쿼터 설정

프로젝트별로 리소스 사용량을 제한하여 클러스터 자원의 과도한 소비를 방지

* **리소스 쿼터 생성**: `ResourceQuota` 객체를 정의하여 적용

```yaml
  apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: example-quota
  spec:
    hard:
      pods: "10"
      requests.cpu: "4"
      requests.memory: 8Gi
      limits.cpu: "8"
      limits.memory: 16Gi
```



적용 명령어

```bash
  oc apply -f quota.yaml -n <project>
```



### 2. 클러스터 리소스 쿼터 설정

여러 프로젝트에 걸쳐 리소스 사용량을 제한하기 위해 `ClusterResourceQuota`를 사용

* **클러스터 리소스 쿼터 생성**: `ClusterResourceQuota` 객체를 정의하여 적용

```yaml
  apiVersion: quota.openshift.io/v1
  kind: ClusterResourceQuota
  metadata:
    name: example-cluster-quota
  spec:
    quota:
      hard:
        pods: "20"
        requests.cpu: "10"
        requests.memory: 20Gi
    selector:
      labels:
        matchLabels:
          team: dev
```



적용 명령어

```bash
  oc apply -f cluster-quota.yaml
```



### 3. 프로젝트 리밋 레인지 설정

`LimitRange`를 사용하여 컨테이너의 기본 및 최대 리소스 사용량을 제한

* **리밋 레인지 생성**: `LimitRange` 객체를 정의하여 적용

```yaml
  apiVersion: v1
  kind: LimitRange
  metadata:
    name: example-limitrange
  spec:
    limits:
    - default:
        cpu: 500m
        memory: 512Mi
      defaultRequest:
        cpu: 250m
        memory: 256Mi
      type: Container
```



적용 명령어

```bash
  oc apply -f limitrange.yaml -n <project>
```



### 4. 프로젝트 템플릿 구성

프로젝트 생성 시 기본적으로 적용할 리소스 구성을 정의하여 일관된 환경을 제공

* **템플릿 생성 및 적용**

  1. 템플릿 YAML 파일을 작성

  2. 템플릿을 `openshift-config` 네임스페이스에 생성

     ```bash
     oc create -f project-template.yaml -n openshift-config
     ```

  3. 클러스터 설정에서 `projectRequestTemplate`을 업데이트하여 새로운 프로젝트 생성 시 해당 템플릿이 적용

## Manage OpenShift Container Platform

### 1. Web Console

* OpenShift Web Console은 프로젝트 데이터를 시각화하고 관리 작업을 수행할 수 있는 그래픽 사용자 인터페이스(GUI)를 제공

* **기능**

  * 프로젝트 및 리소스 관리
  * 애플리케이션 배포 및 모니터링
  * 클러스터 상태 확인
  * 로그 조회 및 문제 해결

### 2. OpenShift CLI

* `oc` 명령어를 사용하는 OpenShift CLI는 터미널에서 애플리케이션을 생성하고 OpenShift 프로젝트를 관리할 수 있는 도구

* **주요 명령어**

  * `oc new-app`: 애플리케이션 생성
  * `oc get`: 리소스 조회
  * `oc delete`: 리소스 삭제
  * `oc logs`: 로그 확인

### 3. Query, Format, Filter OpenShift Resources

* `oc get` 명령어를 사용하여 리소스를 조회하고, 출력 형식을 지정하거나 필터링할 수 있음

* **예시**

  * `oc get pods -o wide`: Pod 상세 정보 조회
  * `oc get pods --selector=app=myapp`: 특정 라벨을 가진 Pod 조회

### 4. Import, Export & Configure Kubernetes Resources

* OpenShift에서는 Kubernetes 리소스를 YAML 파일로 가져오거나 내보낼 수 있음

* **명령어**

  * `oc get deployment myapp -o yaml > myapp.yaml`: 배포 리소스 내보내기
  * `oc apply -f myapp.yaml`: 리소스 적용

### 5. Locate and Examine Container Images

* OpenShift의 내부 레지스트리를 통해 컨테이너 이미지를 관리할 수 있음

* **명령어**

  * `oc get imagestreams`: 이미지 스트림 조회
  * `oc describe imagestream myapp`: 이미지 스트림 상세 정보 확인

### 6. Create and Delete Projects

* 프로젝트는 OpenShift에서 리소스를 그룹화하는 단위

* **명령어**

  * `oc new-project myproject`: 프로젝트 생성
  * `oc delete project myproject`: 프로젝트 삭제

### 7. Examine Resources and Cluster Status

* 클러스터의 리소스 상태와 전체 상태를 확인하여 운영 상태를 모니터링할 수 있음

* **명령어**

  * `oc get nodes`: 노드 상태 확인
  * `oc get pods --all-namespaces`: 전체 네임스페이스의 Pod 상태 확인

### 8. View Logs

* 애플리케이션 및 시스템 로그를 조회하여 문제를 진단할 수 있음

* **명령어**

  * `oc logs mypod`: Pod 로그 확인
  * `oc logs -f mypod`: 실시간 로그 스트리밍

### 9. Assess the Health of an OpenShift Cluster

* 클러스터의 건강 상태를 평가하여 안정적인 운영을 유지할 수 있음

* **명령어**

  * `oc get clusterversion`: 클러스터 버전 및 상태 확인
  * `oc get co`: 클러스터 오퍼레이터 상태 확인

### 10. Troubleshoot Common Problems

* 일반적인 문제를 해결하기 위한 접근 방법과 도구를 활용하여 문제를 진단하고 해결할 수 있음

* **방법**

  * 로그 분석
  * 이벤트 조회
  * 리소스 상태 확인

### 11. Use Documentation

* 공식 문서를 활용하여 OpenShift의 기능과 사용법을 익히고 문제를 해결할 수 있음

* **리소스**

  * [OpenShift 공식 문서](https://docs.openshift.com/)
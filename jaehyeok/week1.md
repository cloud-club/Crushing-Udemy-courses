### 1. Introduction

* **OpenShift 개요**: Red Hat OpenShift는 Kubernetes를 기반으로 한 컨테이너 오케스트레이션 플랫폼으로, 애플리케이션의 개발, 배포, 관리를 자동화

* **강의 목표**: 이 강의는 OpenShift의 핵심 개념을 이해하고, EX280 인증 시험을 준비하는 데 중점을 둠

* **학습 내용**

  * OpenShift의 주요 구성 요소
  * 클러스터 설치 및 구성 방법
  * 애플리케이션 배포 및 관리
  * 보안 및 인증 설정
  * 스토리지 및 네트워크 구성

### 2. OpenShift Flavors

* **OpenShift의 버전**

  * **OpenShift Container Platform (OCP)**: Red Hat의 상용 제품으로, 엔터프라이즈 환경에 적합하며, 공식 지원과 업데이트를 제공

  * **OKD (Origin Community Distribution)**: OpenShift의 커뮤니티 에디션으로, 오픈 소스 기반이며, 최신 기능을 시험해볼 수 있음

  * **OpenShift Local**: 개발자용 로컬 환경으로, 단일 노드 클러스터를 로컬 머신에 설치하여 테스트 및 개발에 활용할 수 있음

  * **MicroShift**: 경량화된 OpenShift로, 엣지 컴퓨팅 및 IoT 환경에 적합

* **버전 비교**

  | 버전                           | 용도            | 특징                |               |
  | ---------------------------- | ------------- | ----------------- | ------------- |
  | OpenShift Container Platform | 엔터프라이즈 환경     | 공식 지원, 안정성, 보안 강화 |               |
  | OKD                          | 커뮤니티 및 테스트 환경 | 오픈 소스, 최신 기능 반영   |               |
  | OpenShift Local              | 로컬 개발 및 테스트   | 단일 노드, 간편한 설치     |               |
  | MicroShift                   | 엣지 및 IoT 환경   | 경량화, 리소스 효율성      | 

### 3. Installation Prerequisites

* **하드웨어 요구 사항**

  * **CPU**: 최소 4개의 물리적 코어
  * **RAM**: 최소 16GB (권장 32GB)
  * **스토리지**: 최소 35GB의 여유 공간

* **소프트웨어 요구 사항**

  * **운영 체제**: Windows 10 이상, macOS 11 이상, 최신 Fedora 또는 RHEL/CentOS 7, 8, 9
  * **필수 패키지**

    * Linux: NetworkManager, libvirt, QEMU (qemu-kvm)
    * macOS: HyperKit 또는 VirtualBox
    * Windows: Hyper-V 또는 VirtualBox

* **기타 요구 사항**

  * Red Hat 계정 및 Pull Secret
  * 인터넷 연결
  * 관리자 권한

### 4. OpenShift Local Installation

* **설치 절차**

  1. [Red Hat OpenShift Local 다운로드 페이지](https://console.redhat.com/openshift/create/local)에서 설치 파일 및 Pull Secret을 다운로드

  2. 압축을 해제하고, 실행 파일을 시스템의 PATH에 추가

  3. 터미널에서 `crc setup` 명령어를 실행하여 초기 설정을 진행

  4. `crc start` 명령어를 실행하여 클러스터를 시작, 이 과정은 약 20분 정도 소요될 수 있음

  5. `crc console` 명령어를 통해 OpenShift 웹 콘솔에 접근할 수 있음

* **로그인 정보**

  * **developer** 사용자: 기본 사용자로, 일반적인 작업에 사용
  * **kubeadmin** 사용자: 관리자 권한을 가진 사용자로, `crc console --credentials` 명령어를 통해 비밀번호를 확인할 수 있음

* **주의 사항**

  * OpenShift Local은 단일 노드 클러스터로, 일부 기능이 제한될 수 있음
  * 업데이트 시 기존 클러스터를 삭제하고 재설치해야 할 수 있음
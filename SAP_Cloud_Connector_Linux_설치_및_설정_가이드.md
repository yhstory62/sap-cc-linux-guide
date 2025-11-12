# SAP Cloud Connector 리눅스 설치 및 설정 가이드

본 문서는 SAP BTP Integration Suite와 온프레미스 시스템을 SAP Cloud Connector로 안전하게 연동하기 ## 5. Subaccount## 6. Backend & 리소스(Path) 공개

### 6.1 Backend 시스템 추가
1) "Cloud To On-Premise" → Backend 추가(Host/IP/Port/Protocol)  

![Cloud To On-Premise 매핑](images/09_cloud_to_onpremise_mapping.png)

![Backend 시스템 추가](images/10_add_backend_system.png)

### 6.2 리소스 경로 공개 설정
2) 필요한 Path/Method만 최소 공개 (Exposed Resources)  

![리소스 경로 공개 설정](images/11_exposed_resources.png)

3) 방화벽/보안 정책으로 Cloud Connector → Backend 접근 허용  
4) 필요 시 Principal Propagation 등 고급 보안 검토) "Add Subaccount" 클릭  
2) Subaccount ID / Region / (Location ID 필요 시) 입력  

![Add Subaccount 다이얼로그](images/07_add_subaccount_dialog.png)

3) OAuth 인증 후 Connected 상태 확인  

![Subaccount 연결 완료](images/08_subaccount_connected.png)

4) Location ID 사용 시 BTP Destination에도 동일 값 설정·설정·운영 절차를 고객사 전달용으로 정리한 가이드입니다.  
(본 버전은 실제 SAP Development Tools 다운로드 화면에서 확인한 버전/파일명을 반영하여 수정되었습니다.)

## 1. 개요
- 목적: 온프레미스 ↔ SAP BTP(Integration Suite) 안전 연결
- 구성: Cloud Connector(온프레미스), BTP Subaccount, Integration Flow
- 기본 흐름:
  1) Cloud Connector 설치 및 기동
  2) Subaccount 연결(Connected)
  3) Backend 시스템 & 리소스(Path/Method) 최소 공개
  4) Flow가 온프레미스 Outbound 호출 시 Destination 사용
  5) 온프레미스 → BTP Inbound만 있으면 Destination 불필요

참고 유튜브:
- 설치/기본 설정: https://youtu.be/P_ov18t2Np0
- Integration Suite 연동 실습: https://youtu.be/eN8zboQdzBs
- SAP Cloud Connector 고급 설정: https://youtu.be/iqZX81jOr3U?t=12
- SAP BTP 연동 실습: https://youtu.be/az8jfGDkuOo

## 2. 사전 준비
| 항목 | 내용 |
|------|------|
| OS | Linux (Ubuntu / Red Hat / SUSE 등) |
| Java | OpenJDK 또는 Oracle JDK 8 이상 |
| 네트워크 | BTP ↔ Cloud Connector ↔ 내부 시스템 통신 가능 |
| 권한 | SAP BTP Subaccount, Integration Suite 서비스 권한, 서버/방화벽 수정 권한 |
| 계정 | SAP 다운로드(S-User 등) 계정 |

## 3. 설치 파일 다운로드

공식 페이지: https://tools.hana.ondemand.com/#cloud  
(해당 페이지에서 Cloud Connector 항목을 확인하고 원하는 OS/아키텍처/버전을 선택)

![SAP Cloud Connector 다운로드 페이지](images/01_sap_cloud_connector_download_page.png)

### 3.1 현재(스크린샷 기준) 확인된 주요 파일 목록 (예시)

| Operating System | Architecture | Version | Portable 여부 | File Name | 포맷 | File Size | 해시 종류(표시 링크) |
|------------------|-------------|---------|---------------|-----------|------|-----------|----------------------|
| Linux | x86_64 | 2.16.2 | 일반 | sapcc-2.16.2-linux-x64.zip | zip | 91.8 MB | sha1 |
| Linux | ppc64le | 2.18.2 | 일반 | sapcc-2.18.2-linux-ppc64le.tar.gz | tar.gz | 114.5 MB | sha1 |
| Linux | x86_64 | 2.18.2 | 일반 | sapcc-2.18.2-linux-x64.zip | zip | 114.5 MB | sha1 |
| Linux (Portable) | ppc64le | 2.18.2 | Portable | sapcc-2.18.2-linux-ppc64le.tar.gz | tar.gz | 117.8 MB | sha1 |
| Linux (Portable) | x86_64 | 2.18.2 | Portable | sapcc-2.18.2-linux-x64.tar.gz | tar.gz | 117.7 MB | sha1 |

비고:
- Portable 버전: 관리자 권한 없이 실행할 수 있는 경우를 위한 아카이브. 업그레이드 경로 제한(기존 버전에서 직접 업그레이드 불가) 가능성 있음.
- zip vs tar.gz: 제공 포맷 차이이며 설치 스크립트/압축 해제 방식이 다를 수 있음(zip은 unzip, tar.gz는 tar 사용).
- 표에 나온 sha1 링크는 무결성 확인용 해시 제공. 내부 보안 표준에 따라 SHA256도 직접 계산해 기록 권장.

### 3.2 파일 선택 가이드
- 신규 설치 & 표준 운영: 최신 일반 버전(Linux x86_64 2.18.2 zip) 권장
- 운영 정책상 Portable 필요: Portable x86_64 tar.gz 선택
- 레거시 환경 호환(예: 이전 Cloud Connector 2.16.x와 연계한 업그레이드 동선 검토): 2.16.2도 확인 가능하나 지원 종료 정책(SAP Note 3302250 등) 고려 후 최신 업그레이드 계획 수립

### 3.3 artifacts 폴더 구조 및 다운로드 파일 관리
본 프로젝트의 `artifacts/` 폴더에는 다음과 같은 구조로 설치 파일들이 저장됩니다:

```
artifacts/
├── README.md                    # artifacts 폴더 설명 문서
├── sapcc-2.16.2-linux-x64.zip  # Cloud Connector 2.16.2 설치 파일
├── sapcc-2.18.2-linux-x64.zip  # Cloud Connector 2.18.2 설치 파일
└── (추가 버전 파일들...)
```

**사용 방법:**
1. SAP 공식 사이트에서 원하는 버전의 설치 파일 다운로드
2. `artifacts/` 폴더에 직접 저장
3. 파일명에 버전 정보가 포함되어 있어 별도 폴더 구분 불필요

## 4. 설치 및 서비스 기동 (선택 4번: 실제 설치 실행 단계)
다운로드가 완료된 후 다음 명령 수행:

### 4.1 설치 파일 압축 해제
```bash
# (zip 포맷 예시)
unzip sapcc-2.18.2-linux-x64.zip -d sapcc-2.18.2
cd sapcc-2.18.2

# (tar.gz 포맷 예시)
tar -xvf sapcc-2.18.2-linux-x64.tar.gz
cd sapcc-2.18.2-linux-x64
```

![설치 파일 압축 해제](images/02_extract_installation_file.png)

### 4.2 설치 스크립트 실행
```bash
# 설치 스크립트 (파일명은 버전에 따라 상이할 수 있음)
chmod +x installCloudConnector.sh
./installCloudConnector.sh
```

![설치 스크립트 실행](images/03_install_script_execution.png)

### 4.3 서비스 기동 및 확인
```bash
# 서비스 시작/확인
systemctl start cloud-connector
systemctl status cloud-connector
systemctl enable cloud-connector   # (선택)
```

![서비스 상태 확인](images/04_service_status_check.png)

### 4.4 웹 콘솔 접속
웹 콘솔: `https://<서버IP>:8443`  
초기 계정: `Administrator / manager`

![웹 콘솔 로그인](images/05_web_console_login.png)

![메인 대시보드](images/06_main_dashboard.png)

## 5. Subaccount 연결
1) “Add Subaccount”  
2) Subaccount ID / Region / (Location ID 필요 시) 입력  
3) OAuth 인증 후 Connected 상태 확인  
4) Location ID 사용 시 BTP Destination에도 동일 값 설정

## 6. Backend & 리소스(Path) 공개
1) “Cloud To On-Premise” → Backend 추가(Host/IP/Port/Protocol)  
2) 필요한 Path/Method만 최소 공개 (Exposed Resources)  
3) 방화벽/보안 정책으로 Cloud Connector → Backend 접근 허용  
4) 필요 시 Principal Propagation 등 고급 보안 검토

## 7. Integration Suite 구성
### 7.1 Outbound (Flow → 온프레미스 호출)
- BTP Cockpit → Destinations → New
  - URL: Cloud Connector 노출 Endpoint
  - Proxy Type: OnPremise
  - Authentication: Basic/OAuth2 등
  - 필요시 Additional Properties: `LocationID`, `sap-client` 등

![BTP Destinations 설정](images/12_btp_destinations.png)

![Destination 상세 설정](images/13_destination_details.png)

### 7.2 Inbound (온프레미스 → Flow 호출)
- 온프레미스 측에서 BTP Flow Endpoint(HTTPS URL)를 직접 호출
- 별도 On-Premise Destination 불필요

### 7.3 Integration Flow 체크 리스트
| 항목 | 확인 내용 |
|------|-----------|
| Receiver 채널 | Outbound 시 Destination 이름 참조 |
| 인증 | HTTPS + 적절한 Auth 헤더 |
| 모니터링 | Trace 활성화 후 메시지 흐름 확인 |

## 8. 테스트 체크리스트
- Cloud Connector Subaccount: Connected
- Backend 리소스 접근/허용 로그 정상
- Destination 호출 응답(Outbound) 정상
- Flow Trace 에러 없음
- Cloud Connector 로그에서 Block/Denied 없음

## 9. 보안/운영
| 주제 | 권장 |
|------|------|
| 패치 | 최신 릴리스 추적(SAP Note & 릴리스 노트) |
| 계정 | 관리자 비밀번호 주기적 교체 |
| 인증서 | TLS 인증서 갱신/만료 관리 |
| 로그 | 백업·보관 주기 설정 |
| 최소공개 | 불필요 Path/Port 비활성화 |
| 무결성 | 설치 파일 해시(SHA256) 검증 |

## 10. 스크린샷 목록
본 문서에서 사용된 스크린샷들:

| 번호 | 파일명 | 설명 | 섹션 |
|------|--------|------|------|
| 01 | 01_sap_cloud_connector_download_page.png | SAP 다운로드 페이지 | 3. 설치 파일 다운로드 |
| 02 | 02_extract_installation_file.png | 설치 파일 압축 해제 | 4.1 압축 해제 |
| 03 | 03_install_script_execution.png | 설치 스크립트 실행 | 4.2 설치 실행 |
| 04 | 04_service_status_check.png | 서비스 상태 확인 | 4.3 서비스 기동 |
| 05 | 05_web_console_login.png | 웹 콘솔 로그인 | 4.4 웹 콘솔 |
| 06 | 06_main_dashboard.png | 메인 대시보드 | 4.4 웹 콘솔 |
| 07 | 07_add_subaccount_dialog.png | Subaccount 추가 다이얼로그 | 5. Subaccount 연결 |
| 08 | 08_subaccount_connected.png | Subaccount 연결 완료 | 5. Subaccount 연결 |
| 09 | 09_cloud_to_onpremise_mapping.png | Cloud To On-Premise 매핑 | 6.1 Backend 추가 |
| 10 | 10_add_backend_system.png | Backend 시스템 추가 | 6.1 Backend 추가 |
| 11 | 11_exposed_resources.png | 리소스 경로 공개 설정 | 6.2 리소스 공개 |
| 12 | 12_btp_destinations.png | BTP Destinations 설정 | 7.1 Outbound 설정 |
| 13 | 13_destination_details.png | Destination 상세 설정 | 7.1 Outbound 설정 |

**참고:** 실제 화면 캡처 시 위 파일명으로 저장하여 이미지를 교체하세요.

## 11. 참고 링크
- [SAP Cloud Connector Help](https://help.sap.com/docs/cloud-connector/latest/en-US/c08de1c9134647c5b47b281b9adc387e.html)
- [SAP BTP 온프레미스 연동](https://help.sap.com/docs/btp/sap-business-technology-platform/connecting-on-premise-systems-to-cloud-applications.html)
- [Integration Suite 문서](https://help.sap.com/docs/integration-suite/)

---
*본 문서는 SAP Cloud Connector 설치 및 설정을 위한 고객사 전달용 가이드입니다.*
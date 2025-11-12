# SAP Cloud Connector & SAP BTP Integration Suite Guide Repository

이 저장소는 고객사 전달용 Cloud Connector 리눅스 설치 및 BTP Integration Suite 연동 가이드를 포함합니다.

## 구조(예시)
```
.
├─ README.md
├─ SAP_Cloud_Connector_Linux_설치_및_설정_가이드.md
├─ images/
│  ├─ 01_download_page.png
│  ├─ 02_install_terminal.png
│  ├─ ...
├─ viewer-screenshots/
│  ├─ typora_preview.png
│  ├─ vscode_markdown_preview.png
└─ artifacts/   # (선택: 다운로드한 설치 파일 저장)
```

## 보기 방법
- GitHub 웹 UI: 자동 렌더링
- VS Code: `Markdown: Open Preview`
- Typora / MarkText / Obsidian: MD 뷰어 및 PDF Export

## 설치 파일 관리
- `artifacts/` 디렉터리에 실제 받은 tar.gz 저장
- 버전/파일명/다운로드 날짜 기록(CHANGELOG 또는 문서 내 표)
- 내부 보안 정책 따라 해시(SHA256) 추가 권장

## 향후 개선
- GitHub Actions로 링크 유효성 검사
- MkDocs로 정적 사이트 배포(`docs/` 폴더 변환)
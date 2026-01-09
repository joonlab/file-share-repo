# 세션 컨텍스트 - 2026-01-09T17:45:00+09:00

## 현재 세션 개요
- **메인 태스크/기능**: MD Share - 마크다운 파일을 웹으로 공유하는 서비스 구축 (Obsidian Share Note 유사)
- **세션 지속 시간**: 약 2시간
- **현재 상태**: 핵심 기능 완료, 이미지 업로드 로직 수정 완료

## 최근 활동 (최근 30-60분)
- **방금 수행한 작업**:
  - 이미지 저장소를 private md-share repo에서 public file-share-repo로 변경
  - push_assets 함수가 호출되지 않던 버그 수정 (ASSETS_ADDED 플래그 문제 → 항상 체크하도록 변경)
  - 이미지 22개 수동 push 완료
- **현재 해결 중인 문제**: Mac 앱 번들 실행 불가 (LSOpenURLsWithCompletionHandler error -1712)
- **작업 중인 파일**: ~/bin/md-share (메인 CLI 스크립트)
- **테스트 상태**: CLI 정상 동작, 앱 번들 실행 안됨

## 주요 기술적 결정 사항
- **아키텍처 결정**:
  - 마크다운 → md-share-repo (private, Vercel 배포)
  - 이미지 → file-share-repo/md-share-assets (public, raw URL 사용)
- **구현 방식**:
  - Next.js 14 + marked + highlight.js로 마크다운 렌더링
  - Bash 스크립트 + Python(이미지 처리) + AppleScript(파일 선택 다이얼로그)
- **기술 선정**: Vercel 배포, GitHub raw URL로 이미지 서빙
- **성능/보안 고려사항**: private repo로 마크다운 보관, public repo로 이미지만 공개

## 코드 컨텍스트
- **수정된 파일**:
  - ~/bin/md-share (CLI 스크립트)
  - ~/md-share-repo/lib/markdown.js (마크다운 렌더링)
  - ~/md-share-repo/styles/globals.css (UI 스타일)
  - ~/md-share-repo/pages/view/[...path].js (뷰어 페이지)
- **새로운 패턴**: NFC 유니코드 정규화로 한글 파일명 처리
- **의존성**: marked, gray-matter, highlight.js, katex
- **설정 변경**: Vercel 환경변수 GITHUB_TOKEN 설정됨

## 현재 구현 상태
- **완료됨**:
  - 마크다운 → HTML 렌더링 (GFM, Obsidian callouts, 코드 하이라이팅, YouTube 임베딩)
  - Frontmatter 파싱 및 표시
  - TOC 자동 생성
  - CLI 도구 (GUI 모드 + 파일 직접 지정)
  - 이미지 자동 업로드 (file-share-repo)
  - 한글 URL 인코딩 (NFC 정규화)
  - Vercel 배포 (https://md-share-joonlab.vercel.app)
- **진행 중**: 없음
- **차단됨**: Mac 앱 번들 실행 (macOS 보안 정책 문제)
- **다음 단계**: 사용자가 CLI로 사용하기로 결정

## 핸드오프를 위한 중요 컨텍스트
- **환경 설정**:
  - ~/md-share-repo - 마크다운 저장소 (private)
  - ~/file-share-repo - 이미지 저장소 (public)
  - ~/bin/md-share - CLI 스크립트
- **실행/테스트 방법**:
  - `~/bin/md-share` - GUI 모드 (파일 선택 창)
  - `~/bin/md-share /path/to/file.md` - 직접 지정
- **알려진 문제**:
  - Mac 앱 번들 실행 안됨 (CLI로 대체)
  - 같은 파일 재업로드 시 기존 파일 삭제 후 새로 업로드
- **외부 의존성**: GitHub API (토큰), Vercel

## 대화 흐름
- **원래 목표**: Obsidian Share Note 같은 마크다운 공유 서비스 구축
- **변천 과정**:
  1. 웹앱 구조 선택 → Vercel + GitHub 방식
  2. Mac 앱 추가 요청 → CLI + AppleScript 파일 선택
  3. 이미지 렌더링 안됨 → private repo 문제 → public repo로 이미지 분리
- **배운 점/인사이트**:
  - macOS NFD 한글 인코딩 → NFC 정규화 필요
  - private repo raw URL은 이미지 렌더링 불가
  - App 번들은 코드 서명/Gatekeeper 문제로 실행 어려움
- **고려했던 대안**:
  - Python tkinter GUI → macOS 호환성 문제로 포기
  - Automator 앱 → 실행 실패
  - CLI로 최종 결정

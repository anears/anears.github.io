---
layout: post
title: Claude Tower — 여러 서버의 Claude Code 세션을 한 화면에서 관리하는 TUI
date: 2026-07-23 16:00:00+0900
description: SSH로 흩어진 Claude Code 세션들을 모아 보고, 채팅을 보내고, 일일 업무 보고까지 뽑아주는 터미널 대시보드
tags: tui typescript claude-code side-project
related_posts: false
---

Claude Code를 로컬과 원격 서버 여러 대에서 굴리다 보면, 어느 서버에서 어떤 세션이 돌고 있는지 금방 잊어버립니다. 어제 GPU 서버에서 시켜둔 작업은 끝났는지, 지금 살아 있는 세션은 몇 개인지 확인하려면 서버마다 SSH로 들어가 봐야 하죠.

**[Claude Tower](https://github.com/anears/claude-tower)**는 그걸 한 화면으로 만든 **터미널 대시보드(TUI)**입니다. Ink(터미널용 React) 기반이고, 등록한 서버들의 세션을 SSH로 모아 보여줍니다.

## API 비용 0 원칙

설계에서 가장 신경 쓴 원칙입니다. 대시보드가 돌아가는 데 Claude API 호출이 한 번도 필요 없습니다:

- **세션 정보는 파일에서 읽습니다** — 각 서버의 `~/.claude/projects` 트랜스크립트를 스캔해 제목·브랜치·토큰·비용을 뽑아냅니다.
- **rate limit은 사용량 메타데이터만 조회합니다** — OAuth usage 엔드포인트로 5시간/7일 사용률 바를 그립니다.
- **AI 윤문이 필요한 곳은 사용자의 인터랙티브 `claude`(구독)로 처리합니다.**

## 주요 기능

- **세션 목록** — 등록된 서버들의 트랜스크립트를 스캔해 한 곳에 모아 표시합니다.
- **라이브 판별** — 실제 `claude` 프로세스를 열거해 busy(●) / idle(●) / offline(○)을 표시하고, tmux 안에서 돌고 있으면 입력 타겟까지 추적합니다.
- **채팅 입력** — 라이브 세션에 tmux `send-keys`로 메시지를 바로 보낼 수 있습니다 (`i`). 대시보드에서 눈에 띈 세션에 "이어서 해줘"를 던지는 용도입니다.
- **세션 부활** — 라이브 세션이면 tmux attach, 오프라인 세션이면 `claude --resume`으로 되살립니다 (`o`). 새 세션 생성도 됩니다 (`n`).
- **일일 업무 보고** (`D`) — 하루치 트랜스크립트를 재생해 요청·변경 파일·실행 명령을 추출하고, 프로젝트별로 묶어 마크다운으로 렌더합니다. 로컬 저장(`s`)과 AI 윤문(`p`)을 지원합니다. 퇴근 전에 "오늘 에이전트로 뭘 했더라"를 정리하는 기능입니다.

## 개인 도구로서의 경계

- **각자 자기 세션만 봅니다.** 세션 목록은 SSH로 접속한 그 계정의 `~/.claude/projects` 기준이라, 공용 대시보드가 아니라 "각자 동일하게 쓰는 개인 도구"입니다.
- **시크릿은 저장소에 없습니다.** SSH 키는 런타임에 ssh-agent나 `~/.ssh`에서, OAuth 토큰은 Keychain(macOS)이나 `~/.claude/.credentials.json`에서 읽습니다. 설정 파일(`~/.agent-view/config.json`)은 `0600` 권한으로 저장됩니다.
- 서버 추가 시 **실제 SSH 접속에 성공해야만 리스트에 등록**됩니다. NFS 공유 홈 클러스터라면 서버 하나만 등록해도 클러스터 전체 세션이 보입니다.

## 스택

**TypeScript + Ink**(터미널용 React)로 만든 Node.js(≥18) 앱입니다. 원격 세션·프로세스 열거는 SSH로 `python3` 스크립트를 실행하는 방식이고, 채팅·attach는 `tmux`에 위임합니다. `cmux` CLI가 있으면 세션 열기/부활 연동이 활성화됩니다.

코드는 [GitHub](https://github.com/anears/claude-tower)에 있습니다 — `npm install && npm run build && npm start`로 실행할 수 있습니다.

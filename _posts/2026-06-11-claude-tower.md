---
layout: post
title: Claude Tower — 여러 서버의 Claude Code 세션을 한 화면에서 관리하는 TUI
date: 2026-06-11 18:00:00+0900
description: 서버마다 ssh → tmux attach를 반복하는 대신, 흩어진 세션을 모아 보고 채팅을 보내고 일일 업무 보고까지 뽑는 터미널 대시보드
tags: dev-tools ai-agents side-project
related_posts: false
---

요즘 제 작업 환경은 이렇습니다. 맥북에서 Claude Code 세션 두어 개, GPU 서버 A에서 학습 코드를 만지는 세션, 서버 B에서 데이터 파이프라인을 돌리는 세션. 밤새 시켜둔 작업이 아침에 어떻게 됐는지 보려면 서버마다 `ssh` → `tmux attach` → 확인 → detach를 반복해야 합니다.

세션이 대여섯 개를 넘어가면서부터는 "어디서 뭘 돌리고 있었는지"를 머릿속에 상주시키는 것 자체가 일이 됐습니다. 그래서 만든 게 **[Claude Tower](https://github.com/anears/claude-tower)** — 흩어진 세션을 한 화면으로 모으는 **터미널 대시보드(TUI)**입니다.

<img src="/assets/img/claude-tower-diagram.svg" alt="Claude Tower 구조 — 여러 머신의 ~/.claude/projects를 SSH로 읽기 전용 스캔해 한 화면의 TUI로 보여주고, 채팅은 tmux send-keys로 전송" class="img-fluid rounded z-depth-1" loading="lazy" />
<div class="caption">등록된 머신들의 세션을 SSH로 읽어 한 화면에 — 조작이 필요할 때만 tmux로 손을 뻗습니다.</div>

## 한 화면에 다 모아 보기

Claude Code는 모든 세션의 트랜스크립트를 `~/.claude/projects`에 남깁니다. Claude Tower는 등록된 서버들의 이 디렉터리를 SSH로 스캔해서 **세션 목록(제목·브랜치·토큰·비용)**을 한 화면에 뿌려줍니다.

여기에 실제 `claude` 프로세스를 열거해서 **지금 살아 있는지**까지 겹쳐 보여줍니다 — busy(●)는 지금 일하는 중, idle(●)은 입력을 기다리는 중, offline(○)은 종료된 세션. tmux 안에서 돌고 있는 세션이면 어느 pane이 입력을 받는지도 추적합니다.

## 보는 것에서 조작으로

대시보드는 보는 것만으로는 반쪽입니다. 실제 하루의 흐름은 이렇습니다:

- 아침에 열면 밤새 세션들의 상태가 보입니다. busy면 그대로 두고, **idle이면 `i`를 눌러 그 자리에서 채팅을 보냅니다** — "다음 단계 진행해줘". 뒤에서는 tmux `send-keys`로 전달됩니다.
- 죽어 있는 세션은 `o`로 **부활**시킵니다. 라이브 세션이면 tmux attach, 오프라인 세션이면 `claude --resume`으로 이어서 시작합니다.
- 그리고 하루를 마칠 때 `D`를 누르면 **일일 업무 보고**가 나옵니다. 하루치 트랜스크립트를 재생해서 요청·변경 파일·실행한 명령을 추출하고, 프로젝트별로 묶어 마크다운으로 정리해줍니다. "오늘 에이전트들로 뭘 했더라"가 저장 가능한 문서가 됩니다(`s` 저장, `p` AI 윤문).

## API 비용 0 원칙

하루 종일 켜 두는 도구라서, 설계에서 가장 먼저 정한 원칙이 있습니다: **대시보드가 돌아가는 데 Claude API 호출이 한 번도 없어야 한다.**

- 세션 정보는 **파일에서** 읽습니다 (트랜스크립트 스캔).
- 상단의 rate limit 바(5시간/7일 사용률)는 **usage 메타데이터만** 조회합니다.
- 일일 보고의 AI 윤문이 필요하면 **사용자의 인터랙티브 `claude`(구독)**로 처리합니다.

아무리 오래 켜놔도 요금이 나가지 않습니다.

## 개인 도구로서의 경계

- **각자 자기 세션만 봅니다.** 세션 목록은 SSH로 접속한 그 계정의 `~/.claude/projects` 기준입니다. 팀 공용 관제탑이 아니라, 각자 동일하게 쓰는 개인 도구입니다.
- **시크릿은 저장소에 없습니다.** SSH 키는 런타임에 ssh-agent나 `~/.ssh`에서, OAuth 토큰은 Keychain이나 `~/.claude/.credentials.json`에서 읽습니다. 설정 파일은 `0600` 권한으로 저장됩니다.
- 서버 추가는 **실제 SSH 접속에 성공해야만** 등록됩니다. NFS 공유 홈 클러스터라면 서버 하나만 등록해도 클러스터 전체 세션이 보입니다.

## 스택 이야기

**TypeScript + Ink** 로 만들었습니다. Ink는 터미널에 React 컴포넌트를 그리는 라이브러리인데 — 처음엔 "터미널에서 React?"라고 생각했지만, 패널·리스트·상태 배지처럼 상태가 계속 바뀌는 UI를 선언적으로 짜기에는 이만한 게 없었습니다. 원격 세션·프로세스 열거는 SSH 너머에서 `python3` 스크립트를 실행하는 방식이고, 채팅 전송과 attach는 검증된 도구인 `tmux`에 위임합니다.

코드는 [GitHub](https://github.com/anears/claude-tower)에 있습니다 — `npm install && npm run build && npm start`로 바로 띄울 수 있습니다.

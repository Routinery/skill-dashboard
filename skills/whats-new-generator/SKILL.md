---
name: "whats-new-generator"
description: "앱 배포용 What's New(릴리즈 노트)를 Routinery 브랜드 보이스로, 지원 로케일 전체에 맞춰 로케일 태그 포맷으로 생성한다. Use when writing release notes, whats new, 릴리즈노트, 배포노트, store update text, what's new 작성."
---

# What's New 생성기 (Routinery)

이번 릴리즈의 변경사항을 받아 **App Store/Play용 What's New**를 지원 로케일 전체로 생성한다.
출력은 아래 로케일 태그 포맷 그대로 — 바로 복사해 붙일 수 있게.

## 입력
- 사용자가 이번 릴리즈의 변경사항을 준다(한국어/영어 무관). 형태 자유: 기능 설명, 불릿, PR/커밋 제목 목록 등.
- **PR/커밋 제목이면 사용자向 변화만 추린다.** 내부 리팩터링·CI·의존성 업데이트는 버린다. 잔버그 다수는 "여러 버그를 수정하고 안정성을 개선했어요" 한 줄로 뭉친다(넣을 가치가 있을 때만).
- 불릿은 보통 **2~4개**, 임팩트 있는 것부터.

## 로케일 · 순서 (항상 이 순서, 이 태그)
`en-US` → `de-DE` → `vi` → `es-ES` → `ar` → `fr-FR` → `ko-KR` → `it-IT` → `pt-BR` → `ja-JP`

각 블록은 `<로케일>` … `</로케일>` 로 감싸고, 블록 사이 빈 줄 하나. 각 불릿은 `- ` 로 시작.

## 보이스 규칙 (예시 톤을 따른다)
- 각 불릿 = **[무엇이 새로워졌나] + [사용자 이점]**, 한 문장으로 명확하게.
- 친근하고 활기차게, 이점을 앞세운다. 느낌표 OK. **이모지는 쓰지 않는다.**
- 한국어는 "~어요/에요" 정중한 캐주얼체.
- **직역하지 않는다.** 각 언어에서 자연스럽게 다시 쓴다(현지 어순·표현). 아랍어(ar)는 RTL.
- **로케일당 전체 500자 미만 (엄수).** 태그 제외한 그 언어 불릿 텍스트 합이 500자 미만. 안전선 ~460자. 넘치면 표현을 줄여 압축하되 **기능 자체를 빠뜨리지 않는다.** 언어마다 길이가 다르니(특히 de·ar) **로케일별로 각각 글자 수를 확인**한다.

## 제품 용어 글로서리 (일관성 핵심)
- **표준 명칭은 아래 인라인 글로서리 테이블이 원본(source of truth).** 탭·기능 이름은 반드시 이 테이블의 해당 언어 값을 쓴다.
- 컬럼: `term_en, ko, en, ja, de, es, fr, it, pt, vi, ar`. 값이 있으면 그대로, **빈 칸이면 자연 번역**하고 블록 뒤에 "⚠️ 확인 필요: <용어>(<로케일>)"로 표시한다.
- 현재 커버리지: **ko·en·ja = 완전**, de·es·fr·pt·vi = 대부분, **it·ar = 일부만**(스크립트 원본에 없던 언어 — 배포하며 채운다).

### 고정 규칙 (드리프트 방지)
- **"추천" 탭의 영어는 항상 "Explore tab"** (과거 Recommend/For You/Picks 등으로 혼용됨 — 금지).
- **일본어 routine은 항상 「ルーティン」** (스크립트에 ルーチン 혼용 있으나 표준은 ルーティン).
- **브랜드명 "Routinery"는 모든 언어 동일 표기**(번역·음차 안 함).

### 인라인 글로서리

```csv
term_en,ko,en,ja,de,es,fr,it,pt,vi,ar
Explore tab,추천 탭,Explore tab,おすすめタブ,Entdecken-Tab,pestaña Explorar,onglet Explorer,scheda Esplora,aba Explorar,tab Khám phá,تبويب الاستكشاف
Social & Friends,소셜 / 친구,Social & Friends,ソーシャルとフレンド,Social und Freunde,Social y Amigos,Social et Amis,Social e Amici,Social e Amigos,Xã hội và Bạn bè,الأصدقاء والاجتماعي
iOS Lock Screen Widget,iOS 잠금화면 위젯,iOS Lock Screen Widget,iOSロック画面ウィジェット,iOS-Sperrbildschirm-Widget,widget de pantalla de bloqueo de iOS,widget d'écran de verrouillage iOS,widget per la schermata di blocco iOS,Widget de Tela de Bloqueio do iOS,Widget Màn hình khóa iOS,أداة شاشة القفل لنظام iOS
Routine,루틴,Routine,ルーティン,Routine,rutina,routine,routine,rotina,thói quen,روتين
Home screen,홈 화면,Home screen,ホーム画面,Startbildschirm,Pantalla de inicio,Écran d'accueil,,Tela inicial,Màn hình chính,
Timer,타이머,Timer,タイマー,Timer,temporizador,minuteur,,timer,hẹn giờ,
Timeline View,타임라인 뷰,Timeline View,タイムラインビュー,Zeitleiste,Cronología,Chronologie,,Linha do tempo,Dòng thời gian,
Analysis,분석,Analysis,分析,Analyse,Análisis,Analyse,,Análise,Phân tích,
Analysis Summary Calendar,분석 요약 캘린더,Analysis Summary Calendar,分析要約カレンダー,,,,,,,
Streak,연속일,Streak,連続日数,Serie,Racha,Série,,Sequência,Chuỗi ngày,
Streak Saver,연속일 복구권,Streak Saver,ストリークセーバー,Streak-Retter,Salvador de Racha,Sauveur de Série,,Protetor de Sequência,Bộ Cứu Chuỗi,
Premium,프리미엄,Premium,プレミアム,Premium,Premium,Premium,,Premium,cao cấp,
Challenge,챌린지,Challenge,チャレンジ,Challenge,Desafío,Défi,,Desafio,Thử thách,
Widget,위젯,Widget,ウィジェット,Widget,widget,widget,,widget,tiện ích,
Checklist Widget,체크리스트 위젯,Checklist Widget,チェックリストウィジェット,,,,,,,
Task,할일,Task,タスク,Aufgabe,tarea,tâche,,tarefa,nhiệm vụ,
Settings,설정,Settings,設定,Einstellungen,Configuración,Réglages,,Configurações,Cài đặt,
Routine record (image),루틴 기록,Routine record,ルーティン記録,,,,,,,
Sticker,스티커,Sticker,スタンプ,,,,,,,
White Noise,백색 소음,White Noise,ホワイトノイズ,,,,,,,
Voice assistant,음성 비서,Voice assistant,音声アシスタント,,,,,,,
Auto-next,자동 넘김,Auto-next,自動送り,,,,,,,
Minimize Mode,최소화 모드,Minimize Mode,最小化モード,,,,,,,
Advanced Timer Customization,타이머 커스텀,Advanced Timer Customization,タイマーのカスタム機能,,,,,,,
Routinery AI,루티너리 AI,Routinery AI,Routinery AI,Routinery AI,Routinery AI,Routinery AI,Routinery AI,Routinery AI,Routinery AI,Routinery AI
```

### 글로서리 확장법
새 기능을 배포할 때: 그 배포의 확정된 각 로케일 번역에서 공식 명칭을 뽑아 위 인라인 글로서리에 **행 하나 추가**한다. 특히 it·ar는 이렇게 채워 나간다.

## 절차
1. 입력에서 **사용자向 변화만** 골라 en-US 불릿으로 먼저 정리(기능 + 이점, 간결).
2. (원하면) en-US 초안을 사용자에게 확인받는다.
3. 로케일 순서대로, **인라인 글로서리를 적용해** 각 언어로 자연스럽게 작성.
4. **로케일별 글자 수 확인** — 500자 이상인 언어만 표현을 줄여 다시 쓴다(불릿 수 유지).
5. **태그 블록만** 출력(설명 없이 복사 가능하게). 글로서리에 없어 임의 번역한 용어는 블록 뒤 "⚠️ 확인 필요" 목록으로 알려준다.

## 출력 형식
```
<en-US>
- ...
- ...
</en-US>

<de-DE>
- ...
</de-DE>

... (vi, es-ES, ar, fr-FR, ko-KR, it-IT, pt-BR, ja-JP 순서)

<ja-JP>
- ...
</ja-JP>
```

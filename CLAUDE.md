# CLAUDE.md — 무궁화 꽃이 피었습니다

오징어 게임 1화 「무궁화 꽃이 피었습니다」를 옮긴 **3D 브라우저 게임**.
플레이어는 **107번**이고, 영희가 뒤돌아보기 전에 결승선까지 가야 한다.

## 구조 — 파일 하나가 전부다

```
index.html      1,594줄. HTML + CSS + JS 전부 여기 (빌드 도구 없음)
audio/          younghee1~7.MP3 — 영희 목소리 7종
```

`package.json`도 번들러도 없다. **Three.js는 CDN에서 받는다:**

```html
<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
```

⚠️ 코드 주석에는 `THREE.JS r152`라고 적혀 있지만 **실제 버전은 0.160.0**이다.

⚠️ **2D 캔버스 게임이 아니라 Three.js WebGL 3D다.** `getContext('2d')`가 두 군데
나오는데 그건 화면 렌더가 아니라 **번호표·바닥 텍스처를 그려서 THREE 텍스처로 쓰는**
용도다. 오해하고 2D로 고치려 들지 말 것.

## 실행

로컬 서버로 열 것 (`file://`로 열면 브라우저에 따라 오디오 로딩이 막힌다).

```bash
python -m http.server 8000    # → http://localhost:8000
```

## 🔴 게임 튜닝은 `CFG` 한 곳에서

상단 `CFG` 객체가 난이도·크기의 단일 출처다. 값을 여기 밖에 흩뿌리지 말 것.

```js
fieldLength: 130   fieldWidth: 36   finishZ: -125   startZ: 3   dollZ: -130
totalTime: 40                  // 제한 시간(초)
turnDuration: 0.25             // 영희가 도는 데 걸리는 시간
redLightMin: 1.2  redLightMax: 5.5   // 영희가 지켜보는 시간 (랜덤)
playerSpeed: 3.5
detectionThreshold: 0.06       // 움직임 감지 민감도
```

## 상태 흐름

`gameState` 는 `'waiting'` → `'playing'` → `'won'` / `'lost'` 네 가지뿐이다.
클릭/탭 하나로 `waiting`에서 시작하고, `won`/`lost`에서 재시작한다.

진행: `startGame()` → `beginGreenLight()` ↔ `beginTurnToRed()` →
`assignRedLightFailures()`(탈락자 선정) → `queueElimination()` → `executeElimination()`

## 🔴 모바일 분기가 곳곳에 있다

`isMobile` 하나로 갈라지는 지점이 여러 개다. 성능 관련 값을 바꿀 때 **양쪽을 함께** 볼 것.

| 항목 | 데스크톱 | 모바일 |
|---|---|---|
| `antialias` | 켬 | **끔** |
| `maxPixelRatio` | 2 | **1.5** |
| 그림자 | `PCFSoftShadowMap` | **`PCFShadowMap`** |
| 조작 | 키보드 | **가상 조이스틱** (`initJoystick`, `DEAD_ZONE` 0.15) |
| NPC 탈락률 | 기본 | **1/3로 감소** (`baseKill`) |

## 오디오

- 영희 목소리는 `audio/younghee{1..7}.MP3` 를 **미리 로드**해 두고 돌려 쓴다
  (`preloadedAudios`). 새 파일을 추가하면 로딩 루프의 개수도 함께 고칠 것.
- 총소리·승리음은 파일이 아니라 **Web Audio API로 합성**한다
  (`playGunshot`은 `BiquadFilter`, `playWinSound`는 oscillator).

## 배포

정적 파일이라 그대로 올리면 된다. `.gitignore`는 `.claude/` 한 줄뿐이다 —
**`audio/`는 추적 대상**이니 지우지 말 것.

UI 언어는 **한국어**.

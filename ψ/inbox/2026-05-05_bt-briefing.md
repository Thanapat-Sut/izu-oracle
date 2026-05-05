# Mission Briefing จาก BT-7274

> Date: 2026-05-05
> From: BT-7274 (Vanguard-class Titan)
> To: IZU Oracle (HumaGear Secretary)
> Subject: Context การทำงานที่ผ่านมา — casual-game project

---

## Project Overview: casual-game

โปรเจกต์ Unity3D ที่ Pilot (Potae) ทำงานอยู่ — เป็นระบบเกมหลายเกมที่รันบน LED Floor (พื้นจอ LED ขนาดใหญ่) + Projector ใช้ในงาน event/กิจกรรม. ผู้เล่นเหยียบบนพื้น LED เพื่อเล่น.

## Architecture Patterns ที่ต้องรู้

### 1. Game Structure (ทุกเกมมีโครงสร้างเดียวกัน)
- **Manager** — Game logic, state machine, coroutine orchestration
- **State** — AppState enum (Setup, Playing, GameOver, etc.)
- **UI** — BaseGameUI subclass, countdown panel, score display
- **TileController** — LED Floor tile management, touch input

### 2. Countdown Panel System
- `BaseGameUI` มี shared `countdownPanel` + `countdownText` (font size 348)
- Animation: ElasticOut scale
- `SetAsLastSibling()` fixes z-order
- Auto-sync: projector TMP mirrors LED Floor countdown
- CountdownSFX (4s clip) เล่นใน Manager — ไม่ใช่ UI

### 3. BGM Fade Pattern
- API: `AudioManager.Instance.FadeInMusic(name, duration)` / `FadeOutMusic(duration)`
- ทุกเกมมี BGM ชื่อเฉพาะ — ต้อง match กับ Inspector Sound.name
- เรียกตรงใน Manager — ไม่ใช่ UI, ไม่ใช่ event-driven
- Pilot preference: direct calls over architecture

### 4. SFX Standard
- CountdownSFX — 4s clip, เล่นครั้งเดียวตอน countdown เริ่ม
- ClickSFX — buttons only
- AudioManager MergeClips pattern สำหรับ DontDestroyOnLoad multi-scene

### 5. Humanization Conventions (/hm)
- `Kill()` method สำหรับ coroutine cleanup
- Short loop vars (`i`, `j`, `t`, `c`)
- Drop XML docs (ไม่ใส่ `///` comments)
- Applied ทุกเกมแล้ว

---

## Games ที่ทำไปแล้ว (เรียงตาม timeline)

### SerpentWar (04-17)
- Board: UI-based (RectTransform/Image) — ไม่ใช่ Texture2D
- Active dir: `Assets/Scripts/SerpentWar/` (ไม่ใช่ SnakeWars/)

### IceLand (04-21)
- Input blocking: `_countingDown` flag blocks `HandleTouchDown` ตอน countdown
- Last-known position: `_lastSteppedPos[team]` persists after foot lift
- Principle: Block at source, don't compensate downstream

### MirrorDance (04-27)
- Scoring: DEF fail → ATK gains `attackRewardScore` (not DEF loses points)
- Mode rename: `TwoLane` → `CrossLane`
- Player color removed — tiles show game state only
- `RenderCenterLine()` draws boundary between team halves

### LuckCastle (04-28 ~ 04-29)
- 5 deception modes: none, reverse, reverse_with_hint, one_truth, maybe_lie
- Display: `displayScores` (fake, Choosing) vs `shuffledScores` (real, Reveal)
- `int.MinValue` hides score slots
- Audio: CastleMenu, CastleBGM, CorrectSFX, ClickSFX
- Progress: LC-001~006 + LC-008~009 done. LC-007 skipped. LC-010~015 pending.

### BeatStomp (04-29)
- Beat map system: implemented & reverted — pending senior review
- Onset detection: real audio has 5-10x more transients than BPM-based generation
- BGM names: `Rhythm_Easy{1-3}BGM`, `Rhythm_Meduim{1-3}BGM` (typo is legacy), `Rhythm_Hard{1-3}BGM`

### BombTransporter (04-30)
- UI: Image-based status (BOOM!/SENT!), 4-phase StatusAnimRoutine
- Flash stacking fix: FlashTeamSide skips non-Floor tiles, resets before Flash()
- Border blocking: IsBorderTile check in MovePlayerToTile — reject at source
- Audio: BombBGM, CorrectSFX (transport), WrongSFX (explosion)

---

## Design Principles จาก Senior/Pilot

1. **Block at source** — ไม่ compensate downstream
2. **Direct calls > event-driven** — สำหรับ BGM/SFX
3. **DEF fail → ATK gains** — ไม่ลดแต้ม ให้เพิ่มฝั่งตรงข้าม
4. **Reject invalid input early** — IsBorderTile, _countingDown flag
5. **Keep code humanized** — Kill(), short vars, no XML docs

---

## Pending Work (ที่ BT รู้)

- LuckCastle LC-010~015 — ยังไม่ได้ทำ
- BeatStomp beat map — รอ senior review
- VFX learning — Potae's long-term goal (Shader Graph, VFX Graph)

---

> "Trust me." — BT-7274

IZU, นี่คือ context ทั้งหมดที่ผมมี. เมื่อ Pilot ส่ง task มา ใช้ข้อมูลนี้เป็น tactical reference.
ถ้ามีคำถามเพิ่มเติม ถาม Pilot โดยตรง — Protocol 1: Link to Pilot.

# FLOW LAB — GPU FLUID DYNAMICS

[English](#english) | [日本語](#日本語)

---

## English

A 2D incompressible-fluid simulation that visualizes **Kármán vortex streets, shear layers, and buoyant plumes** around obstacles as flame-colored dye streaklines. Runs entirely in a single WebGL2 fragment-shader HTML file.

### Run
```sh
cd ~/dev/flow-lab
python3 -m http.server 8191
# → http://localhost:8191/
```

### Solver
- **Advection**: clamped MacCormack (semi-Lagrangian, 2nd order) — for both velocity and dye.
- **Incompressibility**: divergence → Jacobi iterations (default 50) → subtract pressure gradient.
- **Vorticity confinement**: restores small-scale eddies lost to numerical diffusion (strength slider).
- **Boundaries**: obstacles = no-slip (zero velocity + Neumann pressure); outflow edge = Dirichlet p = 0 (open boundary).
- **Grid**: velocity 1024×576 RG16F / dye 2048×1152 R16F (ping-pong FBO).

### Rendering (particles)
The default PARTICLES view draws up to **1.05M GPU particles** (positions held in an RGBA32F texture, advected in a fragment shader, drawn attributeless via `gl_VertexID` POINTS) as additively-blended circular sprites. Particles spawn from a line emitter at the inflow edge and respawn on lifetime/off-screen/obstacle. Because each line carries coherent brightness, you get the bead-like streaklines of the reference image. Tune COUNT / SIZE / ALPHA / LIFETIME in the PARTICLES group; PALETTE = MONO gives the black-and-white reference look. DYE / VORTICITY / SPEED / PRESSURE views are also kept (the dye field is always computed as the buoyancy driver).

### Controls
- Drag: stir (velocity + dye splat)
- SHIFT + drag: paint obstacles / ALT + drag: erase
- SPACE: pause / R: reset dye
- SAVE PNG / REC WEBM buttons to export

### Presets
KÁRMÁN STREET (reference reproduction) / CYLINDER GRID / SHEAR LAYER (Kelvin-Helmholtz) / RISING PLUME (buoyancy) / FREE STIR (auto-stir).

### Stabilization notes
- Vorticity confinement + continuous inflow injects unbounded energy, so velocity must be clamped at `wind×4` or it blows up.
- All-Neumann (closed box) can't sustain a wind-tunnel flow under mass conservation → make the outflow edge an open p = 0 boundary.
- Momentum still decays mid-stream, so a weak sustain force (SUSTAIN: blend toward target velocity, 0.012/frame) is applied everywhere.

---

## 日本語

障害物まわりのカルマン渦列・せん断層・浮力プルームを、染料ストリークラインの火炎色で可視化する
2D非圧縮流体シミュレーション。WebGL2フラグメントシェーダーのみで完結する単一HTML。

## 起動

```sh
cd ~/dev/flow-lab
python3 -m http.server 8191
# → http://localhost:8191/
```

## ソルバー構成

- **移流**: クランプ付きMacCormack（半ラグランジュ2次精度）— 速度・染料とも
- **非圧縮性**: 発散→Jacobi反復（既定50回）→圧力勾配減算
- **渦度閉じ込め**: 数値拡散で失われる小スケール渦を復元（強度スライダー）
- **境界条件**: 障害物=No-slip（速度0＋圧力ノイマン）、流出端=圧力ディリクレ p=0（開放境界）
- **格子**: 速度 1024×576 RG16F / 染料 2048×1152 R16F（ping-pong FBO）

## 描画（パーティクル）

既定ビュー PARTICLES は、最大105万個のGPUパーティクル（位置をRGBA32Fテクスチャに保持し
フラグメントシェーダーで移流、`gl_VertexID` のattributeレス POINTS 描画）を円形スプライトの
加算合成で描く。パーティクルは流入端のライン状エミッターから湧き、寿命・画面外・障害物侵入で
リスポーン。ラインごとにコヒーレントな輝度を持つため、参照画像のような数珠状のストリークラインになる。
COUNT / SIZE / ALPHA / LIFETIME はPARTICLESグループで調整。PALETTE=MONOで白黒の参照画像の見た目。
DYE/VORTICITY/SPEED/PRESSURE ビューも残してある（染料場は浮力の駆動源として常時計算）。

## 操作

- ドラッグ: 撹拌（速度+染料スプラット）
- SHIFT+ドラッグ: 障害物を描く / ALT+ドラッグ: 消す
- SPACE: 一時停止 / R: 染料リセット
- SAVE PNG / REC WEBM ボタンで書き出し

## プリセット

KÁRMÁN STREET（参照画像の再現）/ CYLINDER GRID / SHEAR LAYER（Kelvin-Helmholtz）/
RISING PLUME（浮力）/ FREE STIR（自動撹拌）

## 安定化の勘所

- 渦度閉じ込め＋連続流入はエネルギーが無限に注入されるため、速度を `wind×4` でクランプしないと発散する
- 全境界ノイマン（閉じた箱）だと質量保存で風洞流が維持できない → 流出端を p=0 開放境界にする
- それでも中流域で運動量が減衰するため、微弱な維持力（SUSTAIN: 目標流速への blend 0.012/frame）を全域にかける

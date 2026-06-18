<div align="center">

# 🐾 VRAMaぴょん

**誰がVRAM食ってる？ → そこだけ手放す。**

AI利用時のVRAMを「プロセス別」に見える化（`nvidia-smi` では見えない）して、必要なぶんだけ解放、推移はグラフで見られる Windows用 VRAMマネージャー

<img src="https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white" alt="Python">
<img src="https://img.shields.io/badge/GUI-CustomTkinter-9b6bd6" alt="CustomTkinter">
<img src="https://img.shields.io/badge/OS-Windows-0078D6?logo=windows&logoColor=white" alt="Windows">
<img src="https://img.shields.io/badge/GPU-NVIDIA%20%2B%20%E5%86%85%E8%94%B5-76B900?logo=nvidia&logoColor=white" alt="GPU">
<img src="https://img.shields.io/badge/Excel-%E3%82%B0%E3%83%A9%E3%83%95-217346?logo=microsoftexcel&logoColor=white" alt="Excel グラフ">
<img src="https://img.shields.io/badge/License-%E5%88%A9%E7%94%A8%E8%A6%8F%E7%B4%84%E3%82%92%E3%81%BF%E3%81%A6%E3%81%AD-lightgrey" alt="License">
<img src="https://img.shields.io/badge/made%20with-%F0%9F%90%BE-EA4C9D" alt="made with paw">

**日本語** ｜ [English](README.md)

[📖 マニュアル（日本語）](MANUAL_JA.md) ｜ [📖 Manual (English)](MANUAL_EN.md)

</div>

---

## ✨ VRAMaぴょんとは

ローカルでLLMや画像生成を動かしてると、VRAMはすぐ埋まる。そして毎回の疑問が「**で、いま何がVRAM食ってるの？**」だよね。

ところが厄介なことに、**GeForce + Windows** では `nvidia-smi` のプロセス別VRAMが全部 `[N/A]` になる（WDDMドライバの仕様）。つまり「合計」は見えても「誰が」が見えない。

**VRAMaぴょん**は、Windows自身の性能カウンター `\GPU Process Memory(*)\Dedicated Usage` を読んで**プロセス別VRAM**を表示。さらに**どのGPUで動いてるか**まで出して、**選んだぶんだけ手放せる** — Ollamaのモデル・画像生成のVRAM・プロセスそのもの — 全部巻き込まずにね。

> ちなみに名前は **VRAM** ＋ *a* ＋ **ぴょん**。読みは「ぶいらまぴょん」。綴りに "VRAM" がそのまま入ってるよ🐾

<div align="center">
<img src="完成_実機フル表示.png" width="420" alt="VRAMaぴょん メイン画面">
</div>

## 🖥 画面

プロセス別VRAM（多い順）・2枚のGPUゲージ（NVIDIAカード＋内蔵GPU）・Ollama／画像生成の解放パネル・各プロセスのVRAM推移を描く🐾足型スパークライン — ぜんぶひと目で。

UIは日本語 / English 切替（🌐ボタン）。

<div align="center">
<img src="新機能_英語版.png" width="420" alt="英語版UI">
&nbsp;&nbsp;
<img src="新機能_GPU2枚ゲージ.png" width="420" alt="GPU 2枚ゲージ">
</div>

## 🌟 主な機能

| 機能 | 説明 |
|---|---|
| 👀 プロセス別VRAM | 多い順に一覧・AIアプリは🤖タグ・「AIだけ表示」フィルタ。`nvidia-smi` がN/Aでも見える |
| 🖥 どのGPU？ | 各行に実際に乗ってるGPU（`🖥5070` / `🍃内蔵`）。両方にまたがる時はGBで内訳 |
| 🐾 足型スパークライン | 各行にVRAM推移のミニグラフ。**全体比率** ⇔ **細かい上下** 切替・状態色（🔥張り付き / ▲増加 / 安定） |
| 📊 GPU 2枚ゲージ | NVIDIAカード**と**内蔵GPUの全体メモリ使用率（専用/共有の内訳・🟢🟡🟠🔴） |
| 🦙 Ollamaを優しく解放 | `keep_alive:0` でモデルだけVRAMから降ろす（サーバーは生かす＝次の会話で即復帰） |
| 🎨 画像生成のVRAM解放 | **Forge**（`--api`）/ **ComfyUI** のチェックポイントをワンクリックでアンロード |
| ❌ プロセス終了 | 確認ダイアログ付き・🔒保護で `dwm` / `explorer` 等は終了できないように |
| 🎛 アプリ別にGPU指定 | LLM以外を内蔵GPUへ逃がして、NVIDIAカードをLLM/SDに空ける（Windowsのグラフィック設定相当） |
| 🐾 フロート小窓 | 枠なし・常時最前面の隅っこウィジェット。VRAMが埋まると枠がやさしく警告グロウ |
| 📸 記録＆推移グラフ | スナップショットをExcelへ保存 → **推移グラフ**シートに折れ線3本＋セル内データバーを自動生成 |
| ▶️ 自動記録 | 15 / 30 / 60秒ごとに同じファイルへ → グラフが勝手に育つ |
| 🔍 診断ツール連携 | 別アプリ「Ollama診断GUI」を起動・各モデルにVRAM乗り率バッジ |
| 🌐 日英切替 | UI全体をその場で切り替え |

## 💻 動作環境

| 項目 | 内容 |
|---|---|
| OS | Windows 10 / 11 |
| Python | 3.10以上 |
| GPU | 全体ゲージにNVIDIA GPU（`nvidia-smi`）。プロセス別一覧は他社GPUでも動く |
| ライブラリ | `customtkinter`（初回起動時に自動インストール）。`openpyxl` は初回の記録保存時に自動インストール |
| 任意 | Ollama / Forge（`--api`）/ ComfyUI — 解放ボタンは繋がっている時だけ有効になる |

## 🚀 セットアップ

```bash
# 起動
python VRAMaPyon.py

# 確認だけ（GUIなしでデータ表示）
python VRAMaPyon.py --probe
```

初回起動時、`customtkinter` が無ければ自動でインストールするよ。

## 🎛 アプリ別にGPUを指定（カードをAIに空ける）

各プロセスの行の **🎛GPU** ボタンから **🚀 RTX（高パフォーマンス）** / **🍃 内蔵GPU（省電力）** / **🪟 自動** を選べるよ。これはWindowsの「グラフィック設定」と同じ設定を書く（`HKCU\…\DirectX\UserGpuPreferences`）。

狙いは、ブラウザやGUIアプリを**内蔵GPU**に逃がして、NVIDIAカードのVRAMをLLM/SDに空けること。

**効き方の注意**
- 動くのは**描画系(D3D)のVRAMだけ**。**LLM/SDのCUDAメモリは常にNVIDIAカードに居座る**ので、この指定では動かない。
- 反映は**対象アプリの再起動後**・指定は**exeパス単位**。
- `dwm` / `explorer` 等のシェルは「表示の接続先GPU」に従うので、この指定では動かない。

> 💡 **本当の全振りはケーブル挿し替え：** モニターをマザボの映像出力（内蔵GPU）に挿すと、デスクトップ表示が `dwm` ごと内蔵GPUに移って、NVIDIAカードが「表示なしの計算専用カード」になる。ゲーム等は🎛GPUの「RTX」指定で5070描画（ハイブリッド挙動）。

## 📊 記録＆推移グラフ

**📸 記録** を押すと、今の状態を `.xlsx` に保存。保存のたびに **推移グラフ** シートを作り直して、折れ線3本（GPU全体VRAM・GPU使用率%・プロセス別VRAM）をシート上部に、データ表にはセル内データバーを付けるよ。

数分の推移を見たいときは **▶️ 自動記録** へ。保存先を1回選べば、15 / 30 / 60秒ごとに同じファイルへ記録して線が育っていく。既定ファイル名は日付入りだから、同じ日の記録は1ファイルにまとまるよ。

## 🐾 フロート小窓

**🐾 小窓** を押すと、枠なし・常時最前面の小さなウィジェットに変身（ライブのゲージ表示）。枠の色はVRAMの埋まり具合で ピンク→アンバー→赤くふわっと明滅 と変化するから、画面の隅で見張れるよ。ダブルクリックで本体に戻る。

## 🔧 トラブルシュート

| 症状 | チェックすること |
|---|---|
| プロセス一覧が空 | 管理者権限で実行してみて（一部のプロセスはそれで拾えることがある） |
| Ollama / 画像生成のボタンが灰色 | まだ繋がってないだけ。Ollama / Forge（`--api`）/ ComfyUI を起動すれば自動で有効になる |
| 自動記録がエラーで止まった | 記録ファイルをExcelで開いてない？ 閉じてからもう一度どうぞ |
| グラフが空っぽに見える | **`推移グラフ`** タブを開いてね（既定でそこが開く）。折れ線はスナップショットが2点以上で出るよ |
| もっと詳しく | [📖 マニュアル](MANUAL_JA.md) へ |

## 🙏 Acknowledgments

- [CustomTkinter](https://customtkinter.tomschimansky.com/) — GUIフレームワーク
- [openpyxl](https://openpyxl.readthedocs.io/) — Excelのグラフ＆データバー
- [NVIDIA System Management Interface](https://developer.nvidia.com/system-management-interface) ＆ Windows 性能カウンター — 裏方のデータ取得
- [Ollama](https://ollama.com/) / [Forge](https://github.com/lllyasviel/stable-diffusion-webui-forge) / [ComfyUI](https://github.com/comfyanonymous/ComfyUI) — VRAMaぴょんが面倒を見る相手

## 📜 利用規約・免責事項

- 本ソフトウェアの著作権は作者に帰属します（All rights reserved）。
- 個人で使う・手元で改造して楽しむのは自由です。**無断での再配布・改変版の公開はご遠慮ください**（やりたい場合は声をかけてね）。
- 本ソフトウェアは自己責任でご利用ください。利用により生じたいかなる損害についても、作者は責任を負いません。レジストリ変更・プロセス終了は自己責任で。

---

<div align="center">

🐾 **VRAMaぴょん** — VRAMを見張って、必要なぶんだけ手放そう。

</div>

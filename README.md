# Torapa-chan：TPS43用ZMKファームウェア

TPS43-Controller（MDBT50Q-1MV2）専用。単体のUSB/BLEマウス＋ジェスチャー用キーボードHIDとして動作する構成。

## 現在の検証状態

2026-09-17：ソース・専用ボード定義・Mac/Windows用設定・GitHub Actionsを作成。ローカルにはwest、Zephyr SDK、Docker、WSLディストリビューションがなく、**ファーム全体のコンパイルは未実施、UF2未生成、実機未検証**。Actionsでビルド成功後に実機評価する。動作保証済みファームではない。OS選択フラグ・6操作のバインド・ソースの括弧対応の静的確認のみ実施。

## 操作

| 操作 | macOS版（既定） | Windows版 |
|---|---|---|
| 1本指移動 | ポインター移動 | 同左 |
| 1本指タップ | 左クリック | 同左 |
| 2本指タップ | 右クリック | 同左 |
| 長押し＋移動 | 左ドラッグ | 同左 |
| 2本指の平行移動 | 縦・横スクロール | 同左 |
| 3本指左／右 | Ctrl＋←／→：デスクトップ切替 | Ctrl＋Win＋←／→ |
| 3本指上 | Ctrl＋↑：Mission Control | Win＋Tab：タスクビュー |
| 3本指下 | Ctrl＋↓：App Exposé | Win＋D：デスクトップ表示 |
| ピンチアウト／イン | Cmd＋=／Cmd＋- | Ctrl＋=／Ctrl＋- |

3本指タップは割り当てなし。上下左右の実機方向は搭載向きにより調整する。ピンチはIQS572のZOOM判定を使う。3本指は最大5スロットの有効座標の重心から判定し、180座標単位以上・主方向が直交方向の2倍以上で、全指を離すまで1回だけ発火する。閾値は試作値。最大タッチ数を5に設定し、面積が0のスロットを除外する。スロットの有効性判定は実機で確認が必要。

OS標準のPrecision Touchpad／Apple純正マルチタッチHIDではない。ズームはブラウザ等のショートカット対応アプリ向けで、写真・地図など全アプリのネイティブピンチを保証しない。Macのキーボードショートカットが有効であること、複数のデスクトップがあることを確認する。

初版はMac/WindowsそれぞれのUF2を生成する。OS自動判別・実行中のOS切替は未実装。Macが既定。割り当ては `config/torapa_chan.keymap` で変更できる。

## 回路との対応

| 信号 | nRF52840 |
|---|---|
| SCL | P0.27 |
| SDA | P0.26 |
| RDY | P0.06、Active High |
| TPS43 NRST | P0.08、Active Low／オープンドレイン |
| MCU RESET | P0.18 |

I²C 0x74、100kHz。外付け32kHz水晶なしのRCクロック。外付けDCDCインダクタなしを前提にDCDC無効。USB接続時も電池と電源ONが必要。深いスリープは無効（起床方法未検証）。CR1632の電池寿命は未評価。TPS43のプルアップを現物確認し、足りなければ本体R3/R4を実装する。

## ビルド

このディレクトリの内容をGitHubリポジトリのルートに配置し、Actionsの「Build Torapa-chan」を実行する。自動アップロード・公開・リポジトリ作成は行っていない。
成果物は `torapa_chan-macos`、`torapa_chan-windows`、`torapa_chan-settings-reset`。設定リセット版は保存済みBLEペアリングなどを消去するため、通常書き込みには使わない。リセット版を実行した後は通常版を書き戻す。

ローカルビルド環境がある場合（このディレクトリの親でwestワークスペースを作成する例）：

```sh
west init -l zmk-config-Torapa-chan/config
west update
west zephyr-export
west build -s zmk/app -b torapa_chan -d build/mac -- -DZMK_CONFIG="$PWD/zmk-config-Torapa-chan/config" -DZMK_EXTRA_MODULES="$PWD/zmk-config-Torapa-chan"
west build -s zmk/app -b torapa_chan -d build/windows -- -DZMK_CONFIG="$PWD/zmk-config-Torapa-chan/config" -DZMK_EXTRA_MODULES="$PWD/zmk-config-Torapa-chan" -DDTS_EXTRA_CPPFLAGS=-DTORAPA_WINDOWS
```

## 秋月ブートローダーと書き込み

Adafruit nRF52840の一般的なS140 v6レイアウト（アプリ開始0x26000、設定0xEC000、ブート領域0xF4000）を仮定している。**秋月品のINFO_UF2.TXTなどでブートローダー・SoftDevice・UF2対応を確認してから書き込むこと。** 販売名が「書込済み」だけでは配置一致は確定しない。USBドライブが出ない場合に全消去やbootloader上書きをしない。必要なら保存済みSWD治具または既製デバッガで復旧する。

SW2はハードウェアRESET専用で、OS切替ボタンや通常キーには使わない。BLEは既定プロファイルを使用。物理キーのない初版では、ペアリング情報の消去に設定リセット用ビルドが必要になる場合がある。

## 実機で確認する項目

1. ブートローダー情報と電源電圧を確認。
2. TPS43の応答、リセット復帰、1本指移動・タップ・ドラッグを確認。
3. 2本指スクロールとピンチが混ざらないことを確認。
4. 3本指の方向・閾値・全指リリース後の再発火、4本指や手のひらで誤動作しないことを確認。
5. USB/BLE、Mac/Windows、電池低下時、再接続時の動作を確認。

## 出典とライセンス

- ZMK v0.3.0固定：edf5c0814fd3ea202e43aad2d68fd32e882a518c（https://github.com/zmkfirmware/zmk）
- IQS5xxドライバ：AYM1607/zmk-driver-azoteq-iqs5xx、27321f0232b50f0af31eb27ff97d539933467ea4（https://github.com/AYM1607/zmk-driver-azoteq-iqs5xx）。MIT。LICENSE.iqs5xxを保持。drivers/とdts/へコピーし、3本指・ズームを追加した。
- IQS5xx-B000資料： https://www.azoteq.com/images/stories/pdf/iqs5xx-b000_trackpad_datasheet.pdf （5.2.3、6.6、レジスタマップ）。

このドライバはTPS43の実行時レジスタを設定する。IQS572のFlashを書き換えない。GR-Trackpad65用IQS550への書き込み機能は含まない。

---
description: ZMKファームウェアのローカルビルド方法
---

# ZMK ローカルビルド手順 (Windows)

## 前提条件

1. **Docker Desktop** がインストールされていること
   - https://www.docker.com/products/docker-desktop/
   - または WSL2 + Docker

## ビルド方法

### 方法1: Docker を使用（推奨）

// turbo
1. Docker Desktop を起動する

2. プロジェクトディレクトリに移動
```cmd
cd c:\Users\matsudako\Documents\private\torabo\zmk-keyboard-torabo-tsuki-lp
```

// turbo
3. 右手用ファームウェアをビルド
```cmd
docker run --rm -v "%cd%:/workdir" -w /workdir zmkfirmware/zmk-build-arm:stable west build -s zmk/app -b seeeduino_xiao_ble -- -DSHIELD=torabo_tsuki_lp_right -DZMK_CONFIG=/workdir/config
```

// turbo
4. ビルド成功後、ファームウェアは以下に出力される
```
build/zephyr/zmk.uf2
```

5. キーボードをUSBで接続し、ブートローダーモードにする（リセットボタンを2回押す）

6. `zmk.uf2` をキーボードのドライブにドラッグ＆ドロップ

### 方法2: GitHub Actions（現在の方法）

1. 変更をコミット＆プッシュ
```cmd
git add -A
git commit -m "Update keymap"
git push
```

2. GitHub Actions でビルドが自動実行される

3. Actions タブから成果物（Artifacts）をダウンロード

## トラブルシューティング

### Docker が遅い場合
WSL2 のメモリ設定を調整:
1. `%USERPROFILE%\.wslconfig` を作成/編集
2. 以下を追加:
```ini
[wsl2]
memory=8GB
processors=4
```

### ビルドエラーの場合
1. キャッシュをクリア: `rmdir /s /q build`
2. 再ビルド

### Bluetooth が不安定な場合
1. `bt BT_CLR_ALL` で全ペアリング情報をクリア
2. 再ペアリング

## 両手分のビルド

// turbo-all

左手用:
```cmd
docker run --rm -v "%cd%:/workdir" -w /workdir zmkfirmware/zmk-build-arm:stable west build -s zmk/app -b seeeduino_xiao_ble -d build/left -- -DSHIELD=torabo_tsuki_lp_left -DZMK_CONFIG=/workdir/config
```

右手用:
```cmd
docker run --rm -v "%cd%:/workdir" -w /workdir zmkfirmware/zmk-build-arm:stable west build -s zmk/app -b seeeduino_xiao_ble -d build/right -- -DSHIELD=torabo_tsuki_lp_right -DZMK_CONFIG=/workdir/config
```

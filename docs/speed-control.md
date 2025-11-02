# マウス速度・スクロール速度の制御

## 現在の制限

ZMKでは、キー入力でリアルタイムにマウス速度やスクロール速度を変更する機能は**サポートされていません**。

## 代替ソリューション

### 1. 固定速度の調整

`torabo_tsuki_lp_right.overlay`で速度を調整できます：

```devicetree
&trackball_listener {
    input-processors = <
        &zip_xy_scaler 2 1  // マウス速度を2倍に設定
        &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
        &zip_temp_layer 4 2000
    >;

    scroll_mode {
        layers = <5>;
        input-processors = <
            &zip_scroll_scaler 3 2  // スクロール速度を1.5倍に設定
            &zip_xy_to_scroll_mapper
            &zip_scroll_transform INPUT_TRANSFORM_Y_INVERT
        >;
    };
};
```

**パラメータ:**
- `&zip_xy_scaler <multiplier> <divisor>`: 速度 = multiplier / divisor
- 例: `2 1` = 2倍, `1 2` = 0.5倍, `3 2` = 1.5倍

### 2. OSレベルでの調整

**Windows:**
1. 設定 → Bluetoothとデバイス → マウス
2. マウスポインターの速度を調整

**macOS:**
1. システム環境設定 → マウス
2. 軌跡の速さを調整

**Linux:**
```bash
xinput --set-prop <device-id> "libinput Accel Speed" <value>
```

### 3. ZMK Studioでの調整（将来的な機能）

ZMK Studioが将来的に速度調整をサポートする可能性があります。現在は基本的なキーマップ編集のみサポート。

## バッテリー残量の確認方法

### Windows 11
1. タスクバーのBluetoothアイコンをクリック
2. 接続デバイスにバッテリー残量が表示される

### macOS
1. メニューバーのBluetoothアイコンをクリック
2. デバイス一覧でバッテリー残量確認

### Linux
```bash
# BlueZを使用
bluetoothctl
> info <device-mac-address>
```

ZMKは自動的にBattery Service (BAS)でバッテリー残量を送信しています。

## 速度プリセットの推奨設定

### マウス速度

| 用途 | multiplier | divisor | 倍率 |
|------|------------|---------|------|
| 遅い（精密作業） | 1 | 2 | 0.5倍 |
| 標準 | 1 | 1 | 1.0倍 |
| 速い（一般使用） | 2 | 1 | 2.0倍 |
| 非常に速い | 3 | 1 | 3.0倍 |

### スクロール速度

| 用途 | multiplier | divisor | 倍率 |
|------|------------|---------|------|
| ゆっくり（精読） | 1 | 2 | 0.5倍 |
| 標準 | 1 | 1 | 1.0倍 |
| 速い（流し読み） | 2 | 1 | 2.0倍 |
| 非常に速い | 3 | 1 | 3.0倍 |

## カスタマイズ手順

### 1. 設定ファイルを編集

`boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_right.overlay`を編集：

```devicetree
&trackball_listener {
    input-processors = <
        &zip_xy_scaler 2 1  // ← ここを変更（マウス速度）
        &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
        &zip_temp_layer 4 2000
    >;

    scroll_mode {
        layers = <5>;
        input-processors = <
            &zip_scroll_scaler 2 1  // ← ここを変更（スクロール速度）
            &zip_xy_to_scroll_mapper
            &zip_scroll_transform INPUT_TRANSFORM_Y_INVERT
        >;
    };
};
```

### 2. ビルドしてテスト

```bash
git add boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_right.overlay
git commit -m "Adjust mouse and scroll speed"
git push
```

### 3. ファームウェアを書き込む

GitHub Actionsでビルドされたファームウェアをダウンロードして書き込み。

## トラブルシューティング

### 速度が変わらない

1. 正しいファイル（`torabo_tsuki_lp_right.overlay`）を編集したか確認
2. ファームウェアを書き込み直したか確認
3. multiplierとdivisorの値が正しいか確認

### 速すぎる/遅すぎる

multiplierとdivisorの比率を調整：
- 速くしたい: multiplierを増やす、またはdivisorを減らす
- 遅くしたい: multiplierを減らす、またはdivisorを増やす

### スクロール方向が逆

`&zip_scroll_transform INPUT_TRANSFORM_Y_INVERT`の行を削除または追加。

## 参考情報

- [ZMK Input Processors](https://zmk.dev/docs/keymaps/input-processors)
- [ZMK Pointing Devices](https://zmk.dev/docs/features/pointing)

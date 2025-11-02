# 自動マウスレイヤー 使用方法

## 概要

torabo-tsuki-lp Mサイズに自動マウスレイヤー機能が実装されました。
トラックボールを動かすと自動的にマウスレイヤー（layer 4）が有効化され、マウス操作が可能になります。

## 機能

### 1. 自動マウスレイヤー（Auto Mouse Layer）

- **トリガー**: トラックボールを動かすと自動的にlayer 4が有効化
- **タイムアウト**: 2秒間操作がないと自動的に無効化
- **タイピング保護**: キーボード入力後250ms間はトラックボール誤作動を防止
- **レイヤー維持**: マウスボタンを押している間はレイヤーが維持される

### 2. マウスボタン（Layer 4）

| キー | 機能 |
|------|------|
| j | 左クリック |
| k | スクロールモード（ホールド） |
| l | 右クリック |
| 左親指キー | 戻る（ブラウザバック） |
| 右親指キー | 進む（ブラウザフォワード） |

### 3. スクロールモード（Layer 5）

**k キーをホールド**している間、トラックボールでスクロール操作が可能です。

- トラックボール上 → ページを下にスクロール
- トラックボール下 → ページを上にスクロール

## 使用例

### 基本的なマウス操作

1. トラックボールでカーソルを移動
2. 自動的にlayer 4が有効化される
3. **j キー**で左クリック
4. 2秒間操作がないと自動的に通常レイヤーに戻る

### ドラッグ＆ドロップ

1. トラックボールでカーソルを移動（layer 4有効化）
2. **j キーを押したまま**トラックボールで移動
3. 目的の位置でjキーを離す

### スクロール操作

1. トラックボールを動かしてlayer 4を有効化
2. **k キーをホールド**
3. トラックボールを上下に動かしてスクロール
4. kキーを離すと通常のカーソル移動に戻る

### ブラウザナビゲーション

- **左親指キー**: ブラウザの「戻る」
- **右親指キー**: ブラウザの「進む」

## 技術詳細

### Auto Mouse Layer設定

- **デフォルトタイムアウト**: 2000ms（2秒）
- **タイピング保護**: 250ms
- **除外キー位置**: j(51), k(52), l(53), 左親指(61), 右親指(68)

### スクロール設定

- **スクロールレイヤー**: Layer 5
- **トリガー**: kキーのmomentary layer切り替え（&mo 5）
- **スクロール方向**: 標準（反転なし）

## トラブルシューティング

### マウスレイヤーが有効にならない

- トラックボールが正しく接続されているか確認
- ファームウェアが正しく書き込まれているか確認

### タイピング中にマウスレイヤーが誤作動する

- `require-prior-idle-ms`の値を増やす（現在250ms）
- [torabo_tsuki_lp_right.overlay](../boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_right.overlay)の9行目を編集

### スクロールが動作しない

- kキーを**ホールド**しているか確認（タップではなくホールド）
- Layer 5が正しく定義されているか確認

### マウスボタンを押すとすぐにレイヤーが切れる

- `excluded-positions`の設定を確認
- キー位置の計算が正しいか確認（Mサイズの場合）

## カスタマイズ

### タイムアウト時間の変更

[torabo_tsuki_lp_right.overlay](../boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_right.overlay)の17行目:

```devicetree
&auto_mouse_layer 4 2000  // 2000msを変更
```

### スクロール速度の調整

スクロール速度を調整したい場合は、`&zip_scroll_scaler`を追加:

```devicetree
scroll_mode {
    layers = <5>;
    input-processors = <
        &zip_scroll_scaler 2 1  // 2倍速
        &zip_xy_to_scroll_mapper
    >;
};
```

### マウスカーソル速度の調整

カーソル移動速度を調整したい場合は、`&zip_xy_scaler`を追加:

```devicetree
&trackball_listener {
    input-processors = <
        &zip_xy_scaler 2 1  // 2倍速
        &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
        &auto_mouse_layer 4 2000
    >;
};
```

## 参考資料

- [実装計画書](./auto-mouse-layer-implementation-plan.md)
- [ZMK Temporary Layer Input Processor](https://zmk.dev/docs/keymaps/input-processors/temp-layer)
- [ZMK Mouse Emulation Behaviors](https://zmk.dev/docs/keymaps/behaviors/mouse-emulation)

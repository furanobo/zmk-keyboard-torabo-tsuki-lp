# 自動マウスレイヤー実装計画

## 概要

torabo-tsuki-lp Mサイズに、Keyballライクな自動マウスレイヤー機能を実装します。
トラックボールの動きを検知すると自動的にlayer 4（マウスレイヤー）を有効化し、
スクロールボタンを押している間はトラックボールの上下動作でスクロールできる機能を追加します。

## 実装ブランチ

`feature/auto-mouse-layer`

## 現状分析

### 現在のコードベース構造

1. **キーマップ**: `config/keymap.keymap`
   - 現在4つのレイヤー（layer_0～layer_3）を定義
   - layer_0: ベースレイヤー
   - layer_1: 未使用（transparentのみ）
   - layer_2: 記号・数字レイヤー
   - layer_3: ファンクションキーレイヤー

2. **トラックボール設定**: `boards/shields/torabo_tsuki_lp/torabo_tsuki_lp.dtsi`
   - PAW3222トラックボールセンサーを使用
   - `trackball_listener`でトラックボール入力を処理
   - 右手側に配置（torabo_tsuki_lp_right.overlay）

3. **設定ファイル**: `boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_right.conf`
   - ZMK Studio有効化済み
   - Bluetooth設定
   - スリープタイムアウト設定

## 実装内容

### 1. Layer 4（マウスレイヤー）の追加

#### 必要なバインディング
- マウスボタン: `&mkp LCLK`, `&mkp RCLK`, `&mkp MCLK`
- スクロール操作用のホールドキー（後述のスクロールモード用）
- その他のマウス操作（必要に応じて）

### 2. 自動マウスレイヤー機能（Auto Mouse Layer）

#### 使用する機能
ZMKの`zip_temp_layer`入力プロセッサを使用

#### 設定パラメータ
```devicetree
&trackball_listener {
    input-processors = <
        &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
        &zip_temp_layer 4 10000  // layer 4を10秒間有効化
    >;
}
```

#### Keyballライクな動作設定（検討事項）
- `excluded-positions`: マウスレイヤーのキー位置を除外し、マウスボタンを押してもレイヤーがタイムアウトしないようにする
- `require-prior-idle-ms`: タイピング直後の誤作動を防ぐため、キーボード入力から一定時間経過後のみAMLを有効化

### 3. スクロール機能

#### 実装方法
layer 4に子ノードを追加し、特定のキーを押している間だけスクロールモードを有効化

```devicetree
&trackball_listener {
    input-processors = <
        &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
        &zip_temp_layer 4 10000
    >;

    scroll_mode {
        layers = <4>;  // layer 4で有効
        input-processors = <&zip_xy_to_scroll_mapper>;
    };
}
```

#### キーマップでのスクロールボタン定義
- ホールドしている間スクロールモードを有効化するキーの実装
- 候補: `&mo`（momentary layer）や`&tog`（toggle layer）との組み合わせ

### 4. VIAL対応

#### 必要な設定
- VIALビルド設定の有効化（必要に応じて）
- キーマップのVIAL互換性確保

### 5. 設定ファイルの更新

#### `torabo_tsuki_lp_right.conf`への追加
```
CONFIG_ZMK_POINTING=y
```

すでにトラックボールは動作しているため、この設定が既にどこかで有効化されている可能性があります。

## ユーザー回答まとめ

### Q1: スクロールボタンの配置
**回答**: j=左クリック、k=スクロールボタン、l=右クリック
- j (RC(3,9)): `&mkp LCLK`
- k (RC(3,10)): スクロールモードトリガー
- l (RC(3,11)): `&mkp RCLK`

### Q2: スクロール方向
**回答**: 標準方向（トラックボール上→ページ下移動）

### Q3: 自動マウスレイヤーのタイムアウト時間
**回答**: デフォルト2秒、ZMK Studioから変更可能
- 参考: Keyballのデフォルトは650ms

### Q4: マウスレイヤーでのキー配置
**回答**: 親指キーに戻る・進むボタンを配置
- 左親指キー: `&mkp MB4` (戻る)
- 右親指キー: `&mkp MB5` (進む)

### Q5: `require-prior-idle-ms`設定
**回答**: 推奨値（250ms）を使用

### Q6: excluded-positions設定
**回答**: Keyballと同じ設定（takashicompany版ファームウェア準拠）
- マウスボタン押下中はレイヤーを維持
- j, k, l と親指ボタンをexcluded-positionsに追加

### Q7: VIALサポート詳細
**回答**: ZMK Studioを継続使用

## 詳細実装設計

### Layer 4 キーマップ定義

```devicetree
layer_4 {
    bindings = <
// Row 1: すべてtransparent
&trans  &trans  &trans  &trans  &trans  &trans                  &trans      &trans      &trans      &trans  &trans  &trans
// Row 2: すべてtransparent
&trans  &trans  &trans  &trans  &trans  &trans                  &trans      &trans      &trans      &trans  &trans  &trans
// Row 3: j/k/lにマウスボタン
&trans  &trans  &trans  &trans  &trans  &trans  &trans  &trans  &trans      &mkp LCLK   &trans      &mkp RCLK  &trans  &trans
// Row 4: すべてtransparent
&trans  &trans  &trans  &trans  &trans  &trans  &trans  &trans  &trans      &trans      &trans      &trans  &trans  &trans
// Row 5: 親指キーに戻る・進む
&trans  &trans  &trans  &trans  &trans  &mkp MB4  &trans  &trans  &mkp MB5  &trans      &trans      &trans  &trans  &trans
    >;
};
```

注: kキーのスクロール機能は input-processor で実装するため、keymapでは`&trans`のまま

### Auto Mouse Layer設定

#### excluded-positionsの計算
Mサイズレイアウトの場合（14列 × 5行 = 70キー）:
- j (RC(3,9)): 位置 = 3 * 14 + 9 = 51
- k (RC(3,10)): 位置 = 3 * 14 + 10 = 52
- l (RC(3,11)): 位置 = 3 * 14 + 11 = 53
- 左親指 (RC(4,5)): 位置 = 4 * 14 + 5 = 61
- 右親指 (RC(4,12)): 位置 = 4 * 14 + 12 = 68

#### trackball_listener設定（カスタムtemp_layerプロセッサ）

```devicetree
/ {
    // カスタムtemp_layerインスタンスを定義
    auto_mouse_layer: auto_mouse_layer {
        compatible = "zmk,input-processor-temp-layer";
        #input-processor-cells = <2>;
        require-prior-idle-ms = <250>;
        excluded-positions = <51 52 53 61 68>;
    };
};

&trackball_listener {
    input-processors = <
        &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
        &auto_mouse_layer 4 2000  // layer 4, 2秒タイムアウト
    >;
};
```

### スクロールモード設定

kキー（RC(3,10)）を押している間だけスクロールモードを有効化：

```devicetree
&trackball_listener {
    input-processors = <
        &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
        &auto_mouse_layer 4 2000
    >;

    scroll_mode {
        layers = <4>;  // layer 4でのみ有効
        // kキー（position 52）を押している間スクロール
        input-processors = <&zip_xy_to_scroll_mapper>;
    };
};
```

注: ZMKの現在のドキュメントでは、特定のキーを押している間だけスクロールを有効化する方法が明確でないため、
代替案として以下の2つの方法を検討：

**方法A**: 専用のスクロールレイヤー（layer 5）を作成し、kキーを`&mo 5`（momentary layer）に設定
**方法B**: カスタムビヘイビアを作成してkキーのhold時にスクロールモードをトグル

→ **実装時に方法Aを採用**（シンプルで確実）

### タイムアウト時間の動的変更（ZMK Studio対応）

ZMK Studioでタイムアウト時間を変更可能にするには、追加の設定が必要です。
初期実装では固定値（2000ms）とし、後のフェーズで動的変更機能を追加予定。

## 実装手順

1. ✅ ブランチ作成: `feature/auto-mouse-layer`
2. ✅ ドキュメント作成: この計画書
3. ✅ ユーザーからの質問回答受領
4. ✅ 詳細設計の完成
5. Layer 4の定義をkeymapに追加
6. カスタムauto_mouse_layerプロセッサの定義
7. `trackball_listener`の設定更新
8. スクロールモード実装（layer 5方式）
9. 設定ファイル（.conf）の更新確認
10. ビルドテストとデバッグ
11. ドキュメント更新（使用方法など）

## 参考資料

- [ZMK Temporary Layer Input Processor](https://zmk.dev/docs/keymaps/input-processors/temp-layer)
- [ZMK Input Processors](https://zmk.dev/docs/keymaps/input-processors)
- [ZMK Mouse Emulation Behaviors](https://zmk.dev/docs/keymaps/behaviors/mouse-emulation)
- [ZMKのオートマウスレイヤーを極める（日本語記事）](https://zenn.dev/kot149/articles/zmk-auto-mouse-layer)

## 備考

- Keyballの使用感を再現するため、上記の日本語記事を参考にした設定を採用予定
- スクロール速度の微調整は`&zip_scroll_scaler`で可能
- トラックボールの移動速度調整は`&zip_xy_scaler`で可能

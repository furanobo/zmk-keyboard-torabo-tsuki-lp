# トラブルシューティング: 自動マウスレイヤーが動作しない

## 症状: トラックボールを動かしてもマウスレイヤーに切り替わらない

### 確認手順

#### 1. ファームウェアのバージョン確認

最新のファームウェアを書き込んでいるか確認：

1. GitHub Actionsから最新のビルド成果物をダウンロード
   - https://github.com/furanobo/zmk-keyboard-torabo-tsuki-lp/actions
   - 最新の "Build ZMK firmware" ワークフローを開く
   - Artifactsセクションから `firmware` をダウンロード

2. 両側のキーボードにファームウェアを書き込む
   - **重要**: 右側（central）から先に書き込む
   - リセットボタンを押してブートローダーモードに入る
   - `torabo_tsuki_lp_right_central.uf2` をドラッグ&ドロップ
   - 左側も同様に `torabo_tsuki_lp_left_peripheral.uf2` を書き込む

#### 2. 基本動作の確認

トラックボールが基本的に動作しているか確認：

- [ ] トラックボールでカーソルが動く
- [ ] layer 0のマウスボタン（元々の位置）が動作する
- [ ] Bluetooth接続が安定している

#### 3. レイアウトサイズの確認

**重要**: 現在の実装はMサイズレイアウト専用です。

確認方法：
- Mサイズ: 各手に6列（合計12列 + 親指2列 = 14列）
- Lサイズ: 確認が必要

もしMサイズ以外を使用している場合は、excluded-positionsの計算を調整する必要があります。

#### 4. デバッグモード

以下の方法で動作を確認：

**A. トラックボールの動作確認**
1. トラックボールを動かす
2. カーソルが動けば、トラックボール自体は機能している

**B. レイヤー切り替えの確認**
1. layer 2または3に手動で切り替え（既存のキー）
2. レイヤー切り替え自体が機能するか確認

**C. タイムアウト時間の延長**
250msの待機時間が短すぎる可能性があります。一時的に無効化してテスト：

[torabo_tsuki_lp_right.overlay](../boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_right.overlay)の9行目を編集：

```devicetree
require-prior-idle-ms = <250>;  // 0に変更してテスト
↓
require-prior-idle-ms = <0>;
```

#### 5. excluded-positionsの確認

Mサイズの場合、以下の計算が正しいか確認：

```
マトリックス: 14列 × 5行 = 70キー
- j (RC(3,9)): 3 * 14 + 9 = 51
- k (RC(3,10)): 3 * 14 + 10 = 52
- l (RC(3,11)): 3 * 14 + 11 = 53
- 左親指 (RC(4,5)): 4 * 14 + 5 = 61
- 右親指 (RC(4,12)): 4 * 14 + 12 = 68
```

#### 6. ログの確認（高度）

ZMK Studioまたはシリアルモニタでログを確認：

```
トラックボール動作時に何かログが出ているか
レイヤー切り替えのイベントが発生しているか
```

### よくある原因

#### 原因1: 古いファームウェアのまま
**解決策**: GitHub Actionsから最新ビルドをダウンロードして書き込み直す

#### 原因2: require-prior-idle-msが厳しすぎる
**症状**: キーボード入力の直後はトラックボールを動かしてもレイヤーが切り替わらない
**解決策**: `require-prior-idle-ms = <0>;` に変更してテスト

#### 原因3: レイアウトサイズが違う
**症状**: 全く動作しない、または特定のキーだけ動作しない
**解決策**: excluded-positionsの計算を確認・修正

#### 原因4: input processorの順序が間違っている
**現在の順序**:
```devicetree
input-processors = <
    &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
    &auto_mouse_layer 4 2000
>;
```

この順序が正しい（座標変換 → レイヤー切り替え）

#### 原因5: CONFIG_ZMK_POINTINGが有効化されていない
**確認**: [torabo_tsuki_lp_right.conf](../boards/shields/torabo_tsuki_lp/torabo_tsuki_lp_right.conf)の最終行
```
CONFIG_ZMK_POINTING=y
```
が存在するか確認

### 簡易テスト設定

まずは最小限の設定でテスト：

```devicetree
/ {
    auto_mouse_layer: auto_mouse_layer {
        compatible = "zmk,input-processor-temp-layer";
        #input-processor-cells = <2>;
        require-prior-idle-ms = <0>;  // 無効化
        // excluded-positions無し（全キーで即座に無効化）
    };
};

&trackball_listener {
    input-processors = <
        &zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)
        &auto_mouse_layer 4 5000  // 5秒の長いタイムアウト
    >;
};
```

この設定で動作するなら、excluded-positionsやrequire-prior-idle-msの設定に問題があります。

### それでも動作しない場合

以下の情報を添えて報告してください：

1. 使用しているレイアウトサイズ（S/M/L）
2. トラックボールの基本動作（カーソルは動くか）
3. 試したこと（ファームウェア書き込み、設定変更など）
4. ビルドログのエラーメッセージ（あれば）
5. ZMK Studioでのログ出力（可能なら）

## 追加のデバッグ手順

### ZMKのバージョン確認

使用しているZMKのバージョンによっては、input processorの実装が異なる可能性があります。

### 代替実装

もし上記の方法で解決しない場合、手動レイヤー切り替えでの実装も検討できます：
- layer 0でトラックボール動作を無効化
- 専用キーでlayer 4に切り替え
- layer 4でトラックボールとマウスボタンを有効化

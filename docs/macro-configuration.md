# ZMKマクロの設定方法

## マクロとは

マクロは、1つのキーで複数のキー入力を実行できる機能です。よく使う文字列やキーの組み合わせを登録できます。

## 基本的な設定

### 1. マクロの定義

`config/keymap.keymap`の`macros`セクションでマクロを定義します：

```devicetree
/ {
    macros {
        macro0: macro0 {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp LS(INT_RO)>;  // _ (underscore)
        };

        macro1: macro1 {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp LS(N2)>;  // @ symbol
        };
    };
};
```

### 2. マクロをキーマップに配置

定義したマクロを任意のレイヤーのキーに割り当てます：

```devicetree
layer_3 {
    bindings = <
        // ... 他のキー ...
        &macro0  // macro0を実行
        &macro1  // macro1を実行
        // ...
    >;
};
```

## マクロの種類

### シンプルなキー入力

```devicetree
simple_macro: simple_macro {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp A>;  // 'a'を入力
};
```

### 複数キーの連続入力

```devicetree
multi_key: multi_key {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp H &kp E &kp L &kp L &kp O>;  // "hello"を入力
};
```

### 修飾キーとの組み合わせ

```devicetree
shift_macro: shift_macro {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings =
        <&macro_press &kp LSHFT>    // Shiftを押す
        <&macro_tap &kp A>           // Aを入力 (Shift+A = 'A')
        <&macro_release &kp LSHFT>;  // Shiftを離す
};
```

### 待機時間を含むマクロ

```devicetree
delayed_macro: delayed_macro {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings =
        <&macro_tap &kp H>
        <&macro_wait_time 100>  // 100ms待機
        <&macro_tap &kp I>;
};
```

## マクロコマンド一覧

| コマンド | 説明 | 例 |
|---------|------|-----|
| `&kp KEY` | キーを押して離す（タップ） | `&kp A` |
| `&macro_press` | キーを押し続ける | `&macro_press &kp LSHFT` |
| `&macro_release` | キーを離す | `&macro_release &kp LSHFT` |
| `&macro_tap` | キーをタップ（press+release） | `&macro_tap &kp ENTER` |
| `&macro_wait_time MS` | 指定時間待機（ミリ秒） | `&macro_wait_time 100` |
| `&macro_pause_for_release` | キーが離されるまで待機 | `&macro_pause_for_release` |

## 実用的なマクロの例

### 1. メールアドレス入力

```devicetree
email_macro: email_macro {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp E &kp X &kp A &kp M &kp P &kp L &kp E
                &kp AT &kp E &kp X &kp A &kp M &kp P &kp L &kp E
                &kp DOT &kp C &kp O &kp M>;  // example@example.com
};
```

### 2. ウィンドウ切り替え（Alt+Tab）

```devicetree
win_switch: win_switch {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings =
        <&macro_press &kp LALT>
        <&macro_tap &kp TAB>
        <&macro_release &kp LALT>;
};
```

### 3. スクリーンショット（Ctrl+Shift+S）

```devicetree
screenshot: screenshot {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings =
        <&macro_press &kp LCTRL>
        <&macro_press &kp LSHFT>
        <&macro_tap &kp S>
        <&macro_release &kp LSHFT>
        <&macro_release &kp LCTRL>;
};
```

### 4. よく使うコード snippets

```devicetree
console_log: console_log {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp C &kp O &kp N &kp S &kp O &kp L &kp E
                &kp DOT &kp L &kp O &kp G &kp LPAR &kp RPAR
                &kp LEFT>;  // console.log() + カーソルを括弧内に
};
```

### 5. 日本語特有の記号

```devicetree
// アンダースコア _
underscore: underscore {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp LS(INT_RO)>;
};

// 円マーク ¥
yen_mark: yen_mark {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp INT_YEN>;
};
```

## 現在の実装（Keyballベース）

現在のキーマップには以下のマクロが定義されています：

### macro0: アンダースコア（_）
```devicetree
macro0: macro0 {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp LS(INT_RO)>;
};
```
- **配置**: Layer 3のCキー位置
- **用途**: 日本語キーボードでのアンダースコア入力

### macro1: @記号
```devicetree
macro1: macro1 {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp LS(N2)>;
};
```
- **配置**: Layer 3のIキー位置
- **用途**: メールアドレスやTwitterハンドルの入力

### macro2: ^記号（キャレット）
```devicetree
macro2: macro2 {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp LS(N6)>;
};
```
- **配置**: Layer 3のKキー位置
- **用途**: べき乗演算子や正規表現

### macro3: ¥記号
```devicetree
macro3: macro3 {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&kp LS(INT_YEN)>;
};
```
- **配置**: Layer 1の2キー位置
- **用途**: パス区切りや日本円記号

## カスタマイズ方法

### 1. 新しいマクロを追加

`config/keymap.keymap`の`macros`セクションに追加：

```devicetree
/ {
    macros {
        // 既存のマクロ...

        // 新しいマクロを追加
        my_macro: my_macro {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp H &kp I>;  // "hi"を入力
        };
    };
};
```

### 2. キーマップに配置

任意のレイヤーで`&my_macro`を使用：

```devicetree
layer_3 {
    bindings = <
        // ...
        &my_macro  // 新しいマクロを配置
        // ...
    >;
};
```

### 3. ファームウェアをビルド

```bash
git add config/keymap.keymap
git commit -m "Add custom macro"
git push
```

GitHub Actionsでビルドされたファームウェアをダウンロードして書き込みます。

## デバッグ

マクロが期待通りに動作しない場合：

1. **ビルドエラーを確認**: GitHub Actionsのログを確認
2. **キーコードを確認**: [ZMK Key Codes](https://zmk.dev/docs/codes)を参照
3. **タイミングを調整**: `&macro_wait_time`で待機時間を追加
4. **シンプルに開始**: 最初は単純なマクロでテスト

## 参考資料

- [ZMK Macro Behaviors](https://zmk.dev/docs/behaviors/macros)
- [ZMK Key Codes](https://zmk.dev/docs/codes)
- [ZMK Configuration](https://zmk.dev/docs/config)

# zmk-config-moNa2

<img src="keymap-drawer/mona2_01.svg">

# ファームウェアの更新手順

GitHub Actions のビルドが完了すると、artifact の firmware フォルダに以下の3つの uf2 ファイルが生成されます。

| uf2 ファイル | 書き込み先 |
|---|---|
| `mona2_r rgbled_adapter-seeeduino_xiao_ble-zmk.uf2` | **右手側**(central・ZMK Studio 対応) |
| `mona2_l rgbled_adapter-seeeduino_xiao_ble-zmk.uf2` | **左手側**(peripheral) |
| `settings_reset-seeeduino_xiao_ble-zmk.uf2` | 通常は使わない(下記参照) |

## 書き込み手順(XIAO BLE)

1. 片方の XIAO BLE を USB で PC に接続する
2. リセットボタンを**素早く2回押す**(ダブルタップ)→ ブートローダーモードに入り、`XIAO-SENSE` という名前の USB ドライブとしてマウントされる
3. そのドライブに対応する uf2 ファイルをドラッグ&コピーする → コピー完了後、自動で再起動して書き込み完了
4. もう片方も同様に行う

左右どちらを先に書き込んでも動作しますが、必ず**両方とも**新しいファームウェアに更新してください。片側だけ更新すると split の通信仕様がずれて動かないことがあります。

## settings_reset について

通常の更新では使いません。更新後に左右間の Bluetooth 接続がうまく回復しない場合のみ、以下の手順で使用します。

1. 左右両方に `settings_reset` の uf2 を書き込む(ペアリング情報が消去される)
2. 改めて左右それぞれに通常のファームウェアを書き戻す
3. 両方の電源を入れると左右が再ペアリングされる

settings_reset を使うと PC やスマホとの Bluetooth ペアリングも消えるため、ホスト側でも再ペアリングが必要になります。

# COROPITを使用するへ

COROPITを使用する方は以下のようにコードを編集してください。

mona2_r.overlay

修正前
```
  trackball_central: trackball_central@0 {
        status = "okay";
        compatible = "pixart,pmw3610";  //トラボセンサ用のドライバとバインド
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 2 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>; //P0.02を指定(MOTION)
        cpi = <600>;
        //swap-xy;
        //invert-x; //COROPIT版ではコメントアウトを外す
        //invert-y; //COROPIT版ではコメントアウトを外す
        evt-type = <INPUT_EV_REL>;
        x-input-code = <INPUT_REL_X>;
        y-input-code = <INPUT_REL_Y>;
    };
};

```
**修正後**
```
  trackball_central: trackball_central@0 {
        status = "okay";
        compatible = "pixart,pmw3610";  //トラボセンサ用のドライバとバインド
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 2 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>; //P0.02を指定(MOTION)
        cpi = <600>;
        //swap-xy;
        invert-x; //COROPIT版ではコメントアウトを外す
        invert-y; //COROPIT版ではコメントアウトを外す
        evt-type = <INPUT_EV_REL>;
        x-input-code = <INPUT_REL_X>;
        y-input-code = <INPUT_REL_Y>;
    };
};

```

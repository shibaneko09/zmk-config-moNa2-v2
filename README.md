# moNa2 v2 (DYA Studio 対応版)

> [!NOTE]
> **このforkでの変更点**（[sayu-hub/zmk-config-moNa2-v2](https://github.com/sayu-hub/zmk-config-moNa2-v2) からの差分）
>
> - **COROPIT**：PMW3610版は元リポジトリの設定（`invert-x` のみ有効）をそのまま使います。カーソルの上下が逆の場合は、DYA Studio の `mouse` 設定で Y 反転を切り替えてください。PAW3222版にはドライバ側の軸反転設定がないため、向きが合わない場合は同じく `mouse` 設定の X/Y 反転で調整します。
> - **トラックボールスクロール**：スクロールレイヤー（`SCRL_W` / `SCRL_M`）では縦横どちらにもスクロールします。縦横の向きが揃うよう横スクロールを反転しています（`mona2_r.overlay` の `x-invert`、macOS 実機で確認した向き）。macOS の「ナチュラルなスクロール」設定でも向きは変わります。好みに合わない場合は DYA Studio の `scroll` 設定で X/Y 反転を切り替えられます。
> - **依存の固定**：`config/west.yml` の可変ブランチ指定（`v0.3-branch+dya`、`main` など）を、動作確認済みのコミットに固定しています。
> - 書き込むファイル：右手（トラックボール側）は `mona2_r-pmw3610.uf2` か `mona2_r-paw3222.uf2` のうちセンサーに合う方、左手は名前が `mona2_l` で始まる uf2 です（`mona2_r-logging` はデバッグ用なので通常は使いません）。DYA Studio で保存した設定は右手の Flash に残るため、初期値に戻したい場合は `settings_reset` を書き込んでください。

このリポジトリは [moNa2 v2](https://github.com/sayu-hub/zmk-config-moNa2) 用 ZMK Config を、[cormoran](https://github.com/cormoran) さんが公開している **[DYA Studio](https://studio.dya.cormoran.works/)** に対応させたものです。

DYA Studio は ZMK Studio をベースに、

- キーマップ／レイヤーの GUI 編集
- トラックボール (PMW3610) のスクロール量・軸反転・自動マウスレイヤーなど **Runtime Input Processor 経由での動的設定**
- バッテリ履歴の閲覧
- BLE プロファイル管理 / Settings RPC

を Web から行えるようにした拡張版です。

## キーマップ

![moNa2 keymap](keymap-drawer/mona2.svg)

[keymap-drawer YAML](keymap-drawer/mona2.yaml) / [SVGを開く](keymap-drawer/mona2.svg)

## DYA Studio の使い方

1. 本リポジトリをビルドし、生成された `mona2_r-...uf2`（中央側＝右手）と `mona2_l-...uf2`（周辺側＝左手）をそれぞれの XIAO BLE に書き込みます。
2. 中央側（右手）を **USB ケーブル** で PC に接続します。
3. Chrome / Edge などの WebUSB 対応ブラウザで **[https://studio.dya.cormoran.works/](https://studio.dya.cormoran.works/)** を開きます。
4. キーボード側で `BLE` レイヤー(レイヤー10/11)を有効にし、右下に配置した **`&studio_unlock`** キーを押してアンロック。
5. DYA Studio 側で「Connect」を押し、USB デバイスとして mona2 を選択。
6. キーマップ／トラックボール設定／バッテリ履歴を編集できます。

### このリポジトリで対応済みの内容

| 対応項目 | ファイル / 箇所 |
| --- | --- |
| ZMK 本体を cormoran さんの fork (`v0.3-branch+dya`) に切替 | `config/west.yml` |
| DYA Studio モジュール 4 本を west.yml に追加 (ble-management / battery-history / settings-rpc / runtime-input-processor) | `config/west.yml` |
| Studio 系 CONFIG (`CONFIG_ZMK_STUDIO`, `CONFIG_ZMK_BLE_MANAGEMENT`, `CONFIG_ZMK_BATTERY_HISTORY`, `CONFIG_ZMK_SETTINGS_RPC`, `CONFIG_ZMK_RUNTIME_INPUT_PROCESSOR`, `CONFIG_ZMK_RUNTIME_INPUT_PROCESSOR_STUDIO_RPC`) | `config/mona2_r.conf` |
| トラックボール処理を Runtime Input Processor に置換 | `boards/shields/mona2/mona2.dtsi`, `mona2_r.overlay` |
| `&studio_unlock` を `ble_win` / `ble_mac` レイヤー右上に配置 | `config/mona2.keymap` |
| 全レイヤーに `display-name` を設定 (DYA Studio 上で `WIN / MAC / NUM_W / NUM_M / MOUSE_W / MOUSE_M / SCRL_W / SCRL_M / FN_W / FN_M / BLE_W / BLE_M` と表示) | `config/mona2.keymap` |
| `studio-rpc-usb-uart` snippet (中央側) | `build.yaml` |

> **Note:**
> - DYA Studio で行った変更は **中央側 (右手) の Flash** に保存されます。初期化したい場合は `settings_reset` ファームウェアを書き込んでください。
> - **`v0.3-branch+dya` / 各 DYA module の revision・CONFIG 名は調査ベースの値**です。ビルドが通らない場合は、おぐ さんの記事 [DYA Studioを導入してみるぞ！moNa2編](https://note.com/heace/n/nf06b797ffa79) を参照して、各 revision/CONFIG 名を最新版に合わせてください。

---

## トラックボール (COROPIT) を使用する場合

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

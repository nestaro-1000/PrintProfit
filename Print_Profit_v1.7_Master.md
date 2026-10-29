# Print Profit v1.7 — マスター版

このファイルはChatGPT上で動作していたPrint Profit v1.7のUI・計算仕様を保存したマスターです。

## 新規チャットでの再現方法

このファイルを新規チャットに添付し、次のように依頼してください。

> 添付したPrint Profit v1.7マスターのDILコードを使い、インタラクティブなUIを再現してください。入力項目、計算式、初期値、配置、サイズ、ボタン動作を変更せず、コードを省略せずに表示してください。

※ 表示環境の更新により見た目にわずかな差が出る可能性はあります。これはChatGPT上のUI定義であり、単独でMac上で実行できるHTML/Pythonアプリではありません。

## 保存対象の仕様

- 商品名、材料価格、材料重量、作品重量を入力
- 「価格 → 利益率」「利益率 → 価格」を切り替え
- 後者は5〜95%・1%刻みのスライダー
- 計算モード切替でも縦サイズが変わらない48px固定の入力・結果エリア
- 原価・販売価格・差引利益を3列で表示
- 結果をコピー
- 全クリアは確認後に実行。確認行の高さを常時18px確保し縦サイズを固定
- フィラメント代のみで計算

## DILソース（v1.7 原版）

```jsx
{@body const [name,setName]=DIL.useState("キーホルダー")}
{@body const [fp,setFp]=DIL.useState("2500")}
{@body const [fw,setFw]=DIL.useState("1000")}
{@body const [iw,setIw]=DIL.useState("20")}
{@body const [mode,setMode]=DIL.useState("price")}
{@body const [price,setPrice]=DIL.useState("200")}
{@body const [target,setTarget]=DIL.useState(75)}
{@body const [confirmClear,setConfirmClear]=DIL.useState(false)}
{@body const valid=[fp,fw,iw].every(v=>v.trim()!==""&&Number.isFinite(Number(v))&&Number(v)>=0)&&Number(fw)>0}
{@body const cost=valid?Number(fp)/Number(fw)*Number(iw):0}
{@body const priceValid=price.trim()!==""&&Number.isFinite(Number(price))&&Number(price)>0}
{@body const sale=mode==="margin"?Math.ceil(cost/(1-target/100)):Number(price)}
{@body const ok=valid&&(mode==="margin"?sale>0:priceValid)}
{@body const profit=sale-cost}
{@body const margin=sale>0?profit/sale*100:0}
{@body const yen=v=>new Intl.NumberFormat("ja-JP",{maximumFractionDigits:2}).format(v)+"円"}
<box border radius="xl" padding={3} gap={3}>
  <row align="center" justify="between">
    <row align="center" gap={2}>
      <icon name="printer" color="secondary"/>
      <title size="lg">Print Profit</title>
    </row>
    <text color="tertiary" size="2xs">v1.7</text>
  </row>
  <box gap={1}>
    <label>商品名</label>
    <input size="lg" value={name} onChange={setName} placeholder="例：ネコのキーホルダー"/>
  </box>
  <grid columns={3} gap={2}>
    <grid-item>
      <label size="xs">材料価格 ¥</label>
      <input size="lg" inputType="number" value={fp} onChange={setFp} placeholder="例：2500"/>
    </grid-item>
    <grid-item>
      <label size="xs">材料重量 g</label>
      <input size="lg" inputType="number" value={fw} onChange={setFw} placeholder="例：1000"/>
    </grid-item>
    <grid-item>
      <label size="xs">作品重量 g</label>
      <input size="lg" inputType="number" value={iw} onChange={setIw} placeholder="例：20"/>
    </grid-item>
  </grid>
  <segmented-control size="lg" block value={mode} onChange={setMode} options={[{"label":"価格 → 利益率","value":"price"},{"label":"利益率 → 価格","value":"margin"}]}/>
  <grid columns={2} gap={2}>
    <grid-item>
      <box gap={1}>
        <row align="center" justify="between">
          <label>{mode==="price"?"販売価格（円）":"目標利益率"}</label>
          {#if mode==="margin"}
            <text weight="semibold" size="sm" tabularNums>{target}%</text>
          {/if}
        </row>
        <box height="48px" justify="center">
          {#if mode==="price"}
            <input size="lg" inputType="number" value={price} onChange={setPrice} placeholder="例：200"/>
          {:else}
            <slider min={5} max={95} step={1} value={target} onChange={setTarget}/>
          {/if}
        </box>
      </box>
    </grid-item>
    <grid-item>
      <box gap={1}>
        <label>{mode==="price"?"利益率":"必要販売価格"}</label>
        <box background="surface-secondary" radius="md" padding={3} justify="center" height="48px">
          <title size="lg" tabularNums>{ok?(mode==="price"?margin.toFixed(1)+"%":yen(sale)):"—"}</title>
        </box>
      </box>
    </grid-item>
  </grid>
  <box background="surface-secondary" radius="lg" padding={3}>
    <grid columns={3} gap={2}>
      <grid-item>
        <text color="secondary" size="xs">原価</text>
        <text weight="semibold" tabularNums>{ok?yen(cost):"—"}</text>
      </grid-item>
      <grid-item>
        <text color="secondary" size="xs">販売価格</text>
        <text weight="semibold" tabularNums>{ok?yen(sale):"—"}</text>
      </grid-item>
      <grid-item>
        <text color="secondary" size="xs">差引利益</text>
        <text weight="semibold" color={profit>=0?"success":"danger"} tabularNums>{ok?yen(profit):"—"}</text>
      </grid-item>
    </grid>
  </box>
  <box gap={1}>
    {#if confirmClear}
      <row align="center" justify="center" height="18px">
        <text size="xs" color="danger" weight="medium">入力内容をすべて消去しますか？</text>
      </row>
    {:else}
      <box height="18px"/>
    {/if}
    <row gap={2}>
      <box flex="1">
        {#if confirmClear}
          <button block size="lg" color="secondary" variant="outline" onClick={()=>setConfirmClear(false)}>キャンセル</button>
        {:else}
          <button block size="lg" disabled={!ok} onClick={()=>GenUI.copy(`【${name.trim()||"商品名未設定"}】\nフィラメント価格：${yen(Number(fp))}／${fw}g\n作品重量：${iw}g\n原価：${yen(cost)}\n販売価格：${yen(sale)}\n材料費差引後の利益：${yen(profit)}\n材料費ベースの利益率：${margin.toFixed(1)}%`)}>
            <icon name="copy" inline size="sm"/> 結果をコピー
          </button>
        {/if}
      </box>
      {#if confirmClear}
        <button size="lg" color="danger" onClick={()=>{setName("");setFp("");setFw("");setIw("");setPrice("");setTarget(75);setMode("price");setConfirmClear(false)}}>クリアする</button>
      {:else}
        <button size="lg" color="secondary" variant="outline" onClick={()=>setConfirmClear(true)}>
          <icon name="eraser" inline size="sm"/> 全クリア
        </button>
      {/if}
    </row>
  </box>
  <caption>※ フィラメント代のみで計算。電気代・その他経費は含みません。</caption>
</box>
```

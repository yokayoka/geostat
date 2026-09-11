# geostat タイル配信

能登半島 真脇(Machino)地区の地質構造解析結果(Φ・μ)を、他のWebGISから読み込めるXYZタイルとして公開するリポジトリです。
元データや解析手法は [IPU_Share/geostat](https://github.com/yokayoka) プロジェクトの `data_preparation.ipynb` / `kriging01.ipynb` を参照してください(Nemoto et al. 2000 の手法に基づく)。

## レイヤー

| レイヤー | ディレクトリ | 内容 | 値の範囲 |
|---|---|---|---|
| Φ (phi) | `tiles/phi_deg/` | 地形と層理面の法線がなす角(0°=地形と層理面がほぼ平行=すべり面として不安定になりやすい) | 0–90° |
| μ (mu)  | `tiles/mu_deg/`  | 地形走向と交線方向のなす角 | 0–90° |

- カラースケールは共通で 0–90° 固定(実データの最大値が90°未満でも、角度の理論範囲でスケーリングしています)。
- 配色: `RdYlBu`(赤=0°側、青=90°側)。凡例画像は `legend/phi_deg_legend.png`, `legend/mu_deg_legend.png`。
- **タイル全体に70%の不透明度(アルファ=178/255)を適用しています。** 元のラスタは能登半島の海岸線を越えて海域まで値が入った矩形グリッドであり、海域側は陸域データの外挿にすぎない(意味を持たない)ため、半透明にして背景の地図(海岸線)が透けて見えるようにしています。厳密な陸域マスクは適用していません。

## タイル仕様

- 形式: PNG (256×256), スキーム: **XYZ (Google/OSM方式, y軸は上から下)** — TMSではありません
- 座標系: **EPSG:3857 (Web Mercator)** — 元データは EPSG:6675 (JGD2011 平面直角座標系VII系) で計算し、`gdal.Warp` (bilinear) で再投影しています
- ズームレベル: 8–15
- 元データ解像度: 約50m(グリッド間隔)、EPSG:3857再投影後は約63m/pixel

## 他のWebGISからの利用方法

GitHub Pages を有効化している場合:

```
https://yokayoka.github.io/geostat/tiles/phi_deg/{z}/{x}/{y}.png
https://yokayoka.github.io/geostat/tiles/mu_deg/{z}/{x}/{y}.png
```

GitHub Pages を使わない場合は raw.githubusercontent.com からも読み込めます(小規模タイルセット向け、レート制限あり):

```
https://raw.githubusercontent.com/yokayoka/geostat/main/tiles/phi_deg/{z}/{x}/{y}.png
https://raw.githubusercontent.com/yokayoka/geostat/main/tiles/mu_deg/{z}/{x}/{y}.png
```

### Leaflet の例

```js
L.tileLayer('https://yokayoka.github.io/geostat/tiles/phi_deg/{z}/{x}/{y}.png', {
  minZoom: 8, maxZoom: 15, maxNativeZoom: 15,
  attribution: 'Noto geostat (Φ)'
}).addTo(map);
```

`index.html` にLeafletを使った表示例(レイヤー切替・凡例付き)を置いています。GitHub Pagesを有効にすると
`https://yokayoka.github.io/geostat/` でそのまま確認できます。

## 再生成

タイルは元の `phi_deg.tif` / `mu_deg.tif` (EPSG:6675, 値ラスタ)から以下の手順で作成しています。

1. `gdal.Warp` で EPSG:3857 に再投影 (resampling=bilinear)
2. 0–90°スケール・`RdYlBu`カラーマップでRGBA化(不透明度70%)
3. `gdal2tiles.py -p mercator -z 8-15 -r bilinear --xyz` でXYZタイル化
4. `matplotlib` でカラーバー凡例画像を生成

# SPENVIS を用いた宇宙陽子線環境の算出手順

**GEO・ISS 軌道における捕捉陽子線・太陽陽子線の算出と合算（AP-8 / AP-9 IRENE / ESP-PSYCHIC）**

このリポジトリは、[ESA SPENVIS](https://www.spenvis.oma.be) を使って軌道を定義し、捕捉陽子・太陽陽子のフラックス／フルエンスを取得して合算する手順をまとめたものです。

**SPENVIS 画面キャプチャについて:** 操作手順用のスクリーンショット（`inputs/images/` のうち SPENVIS Web 画面の撮影にあたる図）には、画面上に表示されている**コピーライト・クレジット・利用に関する表記**が含まれます。これらは **SPENVIS の提供元（European Space Agency が運営する SPENVIS 等、サービスが示す権利者）** に帰属します。第三者への提供や再掲載を行う場合は、[SPENVIS のサイト](https://www.spenvis.oma.be)で示される利用条件・著作権ポリシーを確認し、画面上の表記を改変・削除しないよう留意してください。

## このフォルダの内容

| パス | 説明 |
|------|------|
| **本文（この README）** | 操作手順・モデル設定・単位と合算・適用例を一覧できるプロジェクトのメイン説明です。 |
| [inputs/guide_spenvis.md](./inputs/guide_spenvis.md) | 同等構成の Markdown 原本です。**README と内容を対応させて編集・保守**してください。 |
| [inputs/images/](./inputs/images/) | スクリーンショットおよび結果プロット（`fig01`〜`fig12` の PNG。**README とガイドの画像リンクはすべてこのフォルダを参照します**。 |
| [inputs/geo_10yr_combined_proton_ap9.csv](./inputs/geo_10yr_combined_proton_ap9.csv) | GEO 10 年・Combined proton の例データ（AP-9 + ESP-PSYCHIC）。 |
| [inputs/iss_05yr_combined_proton.csv](./inputs/iss_05yr_combined_proton.csv) | ISS 軌道の Combined proton 例データ（ファイル名は `05yr` ですが、本文の適用例は 180 日条件です。条件とデータを揃える場合はファイル名または計算条件を更新してください）。 |
| [docs/guide_spenvis.docx](./docs/guide_spenvis.docx) | Word 形式の別出力（任意）。 |

## 目次

- [１．概要](#readme-sec-1)
- [２．共通手順：軌道の定義](#readme-sec-2)
- [３．捕捉陽子線の算出](#readme-sec-3)
- [４．太陽陽子線の算出](#readme-sec-4)
- [５．データ整形と合算](#readme-sec-5)
- [６．適用例1: GEO軌道（10年）](#readme-sec-6)
- [７．適用例2: ISS軌道（180日）](#readme-sec-7)
- [８．GEOとISSの比較](#readme-sec-8)
- [９．注意事項](#readme-sec-9)
- [参考文献](#readme-refs)

---
<a id="readme-sec-1"></a>

## １．概要

### 1.1 SPENVISとは

SPENVIS（https://www.spenvis.oma.be）は、ESAが提供するWebベースの宇宙環境解析ツールである。放射線環境、帯電、大気、磁場などのモデルを統合的に利用できる。利用にはアカウント登録（無料）が必要。

### 1.2 軌道別の陽子線環境の特徴

| Orbit | Trapped protons | Solar protons | Dominant source |
|-------|----------------|---------------|-----------------|
| ISS (410km, 51.6°) | Strong (SAA) | Zero (geomag. shielding 99.9%) | Trapped only |
| GEO (35,793km) | Sub-MeV only (L=6.6) | Dominant above ~2.5 MeV | Both contribute |
| Interplanetary | None | Dominant | Solar only |

*Table 1: 軌道別の陽子線環境*

### 1.3 使用するモデル

| Model | Target | Note |
|-------|--------|------|
| AP-8 MIN/MAX | Trapped protons | 1960s data, legacy standard |
| AP-9 (IRENE) | Trapped protons | Van Allen Probes data, recommended |
| AE-8 / AE-9 | Trapped electrons | GEO: dominant for Dd |
| ESP-PSYCHIC | Solar protons | ECSS standard, CL-dependent |
| SAPPHIRE | Solar protons | Alternative to ESP-PSYCHIC |

*Table 2: 主な放射線環境モデル*

---

<a id="readme-sec-2"></a>

## ２．共通手順：軌道の定義

### Step 1: プロジェクト作成

SPENVISログイン後、Project > Newでプロジェクトを作成。Spacecraft trajectoriesをクリック。

![Model Packages](./inputs/images/fig01_model_packages.png)

*Fig. 1: Model Packagesページ*

### Step 2: ミッション定義

Mission durationを設定する。**Representative trajectory durationを「1 days」に設定**することが重要。特にISSなどのLEO軌道では、この値が小さいとSAA通過がサンプリングされず、捕捉粒子フラックスがゼロとなる。

![Mission definition](./inputs/images/fig02_mission_def.png)

*Fig. 2: Mission definition画面*

### Step 3: 軌道タイプの選択

| Orbit | Type | Key parameters |
|-------|------|---------------|
| GEO | geostationary | Auto-set (35,793 km, i=0°) |
| ISS | general (circular) | Alt=410 km, Inc=51.6° |
| SSO | heliosynchronous | Altitude, LTAN |

*Table 3: 軌道タイプの設定例*

![Orbit type](./inputs/images/fig03_orbit_type.png)

*Fig. 3: 軌道タイプ選択（GEOの例）*

### Step 4: 軌道生成・確認

設定確認後、Runをクリック。Mission overviewでパラメータを確認する。

![Mission overview](./inputs/images/fig04_mission_overview.png)

*Fig. 4: Mission overview（GEOの例）*

---

<a id="readme-sec-3"></a>

## ３．捕捉陽子線の算出

### 3.1 AP-8モデル

OutputページのRadiation sources and effectsセクションから**Trapped proton and electron fluxes**を選択する。

![Output page](./inputs/images/fig05_output_page.png)

*Fig. 5: Tables and plotsページ*

| Parameter | Setting | Note |
|-----------|---------|------|
| Proton model | AP-8 MIN / MAX | MIN=solar min, MAX=solar max |
| Electron model | AE-8 MIN / MAX | Dd計算に必要 |

*Table 4: AP-8モデル設定*

AP-8の出力は**flux**（cm⁻² s⁻¹ MeV⁻¹）であり、ミッション期間（秒）を掛けてfluenceに変換する必要がある。

### 3.2 AP-9（IRENE）モデル（推奨）

AP-9はVan Allen Probesの観測データを反映した新しいモデルであり、特に高L値領域（GEO等）での精度が向上している。OutputページのIRENE AE9/AP9 Macroを選択する。

![AP-9 setup](./inputs/images/fig06_ap9_setup.png)

*Fig. 6: IRENE AE9/AP9パラメータ設定画面*

| Parameter | Setting | Note |
|-----------|---------|------|
| Model version | 1.50 | Van Allen Probes data |
| Model run mode | perturbed | Statistical variation included |
| Nr. of runs | 20 | Monte Carlo trials |
| Aggregate | percentile | |
| Percentile | 50 | 50th=mean, 95th=design |
| Energies | default SPENVIS energies | AP-8と同じグリッドで比較しやすい |

*Table 5: AP-9モデル設定*

### 3.3 AP-8とAP-9の比較（GEO）

GEO軌道でのAP-8とAP-9の比較結果を以下に示す。0.1〜0.7 MeVではAP-9はAP-8の0.4〜0.8倍とやや低いが、同オーダーである。1 MeV以上ではAP-9の方が高く、AP-8が2 MeVでゼロとなるのに対しAP-9は7 MeVまで非ゼロの値を予測する。

![AP-8 vs AP-9](./inputs/images/fig07_ap8_vs_ap9.png)

*Fig. 7: GEOにおけるAP-8とAP-9の捕捉陽子線フラックス比較*

---

<a id="readme-sec-4"></a>

## ４．太陽陽子線の算出

OutputページのSolar particle mission fluencesを選択する。

| Parameter | Setting | Note |
|-----------|---------|------|
| Model | ESP-PSYCHIC (total fluence) | ECSS standard |
| Ion range | H - U | Protons sufficient for Dd |
| Confidence level | 90 (%) | 95% for conservative design |
| Magnetic shielding | yes | GEO: atten.~0.994, ISS: 0.001 |

*Table 6: ESP-PSYCHIC設定*

![ESP-PSYCHIC](./inputs/images/fig08_esp_psychic.png)

*Fig. 8: ESP-PSYCHICモデル選択画面*

ESP-PSYCHICの出力は直接**fluence**（cm⁻² MeV⁻¹）であり、AP-8/AP-9のようなfluxからの変換は不要である。

**注意:** ISS軌道（410 km, 51.6°）では地磁気遮蔽の減衰率が0.0009（99.9%遮蔽）であり、太陽陽子線フルエンスは全エネルギーでゼロとなる。一方、GEOでは減衰率が0.994でほぼ全量が到達する。

---

<a id="readme-sec-5"></a>

## ５．データ整形と合算

### 5.1 単位の統一

| Model | Output unit | Conversion |
|-------|-------------|------------|
| AP-8 / AP-9 | Flux (cm⁻² s⁻¹ MeV⁻¹) | × mission seconds |
| ESP-PSYCHIC | Fluence (cm⁻² MeV⁻¹) | No conversion needed |

*Table 7: 単位の違いと変換*

| Duration | Seconds |
|----------|---------|
| 0.5 years (180 days) | 1.556 × 10⁷ |
| 1 year | 3.156 × 10⁷ |
| 5 years | 1.578 × 10⁸ |
| 10 years | 3.156 × 10⁸ |

*Table 8: ミッション期間の秒数変換*

### 5.2 合算方法

Combined fluenceは、各エネルギー点でTrapped fluenceとSolar fluenceを足し算する。

F<sub>combined</sub>(E) = F<sub>trapped</sub>(E) + F<sub>solar</sub>(E)

TrappedのエネルギーグリッドがSolarと異なる場合は、対数線形補間でSolarのグリッドに合わせる。

---

<a id="readme-sec-6"></a>

## ６．適用例1: GEO軌道（10年）

### 6.1 計算条件

| Item | Value |
|------|-------|
| Orbit | GEO (35,793 km, i=0°) |
| Duration | 10 years |
| Trapped proton | AP-9 V1.50, 50th percentile |
| Solar proton | ESP-PSYCHIC, 90% CL |
| Trapped electron | AE-9 (for Dd calculation) |

### 6.2 Trapped vs Solarの支配領域

GEOでは約2.5 MeVを境に、それ以下ではTrapped protonが支配的、それ以上ではSolar protonが支配的である。この結論はAP-8とAP-9の両モデルで一致しており、モデルに依存しない堅牢な結果である。

![Trapped vs Solar dominance](./inputs/images/fig09_dominance.png)

*Fig. 9: GEOのTrapped vs Solar支配領域（AP-9使用）*

### 6.3 Combined Proton Fluence

![GEO combined](./inputs/images/fig10_geo_combined.png)

*Fig. 10: GEO 10年間Combined Proton Fluence（AP-9 + ESP-PSYCHIC）*

---

<a id="readme-sec-7"></a>

## ７．適用例2: ISS軌道（180日）

### 7.1 計算条件

| Item | Value |
|------|-------|
| Orbit | ISS (410 km, i=51.6°, circular) |
| Duration | 180 days (0.5 years) |
| Trajectory sampling | 1 day (15.52 orbits) |
| Trapped proton | AP-8 MIN |
| Solar proton | ESP-PSYCHIC, 90% CL |

**重要:** ISS軌道ではRepresentative trajectory durationを必ず**1 day**に設定する。ISSの軌道周期は約1.55時間であり、1周回や数分ではSAA通過がサンプリングされず、捕捉粒子フラックスがゼロとなる。

### 7.2 結果

ISS軌道では太陽陽子線フルエンスが**全エネルギーでゼロ**となった。地磁気遮蔽の減衰率が0.0009（99.9%遮蔽）であり、太陽陽子線はほぼ到達しない。したがって、**ISS軌道のCombined proton = Trapped protonのみ**である。捕捉陽子線は0.1〜300 MeVの広いエネルギー範囲にわたり、SAA通過による軌道平均フルエンスである。

![ISS combined](./inputs/images/fig11_iss_combined.png)

*Fig. 11: ISS 180日間Combined Proton Fluence（= Trapped only）*

---

<a id="readme-sec-8"></a>

## ８．GEOとISSの比較

GEOとISSの捕捉陽子線フラックスを比較すると、以下の特徴が確認できる。

![GEO vs ISS](./inputs/images/fig12_geo_vs_iss.png)

*Fig. 12: GEO vs ISS 捕捉陽子線フラックス比較*

**GEO（L=6.6）:** 低エネルギー（0.1〜2 MeV）のみに捕捉陽子線が存在。絶対値は大きいが、エネルギー範囲が狭い。AP-8とAP-9で同程度の値が得られ、物理的に存在する集団と考えられる。

**ISS（410 km, 51.6°）:** 軌道平均フラックスは低いが（SAA duty cycle約10-15%による希釈）、0.1〜300 MeVの広いエネルギー範囲に捕捉陽子線が存在する。

---

<a id="readme-sec-9"></a>

## ９．注意事項

**軌道サンプリング:** LEO軌道ではRepresentative trajectory durationを1 dayに設定すること。短い設定ではSAAがサンプルされず、捕捉粒子フラックスがゼロとなる。

**AP-8 flux→fluence変換:** AP-8/AP-9の出力はfluxであり、ミッション期間（秒）を掛けてfluenceに変換する。ESP-PSYCHICは直接fluenceで出力される。

**信頼水準:** ESP-PSYCHICの出力はCLに依存する。90%と95%で数倍の差がある。AP-8/AP-9にはこの概念はない（AP-9はpercentileで指定）。

**モデル選択:** 新規設計にはAP-9（IRENE）を推奨する。AP-8は1960年代のデータに基づくレガシーモデルであり、特に高L値領域での精度が不十分である。

**電子線:** GEOでの太陽電池劣化予測では、陽子線に加えて捕捉電子線（AE-8/AE-9）の計算も必要。GEOでは捕捉電子線がDdの主要因となりうる。

---

<a id="readme-refs"></a>

## 参考文献

[1] SPENVIS - Space Environment Information System, https://www.spenvis.oma.be

[2] SPENVIS Help: Solar proton models, https://www.spenvis.oma.be/help/background/flare/flare.html

[3] SPENVIS Help: Trapped particle radiation, https://www.spenvis.oma.be/help/background/traprad/traprad.html

[4] IRENE AE9/AP9 V1.58, https://www.vdl.afrl.af.mil/programs/ae9ap9/downloads.php

[5] G. P. Ginet et al., "AE9, AP9 and SPM: New models for specifying the trapped energetic particle and space plasma environment," Space Sci. Rev., 179, 579-615, 2013.

[6] M. A. Xapsos et al., "Probability Model for Cumulative Solar Proton Event Fluences," IEEE TNS, 47(3), 486-490, 2000.

[7] P. Jiggens et al., "Updated Model of the Solar Energetic Proton Environment in Space," JSWSC, 8, A31, 2018.

[8] D. M. Sawyer and J. I. Vette, "AP-8 Trapped Proton Environment," NSSDC/WDC-A-R&S 76-06, 1976.

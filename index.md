単一のコンシューマ機でどこまで解けるか、決算と現場のどちらが正しいか、専門用語は本当に必要か。
手を動かして確かめたことを置いています。

---

## Technical Reports

囲碁3編と将棋1編。囲碁の3編は単一の RTX 5090（Blackwell sm_120 / Core Ultra 9 285K, WSL2）による求解で、
前2編は同じマシンで**律速点が逆になる**対、3編目はその証明そのものを素材に、
ニューラルネットが何を持っていて何を持っていないかを測ったものです。
将棋の1編は速さではなく**目的関数**を疑ったもので、
規則が変わっているのにエンジンの目標が変わっていない、という話です。

### [Solving 7×7 Killall-Go Opening JA on a Single RTX 5090](https://soy-tuber.github.io/killallgo-rtx5090/)

NeurIPS 2023 の分散 killall-go ソルバ（Wu et al., *Game Solving with Online Fine-Tuning*）を
Docker 非使用で WSL2 にネイティブ再現し、7×7 開局 **JA** を完全求解。判定は原著実測と一致（*loss*）。

コード・問題・初期モデル・設定を固定したまま計算基盤だけ 2017年世代（1080Ti×12 / 384スロット）から
2026年世代（5090×1 / 24スロット）に替わるため、**世代間システムベンチマーク**として読めます。
スロット当たりノード処理 **2.00×**、必要探索量 **0.59×**。後者はトレーナを同一GPUに同居させたことによる
モデル配信頻度 **19×** の寄与と見ています。**GPU律速**。

### [Solving Cho Chikun Life-and-Death Problems on a Single RTX 5090](https://soy-tuber.github.io/tsumego-rtx5090/)

RZ（relevance-zone）ベースの死活ソルバ（Shih et al., IEEE ToG 2025）で趙治勲事典117問をスイープ。
ハード世代とアルゴリズム（RZS-TT / RZS-PT）だけを振る **2×2 行列**で、両者の寄与を交絡なしに分離。

NN は 765,523 パラメータと極小で GPU 実利用率は約23%。つまりこれは GPU ベンチではなく
**CPU世代ベンチ**（Haswell 2014 → Arrow Lake 2024、L2 キャッシュ12倍）で、killall-go 編とは律速点が逆です。
`USE_POTENTIAL_RZONE` の決定的 segfault を特定・修正し、知識フラグが証明目的には不健全であることも示しました。
当初は28問が「手法律速」で残りましたが、**スレッド数を2→20に上げるだけでこの結論は覆り、117/117 全問証明**に至っています。

### [Right Zone, Wrong Verdict](https://soy-tuber.github.io/rz-katago-rtx5090/)

RZ（relevance zone = 証明が依存する点集合）は現行の死活ソルバの中核ですが、
**既に解けた局面にしか存在しない**ため大盤へスケールしません。
「NN で先読みできないか」は自然な問いでも、正解データを持っている人がほとんどいない。
第1弾で求解した JA の証明木は全証明済みノードに RZ を持っているので、それを正解に KataGo を測りました。

当初の仮説「不確実性の帯 ≒ RZ」は**乱択以下で反証**（AUC 0.357）。
一方で**符号付き ownership は AUC 0.910 で復元**し、幾何ベースライン（0.807）を明確に上回ります。
RZ は争点ではなく**生死が問われる一団の潜在的勢力圏**でした。
副産物として、証明済みの負例1,021件を加えた均衡集合で測ると
**勝敗判定は AUC 0.411 — 順序が反転**しています。同じ盤面で、
**どこが戦場かは AUC 0.910、誰が勝つかは AUC 0.411**。

### [入玉宣言法24点法における将棋AIの到達限界](https://soy-tuber.github.io/nyugyoku24/)

プロ公式棋戦は2013年10月から**24点法**ですが、将棋AIは**27点法**を前提に作られています。
floodgate の公開棋譜 1,868,968局から相入玉 16,780局を抽出し、その差が実戦でどう現れるかを測りました。

24点法の勝ち（31点）は定義上、相手から最低4点を奪わなければ届きません。各陣営の駒の総点が27点しかないからです。
一方「敵陣に10枚」は打ち込みで点数を失わずに進みます。**2つの宣言条件は難易度が正反対**で、
10枚条件を満たす局面が 8.9% しかないのは難しいからではなく、**既存エンジンがそれを目指していない**ためです。
所有31点ある側の **58%** が、勝てる材料を持ちながら変換できずに終わっています。

これは振る舞いの証拠ですが、**dlshogi の入玉用入力特徴には 先手28点 / 後手27点 という閾値が
焼き込まれています**。27点法が、エンジンの振る舞いだけでなく**入力表現の水準**で
前提になっている、ということです。

副産物として、floodgate が2024年に `Max_Moves` を 256→512 に変えていたことが分かりました。
これだけで宣言到達率が **19.4% → 41.4%** に跳ね上がるため、2024年を挟んだ年次比較は成立しません。
あわせて24点法を目的関数に持つ終盤特化エンジンを実装しています（GPL-3.0）。

---

## Notes

### [日産分析ノート](https://soy-tuber.github.io/nissan-notes/)

日産自動車（7201）をめぐる分析ノート。決算数字の逆算と、現場からの検証。

### [定期巡回型回収業務における配車設計](https://soy-tuber.github.io/recyclehub-paperless/)

配車設計を最適化（ルート・頻度）・予測（在庫推論）・転移（未知のマップへの適応）の三問題に分解し、
閉形式・混合整数計画・階層ベイズ・方策学習の使い分けと検証方法を論じたもの。

---

## References

### [AIコーディング辞典](https://soy-tuber.github.io/dictionary-of-ai-coding-ja/)

AIコーディングの語彙を平易な言葉に翻訳した辞典。
[AI Coding Dictionary](https://aicodingdictionary.com) の日本語版。

### [中学受験 学習ガイド](https://soy-tuber.github.io/edojo-reading-guide/)

理科・社会・国語・算数の出題傾向別おすすめ本リスト。

### [SICC 構造台帳](https://soy-tuber.github.io/sicc-ledger/)

シンガポール国際商事裁判所の判決148件の**事実認定部だけ**を構造化した台帳。
投資ストラクチャー47件（ビークル連鎖×法域×失敗モード）と、
訴訟という現実の敵対的攻撃を受けた**条項の戦績**15件（条項×帰趨×準拠法）。
載っている構造は「よく使われる」ではなく「よく揉める」の標本である点に注意。

### [司法試験租税法まとめノート](https://soy-tuber.github.io/taxlaw-notes/)

弘文堂『ケースブック租税法』（第6版）の第1編「租税法の基礎理論」から第3編「法人税法」までを対象に、
ロースクールの講義で扱われた **71ケース**の解答とリファレンスを再構成したもの。
司法試験で租税法の成績が全国1位だった当時のノートが原本です。

法律のノートが古びるのは、本文よりも先に**条文**の側です。
そこで e-Gov 法令API から引用条文 **111条**の現行全文を取得して各ケースに折りたたみで添付し、
原本の執筆時点（2024年12月9日）から改正された **27条にバッジ**を付けました。
さらに税大講本（令和8年度版）および現行条文と対照して、**校訂注8件**を該当箇所に併記しています
（給与所得控除の最低保障額 55万円→65万円、公益信託法制の整備に伴う所得税法59条・60条の改正など）。

方針として、**本文は原本のまま**にしてあります。加筆も訂正もせず、
ズレは本文を書き換えずに注記として隣に置く。何が当時の理解で何が後の改正かが、読んだまま分かります。
判例・裁判例の内容は更新していません。

---

## Case Law

判例・裁判例を分野や利用場面から引くためのページと、個別論点の検証メモ。

### [破棄判例に学ぶ — 最高裁の考え方](https://claude.ai/code/artifact/68e398d6-b34e-4243-9e9f-2b553114609d)

最高裁の破棄判例を**破棄事由**と**分野**から引く。

### [音楽フェスと著作権](https://claude.ai/code/artifact/364f8be4-c24e-4cd9-95f3-7160b58a454f)

著作権・商標の裁判例を**利用シーン**から引く。

### [商法536条3項の任意規定性 — 判例DB検証](https://claude.ai/code/artifact/cef3ab48-4067-456d-82f1-4471c5afd24c)

匿名組合・審級チェーンの検証メモ。

### [飲食店 店舗賃貸借の判例争点](https://claude.ai/code/artifact/99bde091-38b8-4dd2-b036-842b1bee3310)

借地借家法の争点整理。

### [株式等に対する差押命令申立書の審査](https://claude.ai/code/artifact/20592641-5612-4434-b0b9-b73fdbc09f32)

振替株式・株券不発行株式の執行事務と関連判例（マニュアル補遺）。

関連: [司法試験租税法まとめノート](https://soy-tuber.github.io/taxlaw-notes/)（References）

---

## Chemical Regulation

### [化学物質 法規制 横断検索](https://hanrei2.patentllm.org/chemical/)

特化則・有機則・安衛法・毒劇法・PRTR を1検索で横断。**全77,707物質**、裾切り値と GHS つき。

### [SDS AI判定](https://dsdssearch.patentllm.org/)

SDS を読み込ませて法規制の該当を判定（要ログイン）。

### [触媒カタログ](https://catalyst.patentllm.org/)

遷移金属触媒・有機分子触媒 **179件**を反応から引く。構造式・GHS・実測収率・結晶構造つき。

---

## Daily

### [media.patentllm.org](https://media.patentllm.org/)

公式リリースと変更履歴を毎日追う技術メディア。
llama.cpp / vLLM / Ollama、Anthropic・Gemini の API、MCP、NVIDIA・AMD、SQLite、Rust・Cloudflare、
そして将棋・囲碁・チェスのエンジン。[RSS](https://media.patentllm.org/feed.xml)。

---

## Services

| | |
|---|---|
| [PatentLLM](https://patentllm.org) | 特許検索 |
| [HanreiLLM](https://hanrei2.patentllm.org) | 判例検索 |
| [SubsidyDB](https://subsidy.patentllm.org) | 補助金データベース |
| [HoureiLLM](https://hourei.patentllm.org) | 法令×判例の横断セマンティック検索。日本・米国（州法/連邦 USC・CFR）・各国（要ログイン） |
| [PatentLLM AI](https://ai.patentllm.org) | 特許の検索・分析・ランドスケープ（要ログイン） |
| [HanreiLLM DB](https://dhanrei.patentllm.org) | 判例データベース（Gemini による分析・ファクトチェックつき）（要ログイン） |
| [Nemotron Apps](https://nemotron.patentllm.org) | ローカル LLM（Nemotron 9B）のチャット・履歴ダッシュボード（要ログイン） |

---

[GitHub](https://github.com/soy-tuber) · [Inquiry](mailto:q07025a@gmail.com)

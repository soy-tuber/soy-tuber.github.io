単一のコンシューマ機でどこまで解けるか、決算と現場のどちらが正しいか、専門用語は本当に必要か。
手を動かして確かめたことを置いています。

---

## Technical Reports

### [moca — A Rust Shogi Engine Transcribed from YaneuraOu V9.40](https://soy-tuber.github.io/moca/)

Rust で書いている将棋エンジン。やねうら王 V9.40 探索部の**逐語訳をオラクルとして下層に置き**、
その上にチェスエンジン Reckless のフォークを将棋化した独自探索部を載せる二層構成。

評価部は独立実装4系統で**ビット一致**、NAGISA v3.1 の SPSA 調整値を **137/137** 機械照合つきで再現。
1スレッド NPS 0.72M → **1.02M**、31スレッド実戦 20.3M。
逐語訳との直接対戦は **+223 ± 22**（990局）、水匠11 の C++ 実装 4スレッド相手に **+67 ± 41**（200局）。

方法論としての主張は「**コードを読んで考えるより、一次資料と機械的に突き合わせるほうが速い**」。
目視で作った監査リストは過半が誤読で、実在した欠陥3件は機械照合でしか出なかった
（うち1件は、調整済みパラメータ表が一度も効いていなかったというもの）。

---

囲碁の求解を、単一の RTX 5090（Blackwell sm_120 / Core Ultra 9 285K, WSL2）で再現した2編。
同じマシンで**律速点が逆になる**ため、対になっています。

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
| [HoureiLLM](https://hourei.patentllm.org) | 法令を意味で引く — 全法令24万条をベクトルと全文の両方で |
| [SubsidyDB](https://subsidy.patentllm.org) | 補助金データベース |

---

[GitHub](https://github.com/soy-tuber) · [Inquiry](mailto:q07025a@gmail.com)

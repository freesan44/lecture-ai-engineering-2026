# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
第1回（素のLLM）とエージェントの回答の違い：`app.ask` は、LoRA と QLoRA の基本的な違いに加えて、必要な GPU メモリの目安や学習速度、GPU の具体例まで説明した。ただし、メモリが「約 1/3〜1/4」になるという数値には、測定条件や出典が示されていなかった。一方、`app.agent` は講座の用語集で LoRA と QLoRA をそれぞれ検索し、取得した定義をもとに、QLoRA はベースモデルの重みを量子化したうえで LoRA による学習を行う手法だと簡潔に説明した。また、関連する講義が第2回であることも示した。
- `--trace` で見えたツール呼び出し（ツール名と引数）：
search_glossary({'query': 'LoRA'})
search_glossary({'query': 'QLoRA'})
- 1つの質問での LLM 呼び出し回数（`requests`）：
requests=2
- 追加したツールと、docstring に書いた説明：
make_quiz(lecture: int) -> str：指定した回の用語から1つを選び、確認問題と答えを返す。
docstring には「『第2回の確認問題を出して』のように、復習用の問題を求められたときに使う」と書いた。
## 宿題

- 取り組んだレベル：発展
- やったこと：
演習版の指示とツールを再利用し、エージェントの組み立て部分を Google ADK で書き直した。ADK 版では Agent の instruction に指示を、tools にツールの一覧を渡した。
- 結果：
Google ADK：
[user]: LoRA と QLoRA の違いは？
[glossary_agent]: LoRAとQLoRAの主な違いは、**ベースモデルの重みの保持方法とメモリ節約のアプローチ**にあります。
* **LoRA**: 事前学習済みの重みは固定したまま、重みの更新分を小さな低ランク行列（アダプタ）で表し、その部分だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習パラメータや保存ファイルを大幅に削減できます。
* **QLoRA**: LoRAの仕組みをベースにしつつ、**ベースモデルの重みを4ビットなどに量子化**してメモリを極限まで節約したうえで学習を行う手法です。これにより、より大きなモデルを小さなGPU環境でも扱えるようになります。
関係する回：第2回

app.agent：
uv run python -m app.agent --trace "LoRA とQLoRAの違いは？"
[model] gemini-3.5-flash-lite
講座の用語集によると、LoRAとQLoRAの違いは以下のとおりです。
- **LoRA**
  事前学習済みの重みを固定したまま、重みの更新分を小さな低ランク行列で表し、その部分だけを学習するPEFT手法です。学習パラメータや保存ファイルを少なく抑えられます。
- **QLoRA**
  ベースモデルの重みを4ビットなどに**量子化**してメモリを大幅に節約したうえで、LoRAによる学習を行う手法です。これにより、より大きなモデルを小さなGPUメモリでも扱えるようになります。

つまり、QLoRAはLoRAの仕組みをベースに、ベースモデルを低ビット化してさらに省メモリ化した手法です。

（関連する回：第2回）
--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0009063'), details={'text_prompt_tokens': 1346}, input_text_tokens=1346, input_tokens=1346, output_tokens=201, requests=2, tool_calls=2)
- 気づき・疑問：
ADK 版と演習版の違いは「フレームワークと入口」であって、モデルや部品ではない。
演習版は Agent オブジェクトをコードで組み立て、課程自作の CLI で --trace を
見ながら動かす。ADK 版はフォルダ（__init__.py + root_agent）を規約として渡し、
adk run で起動する。ループはどちらもフレームワーク内部にあるが、演習版は
過程を観測でき、ADK 版は最終出力しか見えない。回答の文言が違うのは、
枠組みが違うことによる入力の差とサンプリングの揺れであって、性能差ではない。

# コーディングフェーズのtips

- [コーディングフェーズのtips](#コーディングフェーズのtips)
  - [コードを小さく保つため、適切に抽象化する](#コードを小さく保つため適切に抽象化する)
    - [サイクロマティック複雑度\[2\]](#サイクロマティック複雑度2)
    - [80/24ルール\[2\]](#8024ルール2)
    - [どうやって適切に抽象化するのか](#どうやって適切に抽象化するのか)
    - [抽象化前のコード例(サイクロマティック複雑度11)](#抽象化前のコード例サイクロマティック複雑度11)
    - [抽象化後のコード例(サイクロマティック複雑度1)](#抽象化後のコード例サイクロマティック複雑度1)
  - [良い変更説明を書く](#良い変更説明を書く)
    - [コミットメッセージを適切に書く](#コミットメッセージを適切に書く)
      - [適切なコミットメッセージを書くことの利点](#適切なコミットメッセージを書くことの利点)
      - [Gitのコミットメッセージの標準: 50/72ルール\[2\]\[3\]](#gitのコミットメッセージの標準-5072ルール23)
      - [あまり良くないコミットメッセージ例](#あまり良くないコミットメッセージ例)
      - [良いコミットメッセージ例](#良いコミットメッセージ例)
    - [コードにコメントを書く](#コードにコメントを書く)
    - [コードコメント or コミットメッセージ or PR説明？](#コードコメント-or-コミットメッセージ-or-pr説明)
      - [コードに現れない情報を書くことが出来る場所\[14\]](#コードに現れない情報を書くことが出来る場所14)
      - [コードコメントとコミットメッセージの使い分け](#コードコメントとコミットメッセージの使い分け)
      - [PRの説明欄やPRコメントに書くもの](#prの説明欄やprコメントに書くもの)
  - [コミット単位に注意を払う](#コミット単位に注意を払う)
  - [コミットメッセージにprefixを付ける\[7\]](#コミットメッセージにprefixを付ける7)
  - [merge先のブランチの変更を反映するのに、mergeではなくrebaseを検討する\[4\]\[13\]](#merge先のブランチの変更を反映するのにmergeではなくrebaseを検討する413)
  - [ドラフトPRを上手く使う](#ドラフトprを上手く使う)
  - [コマンドクエリ分離（CQS）\[2\]](#コマンドクエリ分離cqs2)
  - [コードは債務である認識を忘れない\[1\]](#コードは債務である認識を忘れない1)


## コードを小さく保つため、適切に抽象化する

コードが複雑になりすぎていないかは常に注意を払っておくべき。
- 一つの関数で多くのことをし過ぎると巨大な関数になり可読性やテスタビリティの低下が起こる
- そのため適切に抽象化し関心事ごとに別のクラスや関数に委譲できると良い

### サイクロマティック複雑度[2]
- コードの複雑度のメトリクス。コードの経路を数えるもの
- 1から始めてifやforなど分岐とループの命令が何回登場するかを数える
- キーワードが登場するごとに、（1から始まる）数字をインクリメントする
- 7を超える場合はその関数は複雑過ぎるためリファクタリングしたほうが良い
    - 人間の短期記憶は4〜7個の情報しか保持できない[10][11]

<br>
下で例を示す

### 80/24ルール[2]
- 横幅の最大文字数/メソッドの最大行数の目安を持つと良い
- 横幅：80文字
- メソッドの最大行数：24行
- 80/24という数字自体が重要なわけではない。要は、横幅を長くしすぎないことで、画面分割してunit testとテスト対象のコードを両方見ることも出来るし、side by sideビューでdiffを見ることも出来る。また縦にも長くしすぎないことで、スクロールせずにメソッドの全容が見える

メソッドの最大行数については、24は実際難しいと思うが要は関数が長くなりすぎて見通しが悪い状態（何やってるのか把握しづらいに）になっていないのかを意識出来ると良い。

### どうやって適切に抽象化するのか
- 物事の本質を抽出する: ひとまとまりの処理を1つのメソッドに抽出する
    - 実装の詳細を見ずとも、処理の流れのなかでその1つのメソッドの意図が分かるようになっていれば、上手く抽象化出来ている
- 1つの関数内で何でもやろうとせず、責務ごとにクラスを作り、処理を委譲する

### 抽象化前のコード例(サイクロマティック複雑度11)

```python
def calculate_discounted_price(total_price, user_type, is_holiday, coupon_code):
    discount = 0

    if user_type == "premium":
        if total_price >= 100:
            discount = 0.20
        elif total_price >= 50:
            discount = 0.15
        else:
            discount = 0.10
    elif user_type == "regular":
        if total_price >= 100:
            discount = 0.10
        elif total_price >= 50:
            discount = 0.05
        else:
            discount = 0

    if is_holiday:
        discount += 0.05

    if coupon_code == "SAVE10":
        if discount < 0.10:
            discount = 0.10
    elif coupon_code == "SAVE20" and total_price >= 200:
        discount = max(discount, 0.20)

    return total_price * (1 - discount)

# 使用例
price = calculate_discounted_price(120, "premium", True, "SAVE20")
print(f"割引後の価格: {price}")
```

### 抽象化後のコード例(サイクロマティック複雑度1)

```python
def _get_base_discount(total_price, user_type):
    discounts = {
        "premium": [(100, 0.20), (50, 0.15), (0, 0.10)],
        "regular": [(100, 0.10), (50, 0.05), (0, 0)]
    }
    for threshold, discount in discounts[user_type]:
        if total_price >= threshold:
            return discount

def _apply_holiday_discount(discount, is_holiday):
    return discount + 0.05 if is_holiday else discount

def _apply_coupon(discount, total_price, coupon_code):
    if coupon_code == "SAVE10":
        return max(discount, 0.10)
    elif coupon_code == "SAVE20" and total_price >= 200:
        return max(discount, 0.20)
    return discount

def calculate_discounted_price(total_price, user_type, is_holiday, coupon_code):
    discount = _get_base_discount(total_price, user_type)
    discount = _apply_holiday_discount(discount, is_holiday)
    discount = _apply_coupon(discount, total_price, coupon_code)
    return total_price * (1 - discount)

# 使用例
price = calculate_discounted_price(120, "premium", True, "SAVE20")
print(f"割引後の価格: {price}")
```

## 良い変更説明を書く

### コミットメッセージを適切に書く

- コミットメッセージはコミュニケーションであり、チームメンバーや将来の自分のためにもきちんと書くと良い
- その差分に対して、変更意図などコード差分に現れない自分の思考を伝える

#### 適切なコミットメッセージを書くことの利点

| 利点 | 詳細 |
|-----|-----|
| コードレビューの効率化 | ・レビュアーがコミット単位で差分とコード作者の意思決定(そのように修正した理由など)をセットで見られることでレビューがしやすく質も向上する[9]<br>・コミットメッセージを適切に書こうとすること自体が、小さく焦点を絞ったコミットを促す<br>・レビュイー/レビュアー間のコミュニケーションコストが減少する  |
| 保守性の向上 | ・コードの変更理由が明確になり、将来の改修時の判断材料となる<br>・「なぜこの方法で実装したのか」という疑問への回答になる |
| ドキュメンテーションの補完 | ・細かな実装の詳細や決定事項を記録する場所となる<br>・コード内のコメントを減らし、コードの可読性を向上させる |
| コードの品質向上 |・コミットメッセージを書く過程で、自身の変更を再考し、改善する機会となる |
| PRのdescriptionを書くのが捗る |・コミット単位やコミットメッセージが適切に作ってあると自分でも変更点がわかりやすいのでdescriptionを書きやすい（変更理由などはコミットメッセージに残っているため自分で思い出す必要も無い） |


#### Gitのコミットメッセージの標準: 50/72ルール[2][3]

- 50/72ルールとは
    - サマリーを現在形で50文字以内で書く(日本語なら25文字くらい)
    - それ以上のテキストを書く場合は、2行目を空行にする
    - 好きなだけテキストを追加してよいが、幅が72文字以内になるようにフォーマットする
- 50/72ルールの利点
    - git log --onelineやGitHub, Bitbucketなど各ツールでコミットの一覧を見るときに見やすい
    - 現在形で書くことで行が短くなりやすい（見やすくなる）
- コミットメッセージの内容
    - コードが自明なときはサマリーだけで良い
    - 少しでも疑問が浮かびそうなものは、コンテキストを追加する
    - コミットの差分には何をどのように変更したのかの情報がすでに含まれているので、コミットメッセージではそれよりもなぜその変更をしたのか、なぜその形になっているのかを説明するのに最適

#### あまり良くないコミットメッセージ例

```text
ログイン周りの修正
ログイン試行回数制限の実装。ハッカー対策のため。あとパスワードリセット機能のバグ修正もした。関連するテストも更新。
```

- サマリーが曖昧。コミットメッセージだけでは具体的な変更内容がわからないので、主要な変更を明確に示すべき
- 2行目が空行になっていないのでサマリーと詳細の区別がつきづらい
- 横に長過ぎるので可読性が悪い
- 複数の独立した変更が一つのコミットに含まれている
- 説明不足な部分がある
  - 「ハッカー対策のため」という説明は簡素すぎる。どのようなセキュリティ向上が期待できるのかより具体的に書くべき

<br><br>
その他、よくある改善の余地のある例

- 「レビュー指摘対応」
    - コメントがいくつも有る場合などは、どのコメントに対するものかわからないのでレビュアーは何の指摘についてのものか考えたりしないといけない場合がある。また、あとから見ると何の修正かわからない
- 「バグ修正」
    - *これだけではレビュアーや将来のコード考古学者にとっては助けにならない*[1]
    - 何のバグを修正したのかはコミットの中身を見るまでわからない

#### 良いコミットメッセージ例

```text
ログイン試行回数の制限を実装

一定時間内のユーザーごとのログイン試行回数を
制限する機能を追加。これにより、ブルートフォース
攻撃を防ぎ、システム全体のセキュリティを向上させる。
```

- 適切なコミット粒度
  - 一つのことしかしていないので変更の目的と差分が完全に対応する
  - 一つのコミットで複数の異なる変更を行っていないので変更を容易に戻したり別のブランチに適用したりできる
- 変更理由を説明している
  - 将来のメンテナンス時に役立つ(また当該部分を変更したいと思った場合に、そうして良いのかの判断に使える)
- 50/72ルールに沿っており様々なGitツールでの可読性が高い


### コードにコメントを書く

- 自分の変更差分やその理由をレビュアーが理解出来ないなら、自分のやったことが正しいとしても、コードの構造かコメント（またはその両方）の改善が必要[1]
- コミットメッセージより目につきやすいため、重要度の高い補足情報や、挙動の理解に役立つ補足などを記載する
- docstringやJavaDocなどは、最低限Public関数やクラスには完備する

### コードコメント or コミットメッセージ or PR説明？

#### コードに現れない情報を書くことが出来る場所[14]

コードコメント or コミットメッセージ or PR(説明欄、コメント)

#### コードコメントとコミットメッセージの使い分け

どちらもコードベースに残る情報だが、コードを眺める時、コードコメントは目につきやすく、コミットメッセージは(基本)自分で見に行くものという違いがある。

そのためすぐに目についたほうが良いような情報はコードコメントに書いた方が良い。

- 当該部分の改修時などに知っておくべき注意ポイント
    - 直感とは反するがあえてこのようにしている部分の説明など
- 複雑な部分の認知負荷を下げる説明
    - パッと見何しているかすぐにわかりづらいような場所に自然言語で説明を付ける

コミットメッセージに書くもの（主にレビュー時に活躍する）

- その差分に対する変更意図
- コードコメントに書くと冗長になるが、思考として残しておきたい情報
    - この差分の目的の実現方法として他にこういう選択肢があったが、今回の差分ではこの方法を選んだ理由、とか
        - (目につきやすいコードコメントに書いておいた方が良い内容かはその時々による。可読性と内容の重要性のトレードオフ)

#### PRの説明欄やPRコメントに書くもの

レビュー時に必要であろう情報を記載する

- PRの目的、変更概要
- 悩んでいる部分や特に重点的にレビューしてもらいたい箇所(あれば)
- その機能の要件が分かるドキュメントやチケットリンク
- PRの目的とは違うものが少し含まれている場合にそれを補足(ついでに少し既存コメントを直したものがあるとか)

<br>

コード自体の補足説明など、将来コードを眺める際にも見えた方が良い情報は、PRのコメントに自分で書いて説明するくらいなら、コードベースに残る情報としてコードコメントに書く（かコミットメッセージに含めておく）のがベター

## コミット単位に注意を払う

コミット単位は意味のあるまとまりになるようにする。具体的には、一つの意思決定につき一つのコミットという単位になっていると良い[9]

- 例えば、異なる機能の追加は一つのコミットに混ぜない（混ぜると差分のどの部分がどの目的のものかわかりづらくなる）。また、コミットメッセージに合致しない差分を混ぜない
  - ❌「Aを出来るようにする（実はBも出来るようにしている）」
  - ⭕「Aを出来るようにする」「Bを出来るようにする」
- 「一つの意思決定」の粒度：PRの目的を達成するための構成要素一つ一つのイメージ
- コミット単位が適切であればコミットメッセージも書きやすくprefix(後述)も上手く付けられるはず

<br>
コミット単位を一つの意思決定ごとにするメリット↓

- 適切なコミット単位に適切なコミットメッセージがあると、差分とその意図をセットで見ることが出来る（コミット単位でのレビューが可能になる）のでレビューがとてもしやすい
- 一つの意思決定ごとのコミットになっていると、やっぱりやめたいのでresetして戻す、なども容易

<br>

コミットは必要に応じてinteractive rebaseやamend commitを活用してまとめながら開発出来ると良い。
- 例えば機能Aの追加、その修正、そのまた修正、という3つのコミットがあるのであればこれは1つにまとまっていた方がツリーも綺麗でレビューも断然しやすい

参考：[Appendix_コミットまわりのテクニック](./Appendix_コミットまわりのテクニック.md)

## コミットメッセージにprefixを付ける[7]

- コミットメッセージにprefixを付けると、どんな変更なのか瞬時に分かるのでレビューしやすくなる
    - 例：「feat: ◯◯を出来るようにする」
    - 例えばコミットの一覧を眺めたときに今見たいのはどれかなどがすぐ分かりやすい
- 開発者がコミットの粒度を意識するようになる
- （prefixを正しく付けることは本質ではなく、commitの単位を意識したり見やすくする意識付けに効果がある（なので分類で悩みすぎない））

Angularプロジェクトで使用されているprefix(https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#type)
```
feat: A new feature
fix: A bug fix
docs: Documentation only changes
style: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
refactor: A code change that neither fixes a bug nor adds a feature
perf: A code change that improves performance
test: Adding missing or correcting existing tests
chore: Changes to the build process or auxiliary tools and libraries such as documentation generation
```

## merge先のブランチの変更を反映するのに、mergeではなくrebaseを検討する[4][13]

- rebaseの場合は不要なマージコミットが作成されない
- featureブランチへのマージコミットはノイズになり本来の修正の履歴が分かりづらくなってしまう
    - bitbucketなどのGUIではさほど見づらくないが、git logで見るとマージコミットの中身のそれぞれのコミットのログも表示されてしまうのでかなり見づらくなる
    - rebaseしておいたほうがコミットツリーが直線的になりわかりやすい

<br>

注意
- rebaseは履歴を書き換えるため、他の開発者と共有しているブランチではmergeを使うか合意を取る
- PR提出後はmergeのほうが好ましい
    - PR提出後は、他の開発者がレビューやテストを行っている可能性があるので、履歴を変更しないほうが安全である
    - rebaseするとコミットハッシュが変わるので、コミットリンクが古いものになる
    - 要は他の人がpullしている可能性などがある場合はrebaseはやめておいたほうが良い

参考：
- [Merging vs rebasing - Atlassian](https://www.atlassian.com/ja/git/tutorials/merging-vs-rebasing)
- [Appendix_コミットまわりのテクニック](./Appendix_コミットまわりのテクニック.md)

## ドラフトPRを上手く使う

（再掲）プロセスの比較的早期に欠陥を発見すると、(当然ながら)後々修正に必要となる時間が短くなる[1][12]
<br>

- 大きな手戻りを防ぐため、途中でもチームメンバーと相談したいことがあればドラフトPRを出し相談する
- BitbucketなどはドラフトPR機能は無いが、レビュアーの設定を無しにすれば似たような使い方は出来る
- もしくは、ドラフトPRとしてわざわざ出さなくても、単にブランチ画面から差分を見ながら話すとかでも⭕

## コマンドクエリ分離（CQS）[2]

- コマンド：副作用のあるメソッド
	- 副作用：ある手続きが何かの状態を変えること
	- 副作用のあるメソッドはデータを返さないように実装すべきで、つまり戻り値の型はvoidであるべき
        - データを返さないメソッドがあったら、副作用があるメソッドであることがわかる
- クエリ：副作用が無くデータを返すメソッド
	- CQSに従っている場合、あるメソッドに戻り値の型があれば、副作用が無いと理解出来る
- コマンドとクエリを分離したほうが、APIの意図を理解しやすくなる。そして、実装コードを読まなくても、2種類の機能を区別できるようになる
- コマンドよりもクエリのほうが理解しやすいので、コマンドよりもクエリを優先する

## コードは債務である認識を忘れない[1]

- *コードは必要な債務かもしれないが、それ自体では、どこかの誰かにとっての将来の単なる保守タスクである。飛行機が積載する燃料によく似て重量があり、しかし当然ながらその飛行機が飛ぶのに必要だ[1]*
    - そもそも新機能について開発に足る正当な理由があるかや、コードの重複、車輪の再発明等に注意する
- コード行数が多ければ多いほど、コードベースは悪化する[2]
	- コードを増やせば増やすほど、人が読んで理解しなければいけない量が増える
- 一から新しい処理を作ろうとする場合は、既存のコードやライブラリで出来ないのか一度考えてからにする
  - もちろん既存コードを無理に使ってわかりづらくなるような場合等そうすべきでない

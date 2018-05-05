---?image=assets/top.png&size=cover

# 完全自動
# 機械学習

---?image=assets/pika.png&size=cover

# 自己紹介

- coord.e こーでぃ
- Python, C++, 低レイヤ, フロントエンド <!-- .element: class="fragment" -->
- 自動化狂 <!-- .element: class="fragment" -->

- @fa[twitter] @coord\_e
- @fa[github] github.com/coord-e
- qiita.com/coord-e

---

# 問題

機械学習は...
- 何度もパラメーターを調整する必要がある
- 1回1回時間がかかる    
  → 学習回している間作業できない...
- サーバーを借りると進捗チェックするのが面倒

---

# 解決策

自動化

(機械学習の話じゃないですごめんなさい)

---

# 解決策: Tag開始

- `git tag`されたときに学習を開始すれば、パラメーターがごちゃごちゃにならない
- Travis CI

---

# 解決策: 自動終了

学習終了時に, S3にデータを移して、自動で終了すればお金に無駄がない

```bash
python3 train.py --monitor monitor --save model -se 500 --monitor-video 50000 --timesteps 10000000 --tensorboard ./tblog --discord \
  && tar Jcf $ARCHIVE_NAME monitor model tblog \
  && aws s3 cp $ARCHIVE_NAME s3://$BUCKET_NAME/ \
  && sudo shutdown -P now
```

---

# 解決策: Bot

進捗はDiscord Botに教えてもらおう

```python
@self.client.event
async def on_message(message):
  async def send_state():
    state = subprocess.check_output(['tail', '-1', str(self.monitor_dir.joinpath('log.csv'))]).decode().rstrip()
    await self.client.send_message(message.channel, state)
```

---

## これらをEC2に乗っければOK

Terraform!

---

# 構成

- Tag時に開始: Travis CI
- 進捗報告: Discord Bot
- 学習用サーバー: Amazon EC2
- インフラ: Terraform

---

# できた

---?image=assets/arch.png&size=contain

---?image=assets/starttrain.png&size=contain

---?image=assets/discord.png&size=contain

---?image=assets/travis.png&size=contain

---?image=assets/s3.png&size=contain

---

# ここがすごい!

- Tag付により、どの条件でやったかがコードを見れば一目瞭然
- 進捗が見れるのはわかりやすい
- 自動保存が、いちいち落とさなくて良くて地味にありがたい
- Travisのログがかっこいい

---

# ここがダメ! (事実)

- TagのせいでGitHubのreleasesが増えまくる
- Tagでしか指定できないのは面倒
- hashが指定できれば別にtagじゃなくて良いのでは
- 異常終了とかで`shutdown`が効かず多額の請求が発生(実話)

---

# ここがダメ! (気持ち)

- EC2はなんかダサい...
- Terraformってそういうもんじゃないだろ...
- 環境構築シェルスクリプトかよ...

---

# 別の方法を

- Tagやめろ
- 終わったら確実に止まる
- 抽象化(?)された環境構築

---

## Machine Learning as a Service

Amazon MLとか

### Pros:
- 楽(偏見)
- 安い(偏見)

### Cons:
- 既知の手法しか使えない (自由度が低い)

---

## Function as a Service

AWS Lambdaとか

---?image=assets/google.png&size=contain

---

# AWS Batch

- キューがあって、そこにJobを追加していける
- リソースが柔軟(4~255コアで適当にとか)
- 環境はDocker
- 長時間も可能
- 終わったら止まる
- ログもCloudWatchで見れる

---

# これだ

---

## Dockerfileを書いて...

(どっかーん) <!-- .element: class="small" -->

コミットIDやブランチを環境変数から指定できるようにする

```bash
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
git fetch --all
[ -v BRANCH_NAME ] && git checkout $BRANCH_NAME
git pull
[ -v COMMIT_ID ] && git checkout $COMMIT_ID
```

---

## Docker HubのAutomated Buildに投げます

くえうえど。

![hub](assets/hub.png)

---

# そしてキューにJobを投げる

ポチポチでも、シェルでもいけます

```bash
aws batch submit-job \
  --job-name my-super-special-job \
  --job-queue ${JOB_QUEUE_NAME} \
  --job-definition ${JOB_DEFINITION_ARN}
```

---

# できた

---?image=assets/batcharch.png&size=contain

---?image=assets/batchjob.png&size=contain

---?image=assets/batchlog.png&size=cover

---

# まとめ

- 自動化は楽しいが、自動化するのに時間をかけすぎると悲しい
- AWS Batchはいいサービス

---

# 実際に運用してます

強化学習でロボット歩かせるやつです

- @fa[github] Y-modify/deepl2
- @fa[github] Y-modify/deepl2-infra

---

# ありがとうございました

by @coord\_e

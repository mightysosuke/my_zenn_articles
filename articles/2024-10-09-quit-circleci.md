---
title: "CircleCIでのCI/CDをやめるには"
emoji: "🚫"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CircleCI"]
published: true
published_at: 2024-10-09 09:00
publication_name: urstudx_blog
---

## CircleCIのファイルを無くせばCI/CDが動かなくなると思っていた
久々に技術記事を書いています。
CircleCIといえば有名なCI/CDツールですが、最近はGitHub Actionsのを採用する企業も増えているので私もGitHub Actionsへ移行をしようと考えました。
その時に、CircleCI経由のCI/CDが続いており頭にハテナが浮かんだので備忘録として残します。

そもそも、私はCircleCIのファイルをリネームしたり削除したりすることでCI/CDが止まると思っていました。
しかし、それは大きな間違いで以下のようなエラーメッセージが出現しました。

```
No .circleci/config.yml was found in your project.
Add a configuration file to your repository or refer to documentation.
```

なぜだ？？？ファイルは消したのに。。。

## CircleCIによるCI/CDを止めるには
CircleCIのサービス内で設定する必要があったので、スクショ付きで残します。

1. プロジェクト一覧のうち、CI/CDを止めたいプロジェクトの設定(Project Settings)を開く
![](https://storage.googleapis.com/zenn-user-upload/c866d242a898-20241008.png)

2. 下のほうにあるStop Buildingをクリックする
![](https://storage.googleapis.com/zenn-user-upload/4be48c22434d-20241008.png)

3. 確認ダイアログが出てくるので、ダイアログ内のStop Buildingをクリックする

この手順でCircleCI経由でのCI/CDを止めることができました！
直感でやるのはダメ、ゼッタイ

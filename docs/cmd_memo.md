# 環境構築

1. Node.jsをインストール  
2. Hexo をインストール  
```
npm install hexo-cli -g
```

# 【Github管理方法】
ブランチ
・master     ：ブログ生成用のコードを保存
・gh-pages   ：Web公開用のコードを保存

※master フォルダから hexo deploy を行うと、gh-pagesにデプロイされる。
　（_config.yml でデプロイ先のブランチを「gh-pages」とした。）

#### #### #### #### #### #### #### #### #### #### #### #### #### #### ####
# ブログ更新
#### #### #### #### #### #### #### #### #### #### #### #### #### #### ####

cd C:\Repos\computing-atman

# 新規ページ作成
hexo new post “XXX”

# 静的サイトを生成
hexo generate

# サーバー起動
hexo server -g

# ローカルサーバー画面を確認
http://localhost:4000

# デプロイ
hexo deploy -g

# 【注意】Githubカスタムドメインを利用している場合、CNAMEファイルが消えるため注意
# publicフォルダにCNAMEファイルを保存すること！

# 公開サーバー画面を確認
https://atman-33.github.io/template-blog-hexo/


#### #### #### #### #### #### #### #### #### #### #### #### #### #### ####
# ブログソース情報をGithubにバックアップ
#### #### #### #### #### #### #### #### #### #### #### #### #### #### ####

# 更新状況を確認
git status

# ステージエリアにファイルを追加
git add -A

# コミット
git commit -m "XXX"

# originを確認
git remote -v

# origin（master）にプッシュ
git push origin master

# 補足:リモートリポジトリのデータをプル
git pull origin master

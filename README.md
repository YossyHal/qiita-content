```sh
# Qiita CLI をアップデート
npm install @qiita/qiita-cli@latest

# 記事の作成
npx qiita new 記事のファイルのベース名

# 記事の投稿・更新
npx qiita publish 記事のファイルのベース名

# 全ての記事を反映
npx qiita publish --all

# Qiita 上で更新を行い、手元で変更を行っていない記事ファイルを同期
npx qiita pull

# 強制的に Qiita 上の内容を記事ファイルに反映させる
npx qiita pull --force
```

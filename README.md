# yusuke_shigeta

- [yusuke\_shigeta](#yusuke_shigeta)
  - [環境構築](#環境構築)

## 環境構築

1. 開発環境の全体

Dockerコンテナの中に「PHP/Laravel」「データベース」「Redis」などを閉じ込める。
Next.jsは、まずは手軽にローカル（自分のPC）で動かして、Laravelコンテナと通信させるのが一般的でスムーズ。

2. Laravel Sailで環境を構築する（所要時間：3分）

```bash
# プロジェクト作成（DBはMySQLを指定）
curl -s "https://laravel.build/backend?with=mysql,redis" | bash
```

3. 起動

```bash
cd backend
./vendor/bin/sail up -d
```

これで、http://localhost にアクセスするとLaravelの画面が出る。PCにはDocker以外何もインストールされていない。

💡 ヒント: 毎回 ./vendor/bin/sail と打つのは面倒なので、alias sail="./vendor/bin/sail" と設定しておくと、次から sail up だけで起動できるようになる。

テーブルがないよってエラーが出たら以下を叩いてみる

./vendor/bin/sail artisan migrate

4. Next.jsを準備する

Laravelコンテナが動いている状態で、別のターミナルを開き、Laravelの「外側」にNext.jsを作る。

```bash
npx create-next-app@latest frontend
# 設定はすべてYes（TypeScript, App Router, Tailwind CSS）
```

5. コンテナと通信するための「魔法の設定」

ここが一番のポイント。
Docker（Laravel）とローカル（Next.js）は「住んでいる世界」が違うので、
Next.jsからLaravelにアクセスする際は、Laravel側で「CORS（通信許可）」の設定が必要。

① Laravel側の設定 (backend/.env)
Next.js（ポート3000）からの通信を許可。

```
APP_URL=http://localhost
FRONTEND_URL=http://localhost:3000
```

② APIの疎通確認
Laravel側でAPIを作る。

```bash
# コンテナの中でコマンドを実行
docker exec -it backend-laravel.test-1 bash
php artisan install:api
```

routes/api.php にテスト用コードを書く。

```php
// backend/routes/api.php
Route::get('/test', function () {
    return ['status' => 'success', 'message' => 'DockerのLaravelと繋がったよ！'];
});
```

6. Next.jsからデータを取ってくる

frontend/src/app/page.tsx を以下のように書き換えて、ブラウザで http://localhost:3000 を確認。

```tsx
export default async function Home() {
  // Docker内のLaravel APIを叩く
  const res = await fetch("http://localhost/api/test", { cache: "no-store" });
  const data = await res.json();

  return (
    <main className="p-24">
      <h1 className="text-2xl font-bold">Next.js + Docker Laravel</h1>
      <p className="mt-4 p-4 bg-gray-100 rounded">{data.message}</p>
    </main>
  );
}
```

`npm run dev`で起動

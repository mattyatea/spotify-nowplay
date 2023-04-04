# Spotify Nowplay for Misskey

## これは何？

Spotify Nowplay for Misskey は Spotifyで再生中の曲をMisskeyに投稿するためのツールになる予定のものです。
まだ投稿機能はできてません()

## 使い方

### 1. Spotifyのアプリを作成する

[Spotify for Developers](https://developer.spotify.com/dashboard/) にアクセスしてログインします。

ログイン後、`CREATE AN APP`をクリックしてアプリを作成します。

Client IDとClient Secretをメモしておきます。

作成後、`EDIT SETTINGS`をクリックして`Redirect URIs`に`http://localhost:5173/callback` を追加します。

### 2. .envに設定を書く

.env.exampleをコピーして.envを作成します。

```
VITE_URL=https://
VITE_PORT=8080
VITE_SPOTIFY_CLIENT_ID=
VITE_SPOTIFY_CLIENT_SECRET=
```

これの VITE_SPOTIFY_CLIENT_ID と VITE_SPOTIFY_CLIENT_SECRET に先ほどメモしたものを入力します。

これで準備は完了です。

### 3. buildする

`vite build`を実行します。
そうすると build ディレクトリができるので `node build` すればたぶん動きます。

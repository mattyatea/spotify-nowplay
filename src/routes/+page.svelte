{#if User.MisskeyLogin}
    {MiName}@{MiHost}
{:else}
    Misskey No Login
{/if}
<br>
{#if User.SpotifyLogin}
    {#if NowPlaying}
        <img alt="Song Album Jacket" width="30%" src="{NowPlay.item.album.images[0].url}"><br>
        {SongName} - {SongArtist}
    {:else}
        Spotify Yes Login
    {/if}
{:else}
    Spotify No Login
{/if}

<script lang="ts">
    import {onMount} from 'svelte';

    let User = {SpotifyLogin: false, MisskeyLogin: false, MastodonLogin: false};
    let NowPlaying, MiName, MiId, MiInstance, MiToken, MiHost, SpoToken, SpoRefresh, NowPlay, SongName, SongArtist; // くそ長定義

    async function GetNowPlay(SpoToken) {
        const result = await fetch("https://api.spotify.com/v1/me/player/currently-playing", {
            method: "GET",
            headers: {"Authorization": "  Bearer " + SpoToken}
        });
        return await result.json();
    }

    onMount(async () => {
        //いろいろの定義
        SpoToken = localStorage.getItem('SpoToken') ? localStorage.getItem('SpoToken') : null;
        SpoRefresh = localStorage.getItem('SpoRefresh') ? localStorage.getItem('SpoRefresh') : null;
        MiToken = localStorage.getItem('MiToken') ? localStorage.getItem('MiToken') : null;
        MiInstance = localStorage.getItem('MiInstance') ? localStorage.getItem('MiInstance') : null;
        MiHost = MiInstance ? MiInstance.replace(/^https?:\/\//, '').replace(/\/$/, '') : null;

        if (SpoToken) { // spotifyログイン済みの場合に実行
            User.SpotifyLogin = true; // spotifyログイン済みのフラグを立てる
            NowPlay = await GetNowPlay(SpoToken); // 現在再生中の曲を取得
            if (NowPlay.is_playing == true) {
                NowPlaying = true; // 再生中の曲がある場合のフラグ立て
                SongName = NowPlay.item.name; // 曲名
                SongArtist = NowPlay.item.artists[0].name; // アーティスト名
            } else {
                NowPlaying = false; //今再生中の曲がない場合のフラグ立て
            }
        }
        if (MiToken && MiInstance) { // misskeyログイン済みの場合に実行
            User.MisskeyLogin = true; // misskeyログイン済みのフラグを立てる
            fetch(MiInstance + "api/i", { // Miの自分のユーザーの情報取得
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify(({"i": MiToken}))
            })
                .then(res => res.json())
                .then(res => {
                    MiName = res.name; // Miのユーザー名
                    MiId = res.id; // MiのユーザーID
                });

        }
    })
</script>
<div style="padding-left: 16px">
    {#if User.MisskeyLogin}
        {MiName}@{MiHost}
    {:else}
        Misskey No Login
    {/if}
    <br>
    {#if User.SpotifyLogin}
        {#if NowPlaying}
            <img alt="Song Album Jacket" width="30%" src="{NowPlay.item.album.images[0].url}"><br>
            {SongName} - {NowPlay.item.artists[0].name}<br>
            {#if User.MisskeyLogin}
                <IconButton class="material-icons" on:click={() => SendMisskey()}>
                    send
                </IconButton>
            {/if}
        {:else}
            現在再生中の曲はありません。
        {/if}
    {:else}
        Spotify No Login
    {/if}
    <h2>Config</h2>
    <FormField>
        <Switch bind:checked={AutoNote}/>
        <span slot="label">
            AutoNote
  </span>
    </FormField>
</div>

<script lang="ts">
    import IconButton from '@smui/icon-button';
    import {onMount} from 'svelte';
    import FormField from '@smui/form-field';
    import Switch from '@smui/switch';

    const wait = async (ms) => new Promise(resolve => setTimeout(resolve, ms));
    let User = {SpotifyLogin: false, MisskeyLogin: false, MastodonLogin: false};
    let AutoNote = false;
    let MyMi, NowPlaying, MiName, MiId, MiInstance, MiToken, MiHost, SpoToken, SpoRefresh, NowPlay, SongName,
        SendText, AutoNoteChk,
        SongArtist; // くそ長定義
    $: AutoNoteChk = AutoNote ? 'true' : 'false'


    async function GetNowPlay(SpoToken) {
        const result = await window.fetch("https://api.spotify.com/v1/me/player/currently-playing", {
            method: "GET",
            headers: {"Authorization": "  Bearer " + SpoToken}
        });
        if (result.status === 204) {
            return null;
        } else {
            return await result.json();
        }

    }

    async function SendMisskey(SendText) {
        await window.fetch(MiInstance + "api/drive/files/upload-from-url", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify(({
                "i": MiToken,
                "url": NowPlay.item.album.images[0].url,
            }))
        })
        await wait(2500);
        const FileIDExt = await window.fetch(MiInstance + "api/drive/files/find", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify(({
                "i": MiToken,
                "name": NowPlay.item.album.images[0].url.slice(-40) + ".jpg",
            }))
        })
        const FileID = await FileIDExt.json();
        const result = await window.fetch(MiInstance + "api/notes/create", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify(({
                "i": MiToken,
                "text": SendText,
                "visibility": "home",
                "mediaIds": [FileID[0].id],
            }))
        })
        return await result.json();
    }


    onMount(async () => {
        //いろいろの定義

        SpoToken = localStorage.getItem('SpoToken') ? localStorage.getItem('SpoToken') : null;
        SpoRefresh = localStorage.getItem('SpoRefresh') ? localStorage.getItem('SpoRefresh') : null;
        MiToken = localStorage.getItem('MiToken') ? localStorage.getItem('MiToken') : null;
        MiInstance = localStorage.getItem('MiInstance') ? localStorage.getItem('MiInstance') : null;
        MiHost = MiInstance ? MiInstance.replace(/^https?:\/\//, '').replace(/\/$/, '') : null;
        AutoNote = localStorage.getItem('AutoNote') === 'true' ? true : false;
        let prevSongName = '';
        let prevSongArtist = '';
        setInterval(async () => {
            localStorage.setItem('AutoNote', AutoNoteChk)

        }, 0);
        if (SpoToken) { // spotifyログイン済みの場合に実行
            User.SpotifyLogin = true; // spotifyログイン済みのフラグを立てる
            NowPlay = await GetNowPlay(SpoToken); // 現在再生中の曲を取得
            if (NowPlay) {
                NowPlaying = true; // 再生中の曲がある場合のフラグ立て
                SongName = NowPlay.item.name; // 曲名
                SongArtist = NowPlay.item.artists[0].name; // アーティスト名

            } else {
                NowPlaying = false; //今再生中の曲がない場合のフラグ立て
            }
            setInterval(async () => {
                NowPlay = await GetNowPlay(SpoToken); // 現在再生中の曲を取得
                if (NowPlay) {
                    NowPlaying = true; // 再生中の曲がある場合のフラグ立て
                    SongName = NowPlay.item.name; // 曲名
                    SongArtist = NowPlay.item.artists[0].name; // アーティスト名
                    if (SongName !== prevSongName) {
                        let SendText = "NowPlaying \n" + SongName + " - " + SongArtist
                        if (MiInstance && AutoNoteChk) await SendMisskey(SendText);


                        prevSongName = SongName;
                        prevSongArtist = SongArtist;
                    }

                } else {
                    NowPlaying = false; //今再生中の曲がない場合のフラグ立て
                }
            }, 10000);
        }
        if (MiToken && MiInstance) { // misskeyログイン済みの場合に実行
            User.MisskeyLogin = true; // misskeyログイン済みのフラグを立てる

            await window.fetch(MiInstance + "api/i", { // Miの自分のユーザーの情報取得
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify(({"i": MiToken}))
            }).then((res) => {
                return res.json();
            }).then((json) => {
                MiName = json.name; // Miのユーザー名
                MiId = json.id; // MiのユーザーID
            });
        } else {
            User.MisskeyLogin = false; // misskeyログイン済みのフラグを立てる
        }
    })
</script>

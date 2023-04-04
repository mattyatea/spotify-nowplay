<div style="position: absolute;top: 50%;left: 50%;transform: translate(-50%,-50%); min-width: 280px; ">
    <div class="a" class:Activity_Login={Activity_Login}>
        <div>
            <Textfield variant="filled" bind:value={Url} label="Instance Url">
                <HelperText slot="helper">例: https://misskey.io</HelperText>
            </Textfield>
        </div>
    </div>
    {#if !SpotifyNowLogin}
        <Button on:click={() => SpotifyLogin = !SpotifyLogin} style="background-color: #1db954 !important;" touch
                variant="raised"
                target="_blank">
            <Label>Spotifyにログイン</Label>
        </Button>
    {:else}
        <Button on:click={() => SpotifyLogout = !SpotifyLogout} style="background-color: #1db954 !important;" touch
                variant="raised"
                target="_blank">
            <Label>Spotifyからログアウト</Label>
        </Button>
    {/if}
    <br>
    {#if !MisskeyNowLogin}
        <Button on:click={() => MisskeyLogin = !MisskeyLogin}
                style="background: linear-gradient(340deg, rgba(93,80,230,1) 32%, rgba(115,153,0,1) 72%););" touch
                variant="raised"
                target="_blank">
            <Label>Misskeyにログイン</Label>
        </Button>
    {:else}
        <Button on:click={() => MisskeyLogout = !MisskeyLogout}
                style="background: linear-gradient(340deg, rgba(93,80,230,1) 32%, rgba(115,153,0,1) 72%););" touch
                variant="raised"
                target="_blank">
            <Label>Misskeyからログアウト</Label>
        </Button>
    {/if}
    <br>
</div>

<Dialog
        bind:open
        aria-labelledby="check"
        aria-describedby="check-dialog"
>
    <Title id="check">確認</Title>
    <Content id="check-dialog">{instance_name} にログインしますか？</Content>
    <Actions>
        <Button on:click={() => (clicked = true)}>
            <Label>する</Label>
        </Button>
        <Button on:click={() => (clicked = false)}>
            <Label>しない</Label>
        </Button>
    </Actions>
</Dialog>

<script>
    import Textfield from '@smui/textfield';
    import Button, {Label} from '@smui/button';
    import HelperText from '@smui/textfield/helper-text';
    import Dialog, {Actions, Content, Title} from '@smui/dialog';
    import {browser} from "$app/environment";

    // 変数定義(これどっか別のファイルにしたい味はあるけどそんなことできるのだろうか...)
    const CallbackBase = import.meta.env.VITE_URL + "/callback/";
    let Url = "https://";
    let Activity_Login, open, chk, SpotifyNowLogin, MisskeyNowLogin, MisskeyLogin, SpotifyLogin, MisskeyLogout,
        SpotifyLogout = false;
    let clicked = null;
    let instance_name = "";
    const clientId = import.meta.env.VITE_SPOTIFY_CLIENT_ID;
    // 変数定義おしまい

    // Spotifyのログイン処理(公式サイトから引っ張ってきただけなのでふんわりしか理解してない
    async function redirectToAuthCodeFlow() {
        const verifier = generateCodeVerifier(128);
        const challenge = await generateCodeChallenge(verifier);

        localStorage.setItem("verifier", verifier);
        window.location = `https://accounts.spotify.com/authorize?client_id=${clientId}&response_type=code&redirect_uri=${CallbackBase}spotify&scope=user-read-currently-playing&code_challenge_method=S256&code_challenge=${challenge}`;
    }

    function generateCodeVerifier(length) {
        let text = '';
        let possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';

        for (let i = 0; i < length; i++) {
            text += possible.charAt(Math.floor(Math.random() * possible.length));
        }
        return text;
    }

    async function generateCodeChallenge(codeVerifier) {
        const data = new TextEncoder().encode(codeVerifier);
        const digest = await window.crypto.subtle.digest('SHA-256', data);
        return btoa(String.fromCharCode.apply(null, [...new Uint8Array(digest)]))
            .replace(/\+/g, '-')
            .replace(/\//g, '_')
            .replace(/=+$/, '');
    }

    //ここまでSpotifyのログイン実装


    if (browser) {
        SpotifyNowLogin = !!localStorage.getItem('SpoToken'); // Spotifyに今ログインしているか ログインしていればtrueを返すようになっているはず...
        MisskeyNowLogin = !!localStorage.getItem('MiToken'); // Misskeyに今ログインしているか ログインしていればtrueを返すようになっているはず...
    }
    /* 未実装 MastodonLogin でもこれやるのDBとか必要そうなので後回し
    function Mastodon_Login(InstanceURI) {
        const BaseURL = InstanceURI + "/api/";
        const AppData = {
            "client_name": "Nowplaying For Activity Pub",
            "redirect_uris": "`{CallbackBase}`",
            "scopes": "read"
        };
        const header = {method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify(AppData)};
        fetch(BaseURL + "apps", header)
    }
    */
    // Misskeyのログイン実装
    function Misskey_login(InstanceURI, R_data) {
        if (parseFloat(R_data.version) <= 12.27) { // Misskey12.27以下の場合app/createを使ってそれ以上の場合はMiAuthを使う ちなみにこれは変なこだわり
            //app/createを使ってアプリを作成
            fetch(InstanceURI + "/api/app/create", {
                method: "POST",
                headers: {"Content-Type": "application/json"},
                body: {
                    "name": "Nowplaying For Activity Pub",
                    "description": "",
                    "permission": ["write:notes"],
                    "callbackUrl": CallbackBase + "misskey"
                }
            }) //app/createを使ってアプリを作成
                .then(res => res.json())
                .then(data => {
                    fetch(InstanceURI + "/api/auth/session/generate", {
                        method: "POST",
                        headers: {"Content-Type": "application/json"},
                        body: JSON.stringify({"appSecret": data.secret})
                    })// app/createで作ったアプリを使ってセッションを生成
                        .then(res => res.json())
                        .then(data => {
                            window.location = data.url; // 生成したセッションのURLに飛ばす
                        })

                })
        } else if (parseFloat(R_data.version) >= 12.28) { // 12.28以上
            const uuid = crypto.randomUUID();//UUID作成
            window.location.href = InstanceURI + "/miauth/" + uuid + "?name=Nowplaying For Activity Pub&callback=" + CallbackBase + "miauth&permission=write:notes";//飛ばす
        }
    }

    $: if (Activity_Login) {
        Url = "https://";
    } //たぶん入力フォームの初期化

    $: chk = Url.match(/^https?:\/\/[^\s/$.?#]+\.[^\s]*$/i); //URLの正規表現
    // ここでインスタンスを確認しているのかも
    $: if (chk) {
        fetch("https://actv-pub-soft-chk.mattya.workers.dev/?url=" + Url)
            .then(res => res.json())
            .then(data => {
                if (data.url) {
                    (data.name == "undefined") ? instance_name = Url : instance_name = data.name;
                    if (data.software == "misskey" && !MisskeyLogin || data.software == "meisskey" && !MisskeyLogin) {
                        open = true;
                    } else {
                        alert(data.software + "は現在サポートしていません。");
                    }
                    /* 謎のコード
                } else if (data.software == "misskey" && MisskeyLogin || data.software == "meisskey" && MisskeyLogin) {
                    alert("すでに" + data.name + "にログインしています。");
                } else if (data.software == "mastodon" && !MastodonLogin) {
                    open = true;
                } else if (data.software == "mastodon" && MastodonLogin) {
                    alert("すでに" + data.name + "にログインしています。");
                } else {
                    alert(data.software + "は現在サポートしていません。");
                }*/
                }
            });
    }
    $: if (SpotifyLogin) {
        redirectToAuthCodeFlow(); //Spotifyのログイン実装
    }

    $: if (SpotifyLogout) { //Spotifyのログアウト実装
        localStorage.removeItem('SpoToken');
        localStorage.removeItem('SpoRefresh');
        localStorage.removeItem('verifier');
        SpotifyNowLogin = false;
    }

    $: if (MisskeyLogout) { //Misskeyのログアウト実装
        localStorage.removeItem('MiToken');
        localStorage.removeItem('MiInstance');
        MisskeyNowLogin = false;
    }

    // ここでログインのセッション作成
    $: if (clicked) {
        fetch("https://actv-pub-soft-chk.mattya.workers.dev/?url=" + Url) // なぜこんなことをしているかというと自分のローカルの環境でテストしたらcorsに引っかかったからなのでこれは普通にしても良いかも
            .then(res => res.json())
            .then(data => {
                if (data.software == "misskey" || data.software == "meisskey") { //misskeyの場合
                    Misskey_login(Url, data);
                } else if (data.software == "mastodon") { //mastdonの場合
                    alert(data.software + "は現在サポートしていません。");
                    Url = "https://";
                } else { //それ以外
                    alert(data.software + "は現在サポートしていません。");
                    Url = "https://";
                }

            });
        open = false;
        clicked = null;
    } else if (clicked === false) {
        Url = "https://";
        open = false;
        clicked = null;
    }

</script>
<style>
    .a {
        animation-name: fadeDownAnime;
        animation-duration: 0.5s;
        animation-fill-mode: forwards;
        visibility: hidden;
        opacity: 0;
    }

    @keyframes fadeDownAnime {
        from {
            opacity: 1;
            transform: translateY(0px);
        }
        to {
            opacity: 0;
            visibility: visible;
            transform: translateY(100px);
        }
    }

    .a.Activity_Login {
        visibility: visible;
        display: block !important;
        animation-name: fadeUpAnime;
        animation-duration: 0.35s;
        animation-fill-mode: forwards;
        opacity: 0;
    }

    @keyframes fadeUpAnime {
        from {
            opacity: 0;
            transform: translateY(100px);
        }
        to {
            opacity: 1;
            transform: translateY(0);
        }
    }
</style>
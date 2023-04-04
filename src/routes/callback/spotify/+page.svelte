<div style="display: flex; justify-content: center">
    <CircularProgress style="height: 32px; width: 32px;" indeterminate/>

</div>
<div style="display: flex; justify-content: center">
    {Check}
</div>

<script>
    import CircularProgress from '@smui/circular-progress';
    import {onMount} from "svelte";

    const CallbackBase = import.meta.env.VITE_URL + "/callback/";
    const clientId = import.meta.env.VITE_SPOTIFY_CLIENT_ID;
    onMount(async () => {
        async function getAccessToken(clientId, code) {
            const verifier = localStorage.getItem("verifier");

            const params = new URLSearchParams();
            params.append("client_id", clientId);
            params.append("grant_type", "authorization_code");
            params.append("code", code);
            params.append("redirect_uri", `${CallbackBase}spotify`);
            params.append("code_verifier", verifier);

            const result = await fetch("https://accounts.spotify.com/api/token", {
                method: "POST",
                headers: {"Content-Type": "application/x-www-form-urlencoded"},
                body: params
            });

            const {access_token, refresh_token} = await result.json();
            return {access_token, refresh_token};
        }

        async function redirectToAuthCodeFlow() {
            const verifier = generateCodeVerifier(128);
            const challenge = await generateCodeChallenge(verifier);

            localStorage.setItem("verifier", verifier);
            window.location = `https://accounts.spotify.com/authorize?client_id=${clientId}&response_type=code&redirect_uri=${CallbackBase}spotify&scope=user-read-private%20user-read-email&code_challenge_method=S256&code_challenge=${challenge}`;
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

        const params = new URLSearchParams(window.location.search);
        const code = params.get("code");

        if (!code) {
            await redirectToAuthCodeFlow(clientId);
        } else {
            const {access_token, refresh_token} = await getAccessToken(clientId, code);
            console.log(access_token, refresh_token);
            localStorage.setItem('SpoToken', access_token);
            localStorage.setItem('SpoRefresh', refresh_token);
            Check = "認証に成功しました";
            location.href = "/";
        }

    });

    let Check = "認証中です";
</script>
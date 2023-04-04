<div style="display: flex; justify-content: center">
    <CircularProgress style="height: 32px; width: 32px;" indeterminate />

</div>
<div style="display: flex; justify-content: center">
    {Check}
</div>

<script>
    import CircularProgress from '@smui/circular-progress';
    import { onMount } from 'svelte';
    let Check = "認証中です";

    onMount(() => {
        const referrer = document.referrer;
        const urlParams = new URLSearchParams(window.location.search);
        function forEachKey(callback) {
            for (var i = 0; i < localStorage.length; i++) {

            }
        }
        fetch(referrer+"api/miauth/"+urlParams.get('session')+"/check",{method: 'POST',mode: 'cors'})
            .then(res => res.json() )
            .then(data => {
                console.log(data);
                if (data.ok){
                    localStorage.setItem('MiToken', data.token);
                    localStorage.setItem('MiInstance', referrer);
                    Check = "認証に成功しました";
                    location.href = "/";
                }else{
                    Check = "認証に失敗しました";
                    location.href = "/account";
                }
            })
    });
</script>
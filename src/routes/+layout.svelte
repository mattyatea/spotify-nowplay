<div class="top-app-bar-container">
    <TopAppBar style="background-color: #1ed760;"
               variant="static"
               {prominent}
               {dense}
               color={secondaryColor ? 'secondary' : 'primary'}
    >
        <Row>
            <Section>
                <a href="/"><Title>Now playing - Alpha</Title></a>
            </Section>
            <Section align="end" toolbar>
                <a href="/account">
                    <IconButton class="material-icons" aria-label="Account"
                    >account_circle
                    </IconButton
                    >
                </a>
            </Section>
        </Row>
    </TopAppBar>
    <div>

    </div>
</div>
<slot/>
<script lang="ts">
    import TopAppBar, {Row, Section, Title} from '@smui/top-app-bar';
    import IconButton from '@smui/icon-button';

    let prominent = false;
    let dense = false;
    let secondaryColor = false;
</script>

<style>
    a {
        text-decoration: none !important;
        color: #fff !important;
    }

    .top-app-bar-container {
        min-width: 280px;
        width: 100%;
        height: 100%;
        overflow: auto;
        display: inline-block;
    }
</style>

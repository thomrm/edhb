<script>
    import '../lib/base.css';
    import '../lib/colors.css';
    import { onMount } from 'svelte';
 
    let ready = false;

    onMount(() => {
        ready = true;
    });
</script>

{#if ready}
    <div class="content-contain">
        <slot />
    </div>
{/if}

<style>
    :root {
        overflow-y: scroll;
        scrollbar-color: var(--Background-Object) var(--Background-Color);
        font-size: 10px;

        & body {
            background: var(--Background-Color);
            color: var(--Text-Primary);
            font-family: 'Nunito Sans', serif;
            font-weight: 600;
            font-size: 1.4rem
        }

        & a {
            color: var(--Primary-Color);
        }
    }

    .content-contain {
        display: flex;
        flex-direction: column;
        max-width: 1200px;
        min-height: 100dvh;
        padding: 0 50px;
        margin: 0 auto;
    }

    @media screen and (max-width: 720px) {
        .content-contain { padding: 0 10px; }
    }
</style>
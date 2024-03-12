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
        & body {
            background: var(--Background-Color);
            color: var(--Text-Primary);
            font-family: 'Nunito Sans', serif;
            font-weight: 600;
        }

        & a {
            color: var(--Primary-Color);
        }
    }

    .content-contain {
        display: flex;
        flex-direction: column;
        max-width: 1200px;
        min-height: 100vh;
        padding: 0 50px;
        margin: 0 auto;
    }
</style>
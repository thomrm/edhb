<script>
    import Card from './card.svelte';

	export let showModal;
    export let cardPrints;
    export let sortAsc;

    let dialog;

    $: if (dialog && showModal) dialog.showModal();
</script>

<!-- svelte-ignore a11y-click-events-have-key-events a11y-no-noninteractive-element-interactions -->
<dialog class="card-modal" bind:this={dialog} on:close={() => (showModal = false)} on:click|self={() => dialog.close()}>
    <div class="modal-close">
        <button class="button button--large" on:click={() => dialog.close()}>
            <img src="Close.svg" alt="Close" />
        </button>
    </div>
    <div class="cards-contain">
        <!-- <button class="button button--large">
            <img src="ArrowLeft.svg" alt="Previous" />
        </button> -->

        <div class="cards">
            {#if cardPrints}
                {#each (sortAsc ? [...cardPrints].reverse() : cardPrints) as print, i}
                    <Card print={print} totalPrints=null i={i}></Card>
                {/each}
            {/if}
        </div>

        <!-- <button class="button button--large">
            <img src="ArrowRight.svg" alt="Next" />
        </button> -->
    </div>
</dialog>

<style>
    .card-modal {
        padding: 0;
        background: none;
        border: none;
        max-height: 100%;
        max-width: 100%;

        &::backdrop {
            background: rgba(0,0,0,0.85);
        }
    }

    .modal-close {
        position: fixed;
        top: 20px;
        right: 20px;
    }

    .cards-contain {
        width: 80vw;
        padding: 20px;
    }

    .cards {
        display: grid;
        grid-template-columns: repeat(auto-fill,minmax(20rem,1fr));
        gap: 10px;
    }
</style>
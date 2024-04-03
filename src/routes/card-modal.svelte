<script>
    import { fade } from 'svelte/transition';
    import Card from './card.svelte';

	export let showModal;
    export let cardPrints;
    export let sortAsc;

    let dialog;

    let totalPrints;
    let currentPrint;

    $: totalPrints = cardPrints ? cardPrints.length - 1 : 0;

    $: if (dialog && showModal) { dialog.showModal(); currentPrint = sortAsc ? totalPrints : 0; };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events a11y-no-noninteractive-element-interactions -->
<dialog class="card-modal" bind:this={dialog} on:close={() => (showModal = false)} on:click|self={() => dialog.close()}>
    <div class="modal-close">
        <button class="button button--large" on:click={() => dialog.close()}>
            <img src="Close.svg" alt="Close" />
        </button>
    </div>
    <div class="cards-contain">
        <button class="button button--large" disabled={totalPrints == 0 ? true : false} on:click={() => currentPrint = (currentPrint == 0) ? totalPrints : currentPrint - 1}>
            <img src="ArrowLeft.svg" alt="Previous" />
        </button>

        <div class="cards">
            {#if cardPrints}
                {#key currentPrint}
                    <div class="modal-card-contain" in:fade|global={{duration: 200}}>
                        <Card print={cardPrints[currentPrint]} totalPrints=null delay=50></Card>
                    </div>
                {/key}
            {/if}
        </div>

        <button class="button button--large" disabled={totalPrints == 0 ? true : false} on:click={() => currentPrint = (currentPrint == totalPrints) ? 0 : currentPrint + 1}>
            <img src="ArrowRight.svg" alt="Next" />
        </button>
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
            background: var(--Background-Overlay-Dark);
        }
    }

    .modal-close {
        position: fixed;
        top: 20px;
        right: 20px;
    }

    .cards-contain {
        padding: 20px;
        display: flex;
        align-items: center;
        width: 440px;
        gap: 10px;
    }

    .modal-card-contain {
        width: 100%;
    }

    .cards {
        display: flex;
        flex: 1 0 0;
    }
</style>
<script>
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
    <div class="cards-contain">
        <button class="button button--large" disabled={totalPrints == 0 ? true : false} on:click={() => currentPrint = (currentPrint == 0) ? totalPrints : currentPrint - 1}>
            <img src="ArrowLeft.svg" alt="Previous" />
        </button>

        <div class="cards">
            {#if cardPrints}
                {#key currentPrint}
                    <div class="modal-card-contain">
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
        z-index: 5;
    }

    dialog::backdrop {
        background: rgba(0,0,0,0.9);
    }

    dialog[open] {
        animation: fade 400ms ease-out;
    }

	dialog[open]::backdrop {
		animation: fade 200ms ease-out;
	}

	@keyframes fade {
		from { opacity: 0; }
		to { opacity: 1; }
	}

    .cards-contain {
        padding: 10px;
        display: flex;
        align-items: center;
        width: 540px;
        box-sizing: border-box;
        gap: 10px;
        font-size: 14px;
    }

    .modal-card-contain {
        width: 100%;

        & .card {
            font-size: inherit;
        }
    }

    .cards {
        display: flex;
        flex: 1 0 0;
    }

    @media screen and (max-width: 540px) {
        .cards-contain {
            width: 100vw;
            font-size: 2.6vw;
        }
    }
</style>
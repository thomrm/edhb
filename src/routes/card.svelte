<script>
    import { preload } from '../lib/preload.js';
    import { fade } from 'svelte/transition';

    export let print;
    export let totalPrints;
    export let delay;
</script>

<div class="card" class:card--dual={print.layout=="transform" || print.layout=="reversible_card"}>
    <div class="card__image">
        {#if print.image_uris}
            <img class="card-placeholder" src="/Placeholder.svg" alt="Placeholder" />
            {#await preload(print.image_uris.normal)}
                <!--Loading-->
            {:then}
                <img srcset="{print.image_uris.normal}, {print.image_uris.large} 2x" src="{print.image_uris.normal}" alt="{print.name}" in:fade={{ delay: delay, duration: 200 }} />
            {/await}
        {:else}
            {#if print.card_faces}
                <img class="card-placeholder" src="/Placeholder.svg" alt="Placeholder" />
                {#await preload(print.card_faces[0].image_uris.normal)}
                    <!--Loading-->
                {:then}
                    <div class="face-front">
                        <img srcset="{print.card_faces[0].image_uris.normal}, {print.card_faces[0].image_uris.large} 2x" src="{print.card_faces[0].image_uris.normal}" alt="{print.name}" in:fade={{ delay: delay, duration: 200 }} />
                    </div>
                    <div class="face-back">
                        <img srcset="{print.card_faces[1].image_uris.normal}, {print.card_faces[1].image_uris.large} 2x" src="{print.card_faces[1].image_uris.normal}" alt="{print.name}" in:fade={{ delay: delay, duration: 200 }} />
                    </div>
                {/await}
            {/if}
        {/if}
    </div>
    <div class="card__info">
        <div class="price">
            <span>
                {#if print.prices.usd}
                    ${print.prices.usd}
                {:else}
                    --
                {/if}
            </span>
            <div class="card__divider"></div>
            <span class="foil">
                {#if print.prices.usd_foil}
                    ${print.prices.usd_foil}
                {:else if print.prices.usd_etched}
                    ${print.prices.usd_etched} &#10023;
                {:else}
                    --
                {/if}
            </span>
        </div>
        {#if totalPrints && totalPrints > 1}
            <div class="other-prints">
                <span>+{totalPrints-1}</span>
                <img src="Print.svg" alt="Printings" />
            </div>
        {/if}
        <span>{#if print.released_at}{print.released_at.slice(0,4)}{:else}--{/if}</span>
    </div>
</div>

<style>
    .card {
        display: flex;
        padding: 5px;
        gap: 5px;
        flex-direction: column;
        border-radius: 8% / 6%;
        background: var(--Background-Overlay-Light);
        box-shadow: 0 0 0 1px var(--Border-Color);
        perspective: 2000px;
        color: var(--Text-Primary);
    }

    .card-placeholder {
        width: 100%;
        height: 100%;
    }

    .card__image {
        width: auto;
        height: 0;
        padding-top: 140%;
        position: relative;
        display: flex;
        border-radius: 6.5% / 5%;

        transition: transform 600ms;
        transform-style: preserve-3d;

        & img {
            width: 100%;
            height: 100%;
            border-radius: 6.5% / 5%;
            position: absolute;
            top: 0;
        }
    }


    .face-front, .face-back {
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        backface-visibility: hidden;
    }

    .face-back {
        transform: rotateY(180deg);
    }

    .card__info {
        font-size: .9rem;
        padding: 0 12px;
        display: flex;
        justify-content: space-between;
    }

    .card__divider {
        width: 4px;
        height: 4px;
        transform: rotate(45deg);
        background: var(--Background-Object);
    }

    .price {
        display: flex;
        align-items: center;
        gap: 6px;
    }

    .foil {
        background: linear-gradient(90deg, rgba(191,147,255,1) 0%, rgba(115,239,255,1) 100%);
        background-clip: text;
        -webkit-text-fill-color: transparent;
    }

    .other-prints {
        display: flex;
        gap: 4px;
        color: var(--Text-Secondary);
    }

    @media (hover: hover) {
        .card--dual:hover .card__image {
            transform: rotateY(180deg);
        }
    }
</style>
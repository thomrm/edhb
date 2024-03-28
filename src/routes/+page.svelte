<script>
    import { onMount } from "svelte";
    import { fly, fade } from 'svelte/transition';
	import { expoOut } from 'svelte/easing';
    import { page } from '$app/stores';
    import { goto } from "$app/navigation";

    // URL params
    const queryM = $page.url.searchParams.get('m') ? $page.url.searchParams.get('m') : "wubrg";
    const queryS = $page.url.searchParams.get('s') ? $page.url.searchParams.get('s') : false;
    let mString;
    let sVal;
    let sString;

    // Import SVGs - for better formatting
    import svgColorW from "../lib/svg/white.svelte";
    import svgColorU from "../lib/svg/blue.svelte";
    import svgColorB from "../lib/svg/black.svelte";
    import svgColorR from "../lib/svg/red.svelte";
    import svgColorG from "../lib/svg/green.svelte";
    import svgColorC from "../lib/svg/colorless.svelte";

    // Full card lists
    let cardsRaw = [];
    let cardsPre;
    let cards;

    // Card filters and sorting
    let filteredCards;
    let filters = {
        color: {
            white: queryM && queryM.includes("w") ? true : false,
            blue: queryM && queryM.includes("u") ? true : false,
            black: queryM && queryM.includes("b") ? true : false,
            red: queryM && queryM.includes("r") ? true : false,
            green: queryM && queryM.includes("g") ? true : false,
            colorless: queryM && queryM.includes("c") ? true : false,
            strict: queryS && queryS.includes(true) ? true : false
        },
        latestPrint: true,
        sortType: "release",
        sortAsc: false,
        pageSize: 30
    }
    let currentPage = 0;
    let searchTerm;

    // Total numbers for display
    let totalCards;
    let totalPages;

    // Object.groupBy() alternative - safari fix
    const groupBy = (x,f)=>x.reduce((a,b,i)=>((a[f(b,i,x)]||=[]).push(b),a),{});

    // Scryfall search filters
    let filtersString = [
        '%28%28type%3Acreature+type%3Alegendary%29+OR+%28oracle%3A"can+be+your+commander"+type%3Aplaneswalker%29+OR+%28Grist%2C+The+Hunger+Tide%29%29',
        '%28game%3Apaper%29',
        'unique%3Aprints'
    ]
    
    onMount(async () => {
        // Get data from Scryfall api
        const response = await fetch('https://api.scryfall.com/cards/search?q='+filtersString.join("+"));
        const data = await response.json();

        // Combine all pages into one object
        for (let i = 1; i <= Math.ceil(data.total_cards / 175); i++) {
            const rawPage = await fetch('https://api.scryfall.com/cards/search?q='+filtersString.join("+")+'&page='+i);
            const rawPageData = await rawPage.json();

            cardsRaw = cardsRaw.concat(rawPageData.data);
        }

        // Pre-filter card data
        cardsPre = cardsRaw
            .filter(x => x.type_line ? !x.type_line.includes("Token") : true)
            .filter(x => x.card_faces ? !x.card_faces[0].type_line.includes("Land") : true)
            .filter(x => x.lang.includes("en"))
            .filter(x => x.promo_types ? !x.promo_types.includes("stamped") && !x.promo_types.includes("prerelease") && !x.promo_types.includes("serialized") : true)
            .filter(x => !x.oversized)
            .filter(x => !x.set.includes("plst"))
            .filter(x => !x.border_color.includes("silver") && !x.border_color.includes("gold"));

        // Initial filter
        filterCards();
    });

    $: filterCards = async (reset) => {
        // Force a page reset - used for clearing search term
        currentPage = reset ? 0 : currentPage;
        totalCards = reset ? null : totalCards;

        // Sort cards by release by default
        cards = cardsPre.sort((a, b) => (filters.latestPrint ? new Date(b.released_at) - new Date(a.released_at) : new Date(a.released_at) - new Date(b.released_at)));
        
        // Group all prints together
        cards = Object.entries(groupBy(cards, ({ name }) => name));

        // Reset displayed cards
        filteredCards = cards;

        // Filter by color
        if (filters.color.strict) {
            filteredCards = filters.color.white ? filteredCards.filter(x => x[1][0].color_identity.includes("W")) : filteredCards.filter(x => !x[1][0].color_identity.includes("W"));
            filteredCards = filters.color.blue ? filteredCards.filter(x => x[1][0].color_identity.includes("U")) : filteredCards.filter(x => !x[1][0].color_identity.includes("U"));
            filteredCards = filters.color.black ? filteredCards.filter(x => x[1][0].color_identity.includes("B")) : filteredCards.filter(x => !x[1][0].color_identity.includes("B"));
            filteredCards = filters.color.red ? filteredCards.filter(x => x[1][0].color_identity.includes("R")) : filteredCards.filter(x => !x[1][0].color_identity.includes("R"));
            filteredCards = filters.color.green ? filteredCards.filter(x => x[1][0].color_identity.includes("G")) : filteredCards.filter(x => !x[1][0].color_identity.includes("G"));
            filteredCards = filters.color.colorless ? filteredCards.filter(x => x[1][0].color_identity.length == 0) : filteredCards.filter(x => !x[1][0].color_identity.length == 0);
        } else {
            filteredCards = !filters.color.white ? filteredCards.filter(x => !x[1][0].color_identity.includes("W")) : filteredCards;
            filteredCards = !filters.color.blue ? filteredCards.filter(x => !x[1][0].color_identity.includes("U")) : filteredCards;
            filteredCards = !filters.color.black ? filteredCards.filter(x => !x[1][0].color_identity.includes("B")) : filteredCards;
            filteredCards = !filters.color.red ? filteredCards.filter(x => !x[1][0].color_identity.includes("R")) : filteredCards;
            filteredCards = !filters.color.green ? filteredCards.filter(x => !x[1][0].color_identity.includes("G")) : filteredCards;
            filteredCards = !filters.color.colorless ? filteredCards.filter(x => !x[1][0].color_identity.length == 0) : filteredCards;
        }

        // Sort filtered cards
        if (filters.sortType === 'name') {
            filteredCards = filteredCards.sort((a, b) => (filters.sortAsc ? 1 : -1) * a[1][0].name.localeCompare(b[1][0].name));
        }
        if (filters.sortType === 'cmc') {
            filteredCards = filteredCards.sort((a, b) => (filters.sortAsc ? a[1][0].cmc - b[1][0].cmc : b[1][0].cmc - a[1][0].cmc) || (a[1][0].name.localeCompare(b[1][0].name)) || (new Date(b[1][0].released_at) - new Date(a[1][0].released_at)));
        }
        if (filters.sortType === 'release') {
            filteredCards = filteredCards.sort((a, b) => (filters.sortAsc ? new Date(a[1][0].released_at) - new Date(b[1][0].released_at) : new Date(b[1][0].released_at) - new Date(a[1][0].released_at)) || (a[1][0].name.localeCompare(b[1][0].name)));
        }

        // Filter displayed cards by search term
        filteredCards = searchTerm ? filteredCards.filter(function(x) {
            if (x[1][0].card_faces) {
                // Check both faces if dual faced card
                return  x[1][0].card_faces[0].oracle_text ? x[1][0].card_faces[0].oracle_text.toLowerCase().includes(searchTerm.toLowerCase()) || 
                        x[1][0].card_faces[0].type_line.toLowerCase().includes(searchTerm.toLowerCase()) ||
                        x[1][0].card_faces[0].name.toLowerCase().includes(searchTerm.toLowerCase()) || 
                        x[1][0].card_faces[1].oracle_text.toLowerCase().includes(searchTerm.toLowerCase()) || 
                        x[1][0].card_faces[1].type_line.toLowerCase().includes(searchTerm.toLowerCase()) ||
                        x[1][0].card_faces[1].name.toLowerCase().includes(searchTerm.toLowerCase()) : null;
            } else {
                // Check oracle text, type line, and name
                return  x[1][0].oracle_text ? x[1][0].oracle_text.toLowerCase().includes(searchTerm.toLowerCase()) || 
                        x[1][0].type_line.toLowerCase().includes(searchTerm.toLowerCase()) ||
                        x[1][0].name.toLowerCase().includes(searchTerm.toLowerCase()) : null;
            }
        }) : filteredCards;

        // Set card and page totals
        totalPages = Math.ceil(filteredCards.length / filters.pageSize);
        totalCards = totalCards ? totalCards : filteredCards.length;

        //console.log(filteredCards);

        updateURLParams();
    }

    const clearSearch = () => {
        // Clear search term
        searchTerm = null;

        // Reset displayed cards
        filterCards(true);
    }

    const updateURLParams = async () => {
        mString = 
            (filters.color.white ? 'w' : '')+
            (filters.color.blue ? 'u' : '')+
            (filters.color.black ? 'b' : '')+
            (filters.color.red ? 'r' : '')+
            (filters.color.green ? 'g' : '')+
            (filters.color.colorless ? 'c' : '');
        sVal = filters.color.strict ? true : false;
        sString = searchTerm ? '&search='+searchTerm : '';
        goto('?m='+mString+'&s='+sVal+sString);
    }

    // Card image preload
    const preload = async (src) => {
        const resp = await fetch(src);
        const blob = await resp.blob();

        return new Promise(function (resolve) {
            let reader = new FileReader();
            reader.readAsDataURL(blob);
            reader.onload = () => resolve(reader.result);
            reader.onerror = (error) => reject('Error: ', error);
        });
    };
</script>

<div class="filters-contain filters-contain--top">
    <div class="filters">
        <div class="filters-group">
            <fieldset class="filter filter--color">
                <div class="filter-group">
                    <input type="checkbox" id="colorW" name="White" bind:checked={filters.color.white} on:change={filterCards} />
                    <label class="color-toggle" for="colorW">
                        <svelte:component this={svgColorW} />
                    </label>

                    <input type="checkbox" id="colorU" name="Blue" bind:checked={filters.color.blue} on:change={filterCards} />
                    <label class="color-toggle" for="colorU">
                        <svelte:component this={svgColorU} />
                    </label>

                    <input type="checkbox" id="colorB" name="Black" bind:checked={filters.color.black} on:change={filterCards} />
                    <label class="color-toggle" for="colorB">
                        <svelte:component this={svgColorB} />
                    </label>

                    <input type="checkbox" id="colorR" name="Red" bind:checked={filters.color.red} on:change={filterCards} />
                    <label class="color-toggle" for="colorR">
                        <svelte:component this={svgColorR} />
                    </label>

                    <input type="checkbox" id="colorG" name="Green" bind:checked={filters.color.green} on:change={filterCards} />
                    <label class="color-toggle" for="colorG">
                        <svelte:component this={svgColorG} />
                    </label>

                    <input type="checkbox" id="colorC" name="Colorless" bind:checked={filters.color.colorless} on:change={filterCards} />
                    <label class="color-toggle" for="colorC">
                        <svelte:component this={svgColorC} />
                    </label>
                </div>

                <div class="filter__divider"></div>

                <div class="filter-group">
                    <input class="switch-box" type="checkbox" id="strict-match" bind:checked={filters.color.strict} on:change={filterCards} />
                    <label class="switch" for="strict-match"></label>
    
                    <div class="filter__label">Strict</div>
                </div>
            </fieldset>

            <fieldset class="filter">
                <form class="search-form" on:submit|preventDefault={filterCards} on:reset={clearSearch}>
                    <input type="text" class="search" placeholder="Search" bind:value={searchTerm} on:change={filterCards} />
                    {#if searchTerm}
                        <button type="reset" class="search-clear">
                            <img src="/Clear.svg" alt="Clear Search" />
                        </button>
                    {/if}
                </form>
            </fieldset>
        </div>

        <div class="filters-group">
            <fieldset class="filter">
                <input class="toggle-box" type="checkbox" id="printing" bind:checked={filters.latestPrint} on:change={filterCards} />
                <label class="toggle" for="printing">
                    <div class="filter__label">
                        {#if filters.latestPrint}
                            Latest Print
                        {:else}
                            First Print
                        {/if}
                    </div>
                </label>

                <div class="filter__divider"></div>

                <select class="select-box" name="sort-type" id="sort-type" bind:value={filters.sortType} on:change={filterCards}>
                    <option value="name">Sort by Name</option>
                    <option value="cmc">Sort by CMC</option>
                    <option value="release">Sort by Release</option>
                </select>

                <div class="filter__divider"></div>

                <input class="toggle-box" type="checkbox" id="sort-direction" bind:checked={filters.sortAsc} on:change={filterCards} />
                <label class="toggle" for="sort-direction">
                    <div class="filter__label">
                        {#if filters.sortAsc}
                            <img src="Asc.svg" alt="Ascending" />
                            Asc
                        {:else}
                            <img src="Desc.svg" alt="Descending" />
                            Desc
                        {/if}
                    </div>
                </label>
            </fieldset>
        </div>
    </div>
</div>

<div class="grid-contain">
    {#key currentPage}
        {#if filteredCards}
            {#if filteredCards.length === 0}
                <div class="grid-status">No Results</div>
            {:else}
                <div class="card-grid">
                    {#key filteredCards}
                        {#each filteredCards.slice(currentPage * filters.pageSize, currentPage * filters.pageSize + filters.pageSize) as card, i}
                            <div class="card" class:card--dual={card[1][0].layout=="transform"} in:fly|global={{ delay: (i+1)*50, duration: 800, y: 150, opacity: 0, easing: expoOut }}>
                                <div class="card__image">
                                    {#if card[1][0].image_uris}
                                        <img class="card-placeholder" src="/Placeholder.svg" alt="Placeholder" />
                                        {#await preload(card[1][0].image_uris.normal)}
                                            <!--Loading-->
                                        {:then}
                                            <img srcset="{card[1][0].image_uris.normal}, {card[1][0].image_uris.large} 2x" src="{card[1][0].image_uris.normal}" alt="{card[1][0].name}" in:fade|global={{ delay: (i+1)*50, duration: 200 }} />
                                        {/await}
                                    {:else}
                                        {#if card[1][0].card_faces}
                                            <img class="card-placeholder" src="/Placeholder.svg" alt="Placeholder" />
                                            {#await preload(card[1][0].card_faces[0].image_uris.normal)}
                                                <!--Loading-->
                                            {:then}
                                                <div class="face-front">
                                                    <img srcset="{card[1][0].card_faces[0].image_uris.normal}, {card[1][0].card_faces[0].image_uris.large} 2x" src="{card[1][0].card_faces[0].image_uris.normal}" alt="{card[1][0].name}" in:fade|global={{ delay: (i+1)*50, duration: 200 }} />
                                                </div>
                                                <div class="face-back">
                                                    <img srcset="{card[1][0].card_faces[1].image_uris.normal}, {card[1][0].card_faces[1].image_uris.large} 2x" src="{card[1][0].card_faces[1].image_uris.normal}" alt="{card[1][0].name}" in:fade|global={{ delay: (i+1)*50, duration: 200 }} />
                                                </div>
                                            {/await}
                                        {/if}
                                    {/if}
                                </div>
                                <div class="card__info">
                                    <div class="price">
                                        <span>
                                            {#if card[1][0].prices.usd}
                                                ${card[1][0].prices.usd}
                                            {:else}
                                                --
                                            {/if}
                                        </span>
                                        <div class="card__divider"></div>
                                        <span class="foil">
                                            {#if card[1][0].prices.usd_foil}
                                                ${card[1][0].prices.usd_foil}
                                            {:else if card[1][0].prices.usd_etched}
                                                ${card[1][0].prices.usd_etched} &#10023;
                                            {:else}
                                                --
                                            {/if}
                                        </span>
                                    </div>
                                    <span>{#if card[1][0].released_at}{card[1][0].released_at}{:else}--{/if}</span>
                                </div>
                            </div>
                        {/each}
                    {/key}
                </div>
            {/if}
        {:else}
            <div class="grid-status">
                <svg class="loading" width="44" height="44" viewBox="0 0 44 44" fill="none" xmlns="http://www.w3.org/2000/svg">
                    <circle class="shape shape--5" cx="4" cy="18" r="4" fill="white"/>
                    <circle class="shape shape--4" cx="10" cy="39" r="4" fill="white"/>
                    <circle class="shape shape--3" cx="34" cy="39" r="4" fill="white"/>
                    <circle class="shape shape--2" cx="40" cy="18" r="4" fill="white"/>
                    <circle class="shape shape--1" cx="22" cy="5" r="4" fill="white"/>
                </svg>
            </div>
        {/if}
    {/key}
</div>

<div class="filters-contain filters-contain--bottom">
    <div class="filters pagination-contain">
        <div class="filter">
            <div class="filter-group">
                <button class="button" disabled={currentPage == 0 ? true : false} on:click={() => currentPage = 0}>
                    <img src="ArrowLeftEnd.svg" alt="First Page" />
                </button>
                <button class="button" disabled={currentPage == 0 ? true : false} on:click={() => currentPage--}>
                    <img src="ArrowLeft.svg" alt="Previous Page" />
                </button>
            </div>

            {#if filteredCards}
                <div class="filter-group">
                    <div class="filter__label small-hide">Page {currentPage + 1}</div>
                    <div class="filter__divider small-hide"></div>
                    <div class="filter__label">
                        {totalCards ? (currentPage * filters.pageSize + 1) + " - " + (Math.min((currentPage + 1) * filters.pageSize, totalCards)) + " of " : ""}
                        {totalCards} {totalCards == 1 ? "Result" : "Results"}
                    </div>
                </div>
            {/if}

            <div class="filter-group">
                <button class="button" disabled={currentPage == totalPages - 1 || totalPages == 0 || !totalPages ? true : false} on:click={() => currentPage++}>
                    <img src="ArrowRight.svg" alt="Next Page" />
                </button>
                <button class="button" disabled={currentPage == totalPages - 1 || totalPages == 0 || !totalPages ? true : false} on:click={() => currentPage = totalPages - 1}>
                    <img src="ArrowRightEnd.svg" alt="Last Page" />
                </button>
            </div>
        </div>
    </div>
</div>

<style>
    @keyframes -global-animate-shape {
        0% { opacity: 0.15; }
        25% { opacity: 1; }
        50% { opacity: 0.15; }
    }

    .loading {
        & .shape {
            opacity: 0.15;
            animation-name: animate-shape;
            animation-duration: 1000ms;
            animation-fill-mode: forwards;
            animation-iteration-count: infinite;
            animation-timing-function: cubic-bezier(0.65, 0, 0.35, 1);
        }

        & .shape--1 { animation-delay: 0ms; }
        & .shape--2 { animation-delay: 200ms; }
        & .shape--3 { animation-delay: 400ms; }
        & .shape--4 { animation-delay: 600ms; }
        & .shape--5 { animation-delay: 800ms; }
    }

    .filters-contain {
        padding: 20px 0;
        display: flex;
        justify-content: center;
    }

    .filters-contain--top {
        position: sticky;
        z-index: 2;
        top: 0;
    }

    .filters-contain--bottom {
        position: sticky;
        z-index: 2;
        bottom: 0;
    }

    .filters {
        display: flex;
        flex: 1 0 0;
        flex-wrap: wrap;
        justify-content: space-between;
        gap: 10px;
    }

    .filters-group {
        display: flex;
        flex-wrap: wrap;
        gap: 10px;
    }

    .filter {
        display: flex;
        justify-content: space-between;
        align-items: center;
        background: var(--Background-Overlay-Dark);
        box-shadow: 0 0 0 1px var(--Border-Color);
        padding: 8px;
        height: 32px;
        border-radius: 999px;
    }

    .filter-group {
        display: flex;
    }

    .filter__label {
        padding: 0 10px;
        font-size: 1.2rem;
        display: flex;
        align-items: center;
        gap: 6px;
    }

    .filter__divider {
        width: 6px;
        height: 6px;
        transform: rotate(45deg);
        background: var(--Background-Object);
        margin: 10px;
    }

    .filter--color {
        & input[type=checkbox] {
            display: none;
            
            &:checked + label svg {
                color: inherit;
            }
        }

        & .color-toggle {
            display: flex;
            width: 24px;
            height: 24px;
            cursor: pointer;
            border-radius: 999px;
            padding: 2px;
            border: 2px solid transparent;

            transition: border-color 200ms;

            & svg {
                width: 100%;
                height: 100%;
                color: var(--Background-Object);

                transition: color 200ms;
            }
        }
    }

    #colorW + label { color: var(--Mana-W); }
    #colorU + label { color: var(--Mana-U); }
    #colorB + label { color: var(--Mana-B); }
    #colorR + label { color: var(--Mana-R); }
    #colorG + label { color: var(--Mana-G); }
    #colorC + label { color: var(--Mana-C); }

    .grid-status {
        display: flex;
        flex: 1 0 0;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .grid-contain {
        display: flex;
        flex-direction: column;
        flex: 1 0 0;
    }

    .card-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill,minmax(20rem,1fr));
        gap: 10px;
    }

    .card {
        display: flex;
        padding: 5px;
        gap: 5px;
        flex-direction: column;
        border-radius: 8% / 6%;
        background: var(--Background-Overlay-Light);
        box-shadow: 0 0 0 1px var(--Border-Color);
        cursor: pointer;
        outline: 2px solid transparent;
        perspective: 2000px;

        transition: outline 200ms;
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
        padding: 0 15px;
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

    .pagination-contain {
        & .filter {
            width: 420px;
            justify-content: space-between;

            & .filter-group {
                display: flex;
            }
        }
    }

    .button {
        display: flex;
        height: 32px;
        width: 32px;
        justify-content: center;
        align-items: center;
        cursor: pointer;
        background: none;
        border-radius: 999px;
        padding: 2px;
        border: 2px solid transparent;

        transition: border-color 200ms;

        color: #fff;

        &:disabled {
            cursor: default;

            & img {
                opacity: 0.15;
            }

            &:hover {
                border-color: transparent;
            }
        }
    }

    .toggle-box {
        display: none;
    }

    .toggle {
        display: flex;
        height: 24px;
        justify-content: center;
        align-items: center;
        cursor: pointer;
        border-radius: 999px;
        padding: 2px;
        border: 2px solid transparent;
        background-color: var(--Border-Color);

        transition: border-color 200ms;

        color: #fff;
    }

    .switch-box {
        display: none;

        &:checked + .switch:before {
            background-color: var(--Primary-Color);
        }

        &:checked + .switch:after {
            left: 20px;
        }
    }

    .switch {
        position: relative;
        display: inline-flex;
        width: 40px;
        height: 24px;
        padding: 2px;
        align-items: flex-start;
        align-content: flex-start;
        flex-wrap: wrap;
        cursor: pointer;
        border-radius: 999px;
        border: 2px solid transparent;

        transition: border-color 200ms;

        &::before {
            content: '';
            position: absolute;
            width: 40px;
            height: 24px;
            border-radius: 999px;

            background: var(--Background-Object);

            transition: background-color 200ms, color 200ms;
        }

        &::after {
            content: '';
            position: absolute;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            top: 4px;
            left: 4px;

            background-color: var(--Background-Overlay-Full);

            transition: all 200ms;
        }
    }

    select {
        display: flex;
        height: 32px;
        padding: 2px 28px 2px 10px;
        border: 2px solid transparent;
        border-radius: 999px;
        background: none;
        background-image: url("/Select.svg");
        background-repeat: no-repeat;
        background-position: right 12px center;
        background-color: var(--Border-Color);
        color: var(--Text-Primary);
        font: inherit;
        font-size: 1.2rem;
        appearance: none;
        cursor: pointer;

        transition: color 200ms, border-color 200ms;

        &:focus-visible {
            outline: none;
        }
    }

    .search-form {
        position: relative;
        display: flex;
        align-items: center;
        gap: 5px;
        border: 2px solid transparent;
        border-radius: 999px;

        transition: border-color 200ms;

        &:focus-within {
            border-color: var(--Primary-Color);
        }
    }

    .search {
        background: none;
        background-image: url("/Search.svg");
        background-repeat: no-repeat;
        background-position: left 8px center;
        border: none;
        text-overflow: ellipsis;
        color: inherit;
        font: inherit;
        font-size: 1.4rem;
        border-radius: 999px;
        height: 24px;
        padding: 2px 28px;
        width: 120px;
        cursor: pointer;

        &::placeholder {
            color: var(--Text-Secondary);
        }

        &:focus-visible {
            outline: none;
        }
    }

    .search-clear {
        position: absolute;
        right: -2px;
        display: flex;
        height: 32px;
        width: 32px;
        justify-content: center;
        align-items: center;
        cursor: pointer;
        border-radius: 999px;
        background: none;
        border: none;
    }

    @media (hover: hover) {
        .search-form:hover, select:hover, .switch:hover, .toggle:hover, .button:hover, .color-toggle:hover {
            border-color: var(--Primary-Color);
        }

        .card:hover {
            outline: 2px solid var(--Primary-Color);
        }

        .card--dual:hover .card__image {
            transform: rotateY(180deg);
        }
    }

    @media screen and (max-width: 720px) {
        .filters-contain {
            padding: 10px 0;
        }
    }

    @media screen and (max-width: 456px) {
        .small-hide {
            display: none;
        }

        .filters, .filters-group, .filter, .search-form, .search, .search:placeholder-shown, .search:focus {
            width: 100%;
        }

        .card-grid {
            grid-template-columns: repeat(auto-fill,minmax(16rem,1fr));
        }

        .pagination-contain .filter {
            width: 100%;
        }

        .search {
            font-size: 16px;
        }
    }
</style>
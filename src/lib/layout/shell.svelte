<script lang="ts">
    import { browser } from '$app/environment';
    import { beforeNavigate } from '$app/navigation';
    import { page } from '$app/stores';
    import { log } from '$lib/stores/logs';
    import { wizard } from '$lib/stores/wizard';

    export let isOpen = false;
    export let showSideNavigation = false;

    let y: number;

    page.subscribe(({ url }) => {
        isOpen = url.searchParams.get('openNavbar') === 'true';
    });

    const toggleMenu = () => {
        y = 0;
        isOpen = !isOpen;
        if (browser) {
            if (isOpen) {
                document.body.classList.add('u-overflow-hidden');
            } else {
                document.body.classList.remove('u-overflow-hidden');
            }
        }
    };

    /**
     * Cancel navigation when wizard is open and triggered by popstate
     */
    beforeNavigate((n) => {
        if (n.willUnload) return;
        if (!($wizard.show || $wizard.cover)) return;
        if (n.type === 'popstate') {
            n.cancel();
        }
        if (n.type !== 'leave') {
            wizard.hide();
        }
    });
</script>

<svelte:window bind:scrollY={y} />

<main
    class:grid-with-side={showSideNavigation}
    class:is-open={isOpen}
    class:u-hide={$wizard.show || $log.show || $wizard.cover}
    class:is-fixed-layout={$$slots.alert}>
    {#if $$slots.alert}
        <slot name="alert" />
    {/if}
    <header class="main-header u-padding-inline-end-0">
        <button
            class:u-hide={!showSideNavigation}
            class="icon-button is-not-desktop"
            aria-label="Open Menu"
            on:click={toggleMenu}>
            <span class:icon-x={isOpen} class:icon-menu={!isOpen} aria-hidden="true" />
        </button>
        <slot name="header" />
    </header>
    {#if showSideNavigation}
        <nav class="main-side">
            <slot name="side" />
        </nav>
    {/if}
    <section class="main-content">
        {#if $page.data?.header}
            <svelte:component this={$page.data.header} />
        {/if}

        <slot />
        <slot name="footer" />
    </section>
</main>

<style lang="scss">
    main {
        min-height: 100vh;

        &:not(.grid-with-side) {
            display: flex;
            flex-direction: column;
        }
    }

    .main-side {
        z-index: 25;
    }

    @media (max-width: 550.99px), (min-width: 551px) and (max-width: 1198.99px) {
        .main-side {
            top: 4.5rem;
        }
    }
    @media (min-width: 1199px) {
        .grid-with-side {
            grid-template-columns: auto 1fr !important;
        }
    }
</style>

<script lang="ts">
    import { createEventDispatcher } from 'svelte';
    import { fly } from 'svelte/transition';
    import type { Notification } from '../stores/notifications';

    export let type: Notification['type'] = 'info';
    export let icon: Notification['icon'] = null;
    export let title: Notification['title'];
    export let buttons: Notification['buttons'];
    export let message: Notification['message'];
    export let html: Notification['isHtml'] = false;

    const dispatch = createEventDispatcher();
</script>

<div
    class="alert-sticky"
    class:is-success={type === 'success'}
    class:is-warning={type === 'warning'}
    class:is-danger={type === 'error'}
    class:is-info={type === 'info'}
    transition:fly|global={{ x: 50 }}>
    <button
        class="button is-text is-only-icon"
        style="--button-size:1.5rem;"
        aria-label="close alert"
        on:click={() => dispatch('dismiss')}>
        <span class="icon-x" aria-hidden="true" />
    </button>
    <div class="alert-sticky-image">
        <span
            class:icon-check-circle={type === 'success' && !icon}
            class:icon-exclamation={type === 'warning' && !icon}
            class:icon-exclamation-circle={type === 'error' && !icon}
            class:icon-info={type === 'info'}
            class={icon ? `icon-${icon}` : ''}
            aria-hidden="true" />
    </div>
    <div class="alert-sticky-content">
        {#if title}
            <h4 class="alert-sticky-title">{title}</h4>
        {/if}
        {#if html}
            {@html message}
        {:else}
            <p>{message}</p>
        {/if}
    </div>
    {#if buttons}
        <div class="alert-sticky-buttons u-flex">
            {#each buttons as button}
                <button
                    class="button is-text is-small"
                    on:click|preventDefault|stopPropagation={button.method}>
                    <span class="text">{button.name}</span>
                </button>
            {/each}
        </div>
    {/if}
</div>

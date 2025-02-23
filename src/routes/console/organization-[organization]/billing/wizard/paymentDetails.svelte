<script lang="ts">
    import { FormList } from '$lib/elements/forms';
    import { WizardStep } from '$lib/layout';
    import { onMount } from 'svelte';
    import type { PaymentList } from '$lib/sdk/billing';
    import { invalidate } from '$app/navigation';
    import { Dependencies } from '$lib/constants';
    import { initializeStripe, isStripeInitialized, submitStripeCard } from '$lib/stores/stripe';
    import { sdk } from '$lib/stores/sdk';
    import { PaymentBoxes } from '$lib/components/billing';
    import { addCreditWizardStore } from '../store';

    let methods: PaymentList;
    let name: string;

    onMount(async () => {
        methods = await sdk.forConsole.billing.listPaymentMethods();
        $addCreditWizardStore.paymentMethodId =
            methods.paymentMethods.find((method) => !!method?.last4)?.$id ?? null;
    });

    async function handleSubmit() {
        try {
            const method = await submitStripeCard(name);
            $addCreditWizardStore.paymentMethodId = method.$id;
            invalidate(Dependencies.PAYMENT_METHODS);
        } catch (e) {
            throw new Error('Something went wrong. Please try again.');
        }
    }

    $: if ($addCreditWizardStore.paymentMethodId === null && !$isStripeInitialized) {
        initializeStripe();
    }

    $: if ($addCreditWizardStore.paymentMethodId) {
        isStripeInitialized.set(false);
    }

    $: filteredMethods = methods?.paymentMethods.filter((method) => !!method?.last4);
</script>

<WizardStep beforeSubmit={handleSubmit}>
    <svelte:fragment slot="title">Payment details</svelte:fragment>
    <svelte:fragment slot="subtitle">
        Add a payment method to your organization. You will not be charged until you upgrade to a
        Pro or Scale plan.
    </svelte:fragment>

    <p class="text"><b>Payment method</b></p>
    <FormList class="u-margin-block-start-8">
        <PaymentBoxes
            methods={filteredMethods}
            bind:name
            bind:group={$addCreditWizardStore.paymentMethodId} />
    </FormList>
</WizardStep>

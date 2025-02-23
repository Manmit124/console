<script lang="ts">
    import { Modal } from '$lib/components';
    import { Button } from '$lib/elements/forms';
    import {
        Table,
        TableBody,
        TableCellHead,
        TableCellText,
        TableHeader,
        TableRow
    } from '$lib/elements/table';
    import { toLocaleDate } from '$lib/helpers/date';
    import { organization } from '$lib/stores/organization';
    import { createOrganization } from './store';
    import { plansInfo, type Tier } from '$lib/stores/billing';
    import { abbreviateNumber } from '$lib/helpers/numbers';
    import { BillingPlan } from '$lib/constants';

    export let show = false;
    export let tier: Tier;

    $: plan = $plansInfo?.get(tier);

    $: nextDate = $createOrganization?.name
        ? new Date(new Date().getFullYear(), new Date().getMonth() + 1, 1).toString()
        : $organization?.billingNextInvoiceDate;

    const planData = [
        {
            id: 'members',
            resource: 'Organization members',
            unit: 'member'
        },
        { id: 'bandwidth', resource: 'Bandwidth', unit: 'GB' },
        { id: 'storage', resource: 'Storage', unit: 'GB' },
        {
            id: 'executions',
            resource: 'Function executions',
            unit: ' executions'
        },
        {
            id: 'users',
            resource: 'Users',
            unit: ' Users'
        },
        {
            id: 'realtime',
            resource: 'Concurrent connections',
            unit: ' connections'
        }
    ];

    $: isFree = tier === BillingPlan.STARTER;
</script>

<Modal bind:show size="big" headerDivider={false} title="Usage rates">
    {#if isFree}
        Usage on the Starter plan is limited for the following resources. Next billing period: {toLocaleDate(
            nextDate
        )}.
    {:else if tier === BillingPlan.PRO}
        <p>
            Usage on the Pro plan will be charged at the end of each billing period at the following
            rates. Next billing period: {toLocaleDate(nextDate)}.
        </p>
    {:else if tier === BillingPlan.SCALE}
        <p>
            Usage on the Scale plan will be charged at the end of each billing period at the
            following rates. Next billing period: {toLocaleDate(nextDate)}.
        </p>
    {/if}
    <Table noStyles>
        <TableHeader>
            <TableCellHead>Resource</TableCellHead>
            <TableCellHead>Limit</TableCellHead>
            {#if !isFree}
                <TableCellHead>Rate</TableCellHead>
            {/if}
        </TableHeader>
        <TableBody>
            {#each planData as usage}
                {#if usage['id'] === 'members'}
                    <TableRow>
                        <TableCellText title="resource">{usage.resource}</TableCellText>
                        <TableCellText title="limit">
                            {plan[usage.id] || 'Unlimited'}
                        </TableCellText>
                        {#if !isFree}
                            <TableCellText title="rate">
                                ${plan.addons.member.price}/{usage?.unit}
                            </TableCellText>
                        {/if}
                    </TableRow>
                {:else}
                    {@const addon = plan.addons[usage.id]}
                    <TableRow>
                        <TableCellText title="resource">{usage.resource}</TableCellText>
                        <TableCellText title="limit">
                            {abbreviateNumber(plan[usage.id])}{usage?.unit}
                        </TableCellText>
                        {#if !isFree}
                            <TableCellText title="rate">
                                ${addon?.price}/{['MB', 'GB', 'TB'].includes(addon?.unit)
                                    ? addon?.value
                                    : abbreviateNumber(addon?.value, 0)}{usage?.unit}
                            </TableCellText>
                        {/if}
                    </TableRow>
                {/if}
            {/each}
        </TableBody>
    </Table>
    <svelte:fragment slot="footer">
        <Button text on:click={() => (show = false)}>Close</Button>
    </svelte:fragment>
</Modal>

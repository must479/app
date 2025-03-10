<script lang="ts">
  import tokens from '$lib/stores/tokens';
  import assert from '$lib/utils/assert';
  import formatTokenAmount from '$lib/utils/format-token-amount';
  import amtDeltaUnitStore, {
    FRIENDLY_NAMES,
    MULTIPLIERS,
  } from '$lib/stores/amt-delta-unit/amt-delta-unit.store';
  import contractConstants from '$lib/utils/sdk/utils/contract-constants';

  interface Amount {
    amount: bigint;
    tokenAddress: string;
  }

  export let amount: Amount | undefined = undefined;
  export let amountPerSecond: Amount | undefined = undefined;
  export let showSymbol = true;
  export let showPlusMinus = true;
  export let multiplier = BigInt(contractConstants.AMT_PER_SEC_MULTIPLIER);

  export let amountClasses = 'typo-text tabular-nums';
  export let amountPerSecClasses = 'typo-text-small tabular-nums text-foreground-level-4';

  /** Manually set token information to display. Used on the landing page's mock dashboard. */
  export let overrideToDisplay:
    | {
        symbol: string;
        decimals: number;
      }
    | undefined = undefined;

  $: amountTokenInfo = $tokens && amount ? tokens.getByAddress(amount.tokenAddress) : undefined;
  $: amountPerSecondTokenInfo =
    $tokens && amountPerSecond ? tokens.getByAddress(amountPerSecond.tokenAddress) : undefined;

  function format(
    amount: Amount,
    perSec = false,
    multiplier: bigint,
    perSecTimeUnitMultiplier: number,
  ) {
    let amountToShow = { ...amount };

    const tokenDecimals =
      overrideToDisplay?.decimals ?? tokens.getByAddress(amount.tokenAddress)?.info.decimals;
    assert(
      tokenDecimals,
      `Unable to determine decimals for tokenAddress ${amountToShow.tokenAddress}`,
    );

    if (perSec) {
      amountToShow.amount = amountToShow.amount * BigInt(perSecTimeUnitMultiplier);
    }

    return formatTokenAmount(amountToShow, tokenDecimals, multiplier, !perSec);
  }
</script>

<div class="wrapper">
  {#if amount !== undefined}
    {#if amountTokenInfo || overrideToDisplay}
      <div class="amount">
        <span class="amount-wrapper {amountClasses}">
          <span class="amount">
            {format(amount, false, multiplier, MULTIPLIERS[$amtDeltaUnitStore])}
          </span>
          {#if showSymbol}
            <span class="symbol">
              {overrideToDisplay?.symbol ?? amountTokenInfo?.info.symbol}
            </span>
          {/if}
        </span>
      </div>
    {:else}
      <span class="typo-text unknown"> Unknown token </span>
    {/if}
  {/if}
  {#if amountPerSecond !== undefined}
    {#if amountPerSecondTokenInfo || overrideToDisplay}
      <div class="amount-per-second {amountPerSecClasses}">
        <span
          class="amount"
          class:text-positive={amountPerSecond.amount > 0}
          class:text-negative={amountPerSecond.amount < 0}
          >{#if showPlusMinus}{amountPerSecond.amount > 0 ? '+' : ''}{/if}{format(
            amountPerSecond,
            true,
            multiplier,
            MULTIPLIERS[$amtDeltaUnitStore],
          )}{#if showSymbol}
            {' ' + (overrideToDisplay?.symbol ?? amountPerSecondTokenInfo?.info.symbol)}
          {/if}</span
        >/{FRIENDLY_NAMES[$amtDeltaUnitStore]}
      </div>
    {/if}
  {/if}
</div>

<style>
  .wrapper {
    display: flex;
    flex-direction: column;
    user-select: none;
  }

  .unknown {
    color: var(--color-foreground-level-4);
  }
</style>

<script lang="ts">
  import Grid from 'svelte-grid';
  import debounce from 'just-debounce-it';
  import { pb } from '../glue/pocketbase';

  export let section;

  let selectedItemId: string | null = null;
  let items =
    section.expand['items(section)'].map((item) => ({
      ...item,
      ...item.position
    })) || [];
  const COL_COUNT = 24; // TODO: make responsive
  let container;

  const updateSelectedItemStyles = ({ styleKey, styleValue }) => {
    items = items.map((item) => {
      if (item.id !== selectedItemId) return item;

      item.styles[styleKey] = styleValue;
      pb.collection('items').update(selectedItemId, {
        styles: item.styles
      });
      pb.collection;
      return item;
    });
  };

  const debouncedUpdateItem = debounce(({ itemId, data }) => {
    pb.collection('items').update(itemId, data);
  }, 500);

  const debouncedUpdateMovedItem = debounce((event) => {
    const updatedItem = event.detail.unsafeItem;
    pb.collection('items').update(updatedItem.id, {
      position: {
        ...updatedItem.position,
        [COL_COUNT]: updatedItem[COL_COUNT]
      }
    });
  }, 500);
</script>

<div
  class="border border-transparent hover:border-blue-400 w-full min-h-[80vh]"
  on:click={() => {
    selectedItemId = null;
  }}
  bind:this={container}
>
  <Grid
    bind:items
    gap={[11, 11]}
    rowHeight={30}
    let:item
    let:dataItem
    cols={[[1100, COL_COUNT]]}
    fastStart={true}
    sensor={100}
    on:change={debouncedUpdateMovedItem}
    scroller={container}
  >
    <div class="w-full h-full">
      {#if dataItem?.variant === 'TEXT'}
        <p
          class="{Object.values(dataItem?.styles)?.join(
            ' '
          )} focus:border-blue-400 border border-transparent w-full rounded p-2"
          contenteditable={true}
          on:click={(event) => {
            selectedItemId = dataItem.id;
            event.stopPropagation();
          }}
          on:input={(event) => {
            debouncedUpdateItem({
              itemId: dataItem?.id,
              data: {
                value: event?.currentTarget?.textContent
              }
            });
          }}
        >
          {dataItem.value}
        </p>
      {/if}
    </div>
  </Grid>
</div>

<!-- edit styles panel -->
{#if selectedItemId}
  <div class="fixed right-4 top-[50%] -translate-y-1/2 z-20">
    <div class="px-4 py-6 shadow bg-base-300 rounded-box w-52 border border-base-content/20">
      <h3 class="uppercase text-sm font-extrabold mb-3 ml-1 text-base-content/80">Text</h3>
      <button
        class="btn btn-ghost btn-block text-left justify-start btn-sm pl-1"
        on:click={(event) => {
          updateSelectedItemStyles({ styleKey: 'fontSize', styleValue: 'text-7xl' });
        }}
        >Heading 1
      </button>
      <button
        class="btn btn-ghost btn-block text-left justify-start btn-sm pl-1"
        on:click={() => {
          updateSelectedItemStyles({ styleKey: 'fontSize', styleValue: 'text-3xl' });
        }}>Heading 2</button
      >
      <button
        class="btn btn-ghost btn-block text-left justify-start btn-sm pl-1"
        on:click={() => {
          updateSelectedItemStyles({ styleKey: 'fontSize', styleValue: 'text-md' });
        }}>Body</button
      >
      <button
        class="btn btn-ghost btn-block text-left justify-start btn-sm pl-1"
        on:click={() => {
          updateSelectedItemStyles({ styleKey: 'fontSize', styleValue: 'text-sm' });
        }}>Meta</button
      >
    </div>
  </div>
{/if}

<style>
  :global(.svlt-grid-item) {
    height: unset !important;
  }
  :global(.svlt-grid-shadow) {
    background: oklch(var(--bc)) !important;
    opacity: 0.4;
  }
</style>
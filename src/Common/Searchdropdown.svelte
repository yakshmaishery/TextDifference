<script lang="ts">
   export let selectedValue = ""
   export let listdata:{value:string,label:string}[] = []
  import CheckIcon from "@lucide/svelte/icons/check";
  import ChevronsUpDownIcon from "@lucide/svelte/icons/chevrons-up-down";
  import { tick } from "svelte";
  import * as Command from "$lib/components/ui/command/index.js";
  import * as Popover from "$lib/components/ui/popover/index.js";
  import { Button } from "$lib/components/ui/button/index.js";
  import { cn } from "$lib/utils.js";
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();


  let open = false;
  let value = "";


//   const selectedValue = $derived(
//     frameworks.find((f) => f.value === value)?.label
//   );

  function closeAndFocusTrigger() {
   selectedValue = (listdata.find((f) => f.value === value)?.label || "")
   open = false;
   dispatch("updateLanguage")
  }
</script>

<Popover.Root bind:open>
  <Popover.Trigger>
    <Button variant="outline" class="w-[200px] justify-between" role="combobox" aria-expanded={open}>
      {selectedValue || "Select a framework..."}
      <ChevronsUpDownIcon class="opacity-50" />
    </Button>
  </Popover.Trigger>
  <Popover.Content class="w-[200px] p-0">
    <Command.Root>
      <Command.Input placeholder="Search framework..." />
      <Command.List>
        <Command.Empty>No Data found.</Command.Empty>
        <Command.Group>
          {#each listdata as items (items.value)}
            <Command.Item
              value={items.value}
              onSelect={() => {
                value = items.value;
                closeAndFocusTrigger();
              }}
            >
              <CheckIcon class={cn(value !== items.value && "text-transparent")} />
              {items.label}
            </Command.Item>
          {/each}
        </Command.Group>
      </Command.List>
    </Command.Root>
  </Popover.Content>
</Popover.Root>

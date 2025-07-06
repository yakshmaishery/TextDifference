<script lang="ts">
   export let LeftText:string = ""
   export let RightText:string = ""
   import * as Card from "$lib/components/ui/card/index.js";
   import { Textarea } from "$lib/components/ui/textarea/index.js";
   import '$lib/Styles/InputCardCSS.css'
</script>

<div class="p-4">
   <Card.Root>
      <Card.Header>
         <!-- <Card.Title>Input</Card.Title> -->
         <!-- <Card.Description>Card Description</Card.Description> -->
      </Card.Header>

      <Card.Content class="flex gap-4">
         <Textarea placeholder="Old text..." class="w-1/2 h-20" bind:value={LeftText} />
         <Textarea placeholder="New text..." class="w-1/2 h-20" bind:value={RightText} />
      </Card.Content>

      <Card.Footer>
         <!-- <p>Card Footer</p> -->
      </Card.Footer>
   </Card.Root>
</div>

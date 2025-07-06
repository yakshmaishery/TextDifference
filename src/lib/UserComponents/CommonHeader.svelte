<script lang="ts">
   import SunIcon from "@lucide/svelte/icons/sun";
   import MoonIcon from "@lucide/svelte/icons/moon";
   import { toggleMode } from "mode-watcher";
   import { Button } from "$lib/components/ui/button/index.js";
</script>
<nav class="border-b shadow-sm px-4 py-3 flex justify-between items-center">
  <div class="text-xl font-semibold">Text Difference</div>
  <Button onclick={toggleMode} variant="outline" size="icon">
      <SunIcon class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 !transition-all dark:-rotate-90 dark:scale-0"/>
      <MoonIcon class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 !transition-all dark:rotate-0 dark:scale-100"/>
      <span class="sr-only">Toggle theme</span>
   </Button>
</nav>

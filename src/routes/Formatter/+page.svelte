<script lang="ts">
    import CommonHeader from "$lib/UserComponents/CommonHeader.svelte";
    import {listdata} from '$lib/Environment'
    import Searchdropdown from "../../Common/Searchdropdown.svelte";
    import * as monaco from "monaco-editor";
    import { onMount, onDestroy } from "svelte";
    import * as Select from "$lib/components/ui/select/index.js";
    let selectedLanguage = "none";
    let editor1El: HTMLDivElement;
    let editor1: monaco.editor.IStandaloneCodeEditor | null = null;
    let selectedTheme = "vs-dark";
    const themes = ["vs-dark", "vs-light", "hc-black", "hc-light"];
    let wordWrap = false

    onMount(async () => {
      editor1 = monaco.editor.create(editor1El, {
         value: "",
         language: selectedLanguage === "none" ? undefined : selectedLanguage, // Use undefined for 'none'
         theme: selectedTheme,
      });

      // Initial layout fix
      setTimeout(() => {
         editor1?.layout();
      }, 10);

      window.addEventListener("resize", () => {
         editor1?.layout();
      });
   });

   onDestroy(() => {
      editor1?.dispose();
   });

    function updateLanguage() {
      if (editor1) {
         monaco.editor.setModelLanguage(
            editor1.getModel()!,
            selectedLanguage === "none" ? "plaintext" : selectedLanguage,
         );
      }
   }

   function updateTheme() {
      monaco.editor.setTheme(selectedTheme);
   }
</script>
<CommonHeader />
<div class="m-4">
   <div class="flex items-center gap-4 mb-4">
      <div>
         <Searchdropdown listdata={listdata} bind:selectedValue={selectedLanguage} on:updateLanguage={updateLanguage}/>
      </div>
      <div>
         <Select.Root type="single" bind:value={selectedTheme} onValueChange={updateTheme}>
            <Select.Trigger class="w-[180px]">{selectedTheme}</Select.Trigger>
            <Select.Content>
               {#each themes as theme}
                  <Select.Item value={theme}>{theme}</Select.Item>
               {/each}
            </Select.Content>
         </Select.Root>
      </div>
   </div>
   <div class="grid grid-cols-1 gap-4 mb-4" style="height: 70vh;">
      <div bind:this={editor1El} class="border rounded min-w-0"></div>
   </div>
</div>
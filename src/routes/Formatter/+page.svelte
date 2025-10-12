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
   const filechange = async (e:any) =>{
      if(e){
         if(e.target){
            if(e.target.files){
               if(e.target.files.length>0){
                  // if(field == "1"){
                     const file = e.target.files[0];
                     if (file) {
                        const reader = new FileReader();

                        reader.onload = function(e1:any) {
                           const content = e1.target.result;
                              if (editor1) {
                                 editor1.setValue(content);
                              }
                        };

                        reader.onerror = function(e) {
                           console.error('Error reading file:', e);
                        };

                        reader.readAsText(file); // You can also use readAsDataURL, readAsArrayBuffer, etc.
                     }
                  // }
               }
            }
         }
      }
   }

   // --- Clear Function ---
   function clearEditors() {
      if (editor1) {
         editor1.setValue("");
      }
      let file1:any = document.getElementById("file1")
      if(file1){
         file1.value = ""
      }
   }
</script>
<CommonHeader />
<div class="m-4">
   <div class="flex items-center gap-4 mb-4">
      <div>
         <input type="file" id="file1" on:change={(e)=>{filechange(e)}} class="bg-primary text-primary-foreground hover:bg-primary/90 px-4 py-2 rounded-md" />
      </div>
      <div>
         <button on:click={clearEditors} class="bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700">
            Clear All
         </button>
      </div>
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
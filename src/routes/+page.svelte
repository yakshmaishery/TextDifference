<script lang="ts">
   import CommonHeader from "$lib/UserComponents/CommonHeader.svelte";
   import * as Select from "$lib/components/ui/select/index.js";
   import { onMount, onDestroy } from "svelte";
   import "$lib/monaco";
   import * as monaco from "monaco-editor";
   import { Checkbox } from "$lib/components/ui/checkbox/index.js";
   import { Label } from "$lib/components/ui/label/index.js";
   import Searchdropdown from "../Common/Searchdropdown.svelte";
   import {listdata} from '$lib/Environment'

   let editor1El: HTMLDivElement;
   let editor2El: HTMLDivElement;
   let diffEditorEl: HTMLDivElement;

   let editor1: monaco.editor.IStandaloneCodeEditor | null = null;
   let editor2: monaco.editor.IStandaloneCodeEditor | null = null;
   let diffEditor: monaco.editor.IStandaloneDiffEditor | null = null;

   let selectedLanguage = "none";
   let selectedTheme = "vs-dark";

   const themes = ["vs-dark", "vs-light", "hc-black", "hc-light"];

   let wordWrap = false
   let DisableError = false

   onMount(async () => {
      editor1 = monaco.editor.create(editor1El, {
         value: "",
         language: selectedLanguage === "none" ? undefined : selectedLanguage, // Use undefined for 'none'
         theme: selectedTheme,
         stickyScroll: { enabled: false },
      });

      editor2 = monaco.editor.create(editor2El, {
         value: "",
         language: selectedLanguage === "none" ? undefined : selectedLanguage, // Use undefined for 'none'
         theme: selectedTheme,
         stickyScroll: { enabled: false },
      });

      // Initial layout fix
      setTimeout(() => {
         editor1?.layout();
         editor2?.layout();
      }, 10);

      window.addEventListener("resize", () => {
         editor1?.layout();
         editor2?.layout();
         diffEditor?.layout(); // Use optional chaining
      });
   });

   onDestroy(() => {
      editor1?.dispose();
      editor2?.dispose();
      diffEditor?.dispose();
   });

   function updateTheme() {
      monaco.editor.setTheme(selectedTheme);
   }

   function updateLanguage() {
      if (editor1) {
         monaco.editor.setModelLanguage(
            editor1.getModel()!,
            selectedLanguage === "none" ? "plaintext" : selectedLanguage,
         );
      }
      if (editor2) {
         monaco.editor.setModelLanguage(
            editor2.getModel()!,
            selectedLanguage === "none" ? "plaintext" : selectedLanguage,
         );
      }
   }

   function compareTexts() {
      const originalCode = editor1?.getValue() || "";
      const modifiedCode = editor2?.getValue() || "";

      if (diffEditor) {
         const oldModels = diffEditor.getModel();
         if (oldModels) {
            oldModels.original.dispose();
            oldModels.modified.dispose();
         }
         diffEditor.dispose(); // Dispose the existing diff editor instance
         diffEditor = null; // Clear the reference
      }

      // Create new models
      const originalModel = monaco.editor.createModel(
         originalCode,
         selectedLanguage === "none" ? "plaintext" : selectedLanguage,
      );
      const modifiedModel = monaco.editor.createModel(
         modifiedCode,
         selectedLanguage === "none" ? "plaintext" : selectedLanguage,
      );

      if(wordWrap == false){
         // Create a new diff editor instance
         diffEditor = monaco.editor.createDiffEditor(diffEditorEl, {
            theme: selectedTheme,
            renderSideBySide: true,
            wordWrap:"off",
            stickyScroll: { enabled: false },
         });
      }
      else{
         // Create a new diff editor instance
         diffEditor = monaco.editor.createDiffEditor(diffEditorEl, {
            theme: selectedTheme,
            renderSideBySide: true,
            wordWrap:"on",
            stickyScroll: { enabled: false },
         });
      }

      // Set the new models
      diffEditor.setModel({
         original: originalModel,
         modified: modifiedModel,
      });

      setTimeout(() => diffEditor?.layout(), 0);
   }

   // --- Clear Function ---
   function clearEditors() {
      if (editor1) {
         editor1.setValue("");
      }
      if (editor2) {
         editor2.setValue("");
      }

      // Safely dispose the diff editor and its models
      if (diffEditor) {
         // Then dispose of the diff editor instance
         diffEditor.dispose();
         diffEditor = null; // Clear the reference so it can be recreated
      }
      let file1:any = document.getElementById("file1")
      if(file1){
         file1.value = ""
      }
      let file2:any = document.getElementById("file2")
      if(file2){
         file2.value = ""
      }
   }
   const filechange = async (e:any,field:string) =>{
      console.warn(e.target.files)
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
                           if(field == "1"){
                              if (editor1) {
                                 editor1.setValue(content);
                              }
                           }
                           if(field == "2"){
                              if (editor2) {
                                 editor2.setValue(content);
                              }
                           }
                        };

                        reader.onerror = function(e) {
                           console.error('Error reading file:', e);
                        };

                        if(selectedLanguage == "base64"){
                        reader.readAsDataURL(file);
                        }
                        else{
                           reader.readAsText(file); // You can also use readAsDataURL, readAsArrayBuffer, etc.
                        }
                     }
                  // }
               }
            }
         }
      }
   }

   const disableErrorsfunc = () => {
      DisableError = !DisableError
      monaco.languages.typescript.typescriptDefaults.setDiagnosticsOptions({
         noSemanticValidation: DisableError,
         noSyntaxValidation: DisableError,
      });
      updateLanguage()
   }
</script>

<CommonHeader />
<div class="m-4">
   <div class="flex items-center gap-4 mb-4">
      <div>
         <Searchdropdown listdata={listdata} bind:selectedValue={selectedLanguage} on:updateLanguage={updateLanguage}/>
      </div>

      <div>
         <Select.Root
            type="single"
            bind:value={selectedTheme}
            onValueChange={updateTheme}
         >
            <Select.Trigger class="w-[180px]">{selectedTheme}</Select.Trigger>
            <Select.Content>
               {#each themes as theme}
                  <Select.Item value={theme}>{theme}</Select.Item>
               {/each}
            </Select.Content>
         </Select.Root>
      </div>
   </div>

   <div class="grid grid-cols-2 gap-4 mb-4">
      <div>
         <input type="file" id="file1" on:change={(e)=>{filechange(e,"1")}} class="bg-primary text-primary-foreground hover:bg-primary/90 px-4 py-2 rounded-md" />
      </div>
      <div>
         <input type="file" id="file2" on:change={(e)=>{filechange(e,"2")}} class="bg-primary text-primary-foreground hover:bg-primary/90 px-4 py-2 rounded-md" />
      </div>
   </div>
   <div class="grid grid-cols-2 gap-4 mb-4" style="height: 300px;">
      <div bind:this={editor1El} class="border rounded min-w-0"></div>
      <div bind:this={editor2El} class="border rounded min-w-0"></div>
   </div>

   <div class="flex gap-4 mb-4">
      <button on:click={compareTexts} class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">
         Compare
      </button>

      <button on:click={clearEditors} class="bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700">
         Clear All
      </button>

      <Label class="hover:bg-accent/50 flex items-start gap-3 rounded-lg border p-3 has-[[aria-checked=true]]:border-blue-600 has-[[aria-checked=true]]:bg-blue-50 dark:has-[[aria-checked=true]]:border-blue-900 dark:has-[[aria-checked=true]]:bg-blue-950" style="height:45px">
            <Checkbox
            checked={wordWrap}
            onCheckedChange={()=>{wordWrap = !wordWrap}}
               id="toggle-2"
               class="data-[state=checked]:border-blue-600 data-[state=checked]:bg-blue-600 data-[state=checked]:text-white dark:data-[state=checked]:border-blue-700 dark:data-[state=checked]:bg-blue-700"
            />
            <div class="grid font-normal">
               <p class="text-sm font-medium leading-none">Word Wrap</p>
            </div>
         </Label>

         <Label class="hover:bg-accent/50 flex items-start gap-3 rounded-lg border p-3 has-[[aria-checked=true]]:border-blue-600 has-[[aria-checked=true]]:bg-blue-50 dark:has-[[aria-checked=true]]:border-blue-900 dark:has-[[aria-checked=true]]:bg-blue-950" style="height:45px">
            <Checkbox
            checked={DisableError}
            onCheckedChange={()=>{disableErrorsfunc()}}
               id="toggle-2"
               class="data-[state=checked]:border-blue-600 data-[state=checked]:bg-blue-600 data-[state=checked]:text-white dark:data-[state=checked]:border-blue-700 dark:data-[state=checked]:bg-blue-700"
            />
            <div class="grid font-normal">
               <p class="text-sm font-medium leading-none">Disable Error</p>
            </div>
         </Label>
   </div>
   <!-- <div style="width: fit-content;">
         <Label class="hover:bg-accent/50 flex items-start gap-3 rounded-lg border p-3 has-[[aria-checked=true]]:border-blue-600 has-[[aria-checked=true]]:bg-blue-50 dark:has-[[aria-checked=true]]:border-blue-900 dark:has-[[aria-checked=true]]:bg-blue-950" style="height:45px">
            <Checkbox
            checked={wordWrap}
            onCheckedChange={()=>{wordWrap = !wordWrap}}
               id="toggle-2"
               class="data-[state=checked]:border-blue-600 data-[state=checked]:bg-blue-600 data-[state=checked]:text-white dark:data-[state=checked]:border-blue-700 dark:data-[state=checked]:bg-blue-700"
            />
            <div class="grid font-normal">
               <p class="text-sm font-medium leading-none">Word Wrap</p>
            </div>
         </Label>
   </div> -->

   <div
      class="mt-4 border rounded"
      bind:this={diffEditorEl}
      style="height: 500px;"
   ></div>
</div>

<style>
   div {
      min-height: 50px;
   }
</style>

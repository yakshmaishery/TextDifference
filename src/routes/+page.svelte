<script lang="ts">
   import CommonHeader from "$lib/UserComponents/CommonHeader.svelte";
   import * as Select from "$lib/components/ui/select/index.js";
   import { onMount, onDestroy } from "svelte";
   import "$lib/monaco";
   import * as monaco from "monaco-editor";

   let editor1El: HTMLDivElement;
   let editor2El: HTMLDivElement;
   let diffEditorEl: HTMLDivElement;

   let editor1: monaco.editor.IStandaloneCodeEditor | null = null;
   let editor2: monaco.editor.IStandaloneCodeEditor | null = null;
   let diffEditor: monaco.editor.IStandaloneDiffEditor | null = null;

   let selectedLanguage = "none";
   let selectedTheme = "vs-dark";

   const languages = [
      "none",
      "javascript",
      "typescript",
      "html",
      "css",
      "json",
      "sql"
   ];
   const themes = ["vs-dark", "vs-light", "hc-black"];

   onMount(async () => {
      editor1 = monaco.editor.create(editor1El, {
         value: "",
         language: selectedLanguage === "none" ? undefined : selectedLanguage, // Use undefined for 'none'
         theme: selectedTheme,
      });

      editor2 = monaco.editor.create(editor2El, {
         value: "",
         language: selectedLanguage === "none" ? undefined : selectedLanguage, // Use undefined for 'none'
         theme: selectedTheme,
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
      // It's also good practice to dispose of any models created *outside* an editor
      // if they are not explicitly set to `null` or re-assigned.
      // For models created by `createModel` for the diff editor, they are often disposed
      // when a new model is set or the diff editor is disposed.
      // However, if you have other floating models, this can be useful:
      // monaco.editor.getModels().forEach(model => model.dispose());
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
      // For the diff editor, if it exists, you might want to re-run compareTexts
      // to update the language of its models, as `setModelLanguage` doesn't apply
      // to diff editor's internal models. Or, dispose and recreate.
      // For simplicity, we'll assume a language change prompts a re-compare.
      if (diffEditor) {
         // Option 1: Re-run compareTexts to update diff editor's language
         // compareTexts(); // This will recreate models with new language
         // Option 2: If just changing the UI language for syntax, but content is same.
         // The language of diff editor models is set during their creation.
         // You generally recreate the models for a language change.
      }
   }

   function compareTexts() {
      const originalCode = editor1?.getValue() || "";
      const modifiedCode = editor2?.getValue() || "";

      // IMPORTANT: Safely dispose of previous models BEFORE creating new ones
      // and BEFORE setting them on the diff editor.
      if (diffEditor) {
         const oldModels = diffEditor.getModel();
         if (oldModels) {
            oldModels.original.dispose();
            oldModels.modified.dispose();
         }
         // If you don't dispose the diff editor, just set new models
         // We are choosing to recreate it each time for simplicity in this example,
         // but if you want to keep the diffEditor instance, uncomment the dispose above
         // and just call setModel below.
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

      // Create a new diff editor instance
      diffEditor = monaco.editor.createDiffEditor(diffEditorEl, {
         theme: selectedTheme,
         renderSideBySide: true,
         // Optional: Add options for the diff editor itself
         // enableSplitViewResizing: true,
         // ignoreTrimWhitespace: true,
      });

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
         const oldModels = diffEditor.getModel();
         if (oldModels) {
            // Dispose of the models first
            oldModels.original.dispose();
            oldModels.modified.dispose();
         }
         // Then dispose of the diff editor instance
         diffEditor.dispose();
         diffEditor = null; // Clear the reference so it can be recreated
      }
   }
</script>

<CommonHeader />
<div class="m-4">
   <div class="flex items-center gap-4 mb-4">
      <div>
         <Select.Root
            type="single"
            bind:value={selectedLanguage}
            onValueChange={updateLanguage}
         >
            <Select.Trigger class="w-[180px]"
               >{selectedLanguage === "none"
                  ? "Select Language"
                  : selectedLanguage}</Select.Trigger
            >
            <Select.Content>
               {#each languages as lang}
                  <Select.Item value={lang}>{lang}</Select.Item>
               {/each}
            </Select.Content>
         </Select.Root>
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

   <div class="grid grid-cols-2 gap-4 mb-4" style="height: 300px;">
      <div bind:this={editor1El} class="border rounded min-w-0"></div>
      <div bind:this={editor2El} class="border rounded min-w-0"></div>
   </div>

   <div class="flex gap-4 mb-4">
      <button
         on:click={compareTexts}
         class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
      >
         Compare
      </button>

      <button
         on:click={clearEditors}
         class="bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700"
      >
         Clear All
      </button>
   </div>

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

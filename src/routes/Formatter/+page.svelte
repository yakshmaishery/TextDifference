<script lang="ts">
   export let PageloadsidebarOpen = true;
   import '$lib/monaco'
   import CommonHeader from "$lib/UserComponents/CommonHeader.svelte";
   import * as InputGroup from "$lib/components/ui/input-group/index.js";
   import { Label } from "$lib/components/ui/label/index.js";
   import { onMount, onDestroy } from "svelte";
   import * as Select from "$lib/components/ui/select/index.js";
   import * as monaco from "monaco-editor";
   import { Upload,RefreshCwIcon,CornerDownLeftIcon,CopyIcon,Presentation } from "@lucide/svelte";
   import { listdata } from "$lib/Environment";
   import Checkbox from '$lib/components/ui/checkbox/checkbox.svelte';
   import { Input } from "$lib/components/ui/input/index.js";
   import Searchdropdown from '../../Common/Searchdropdown.svelte';
   import { toast } from "svelte-sonner";
   let selectedLanguage = "none";
   let cardhtml: HTMLDivElement;
   let editorWrapper: HTMLDivElement;
   let editor1: monaco.editor.IStandaloneCodeEditor | null = null;
   let selectedTheme = "vs-dark";
   selectedLanguage = listdata[0].value;
   const themes = ["vs-dark", "vs-light", "hc-black", "hc-light"];
   let wordWrap = false;
   let cardhtmlWidth: number = 0;
   let downloadFileName:string = ""
   let downloadFileType:string = ""
   let ShowPreviewList = ["base64","html"]

   $:if(selectedLanguage){updateLanguage()}
   $:if(wordWrap){
      editor1?.updateOptions({
         wordWrap: 'on'
      });
   }

   $:if(!wordWrap){
      editor1?.updateOptions({
         wordWrap: 'off'
      });
   }

   $: if (PageloadsidebarOpen) {
      if (editor1) {
         // alert(cardhtml.clientWidth)
         editorWrapper.setAttribute("style", `width:${cardhtmlWidth - 2}px`);
         requestAnimationFrame(() => editor1?.layout());
      }
   }
   $: if (!PageloadsidebarOpen) {
      if (editor1) {
         // alert(cardhtml.clientWidth)
         cardhtmlWidth = cardhtml.clientWidth;
         editorWrapper.setAttribute("style", "width:100%");
         requestAnimationFrame(() => editor1?.layout());
      }
   }
   $: editor1?.layout();

   onMount(() => {
      editor1 = monaco.editor.create(editorWrapper, {
         value: "",
         language: selectedLanguage === "none" ? undefined : selectedLanguage, // Use undefined for 'none'
         theme: selectedTheme,
         stickyScroll: { enabled: false },
      });

      const ro = new ResizeObserver(() => {
         requestAnimationFrame(() => editor1?.layout());
      });

      // Observe card + editor wrapper
      ro.observe(cardhtml);
      ro.observe(editorWrapper);

      window.addEventListener("resize", () => editor1?.layout());

      // setTimeout(() => editor1?.layout(), 50);
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

   const filechange = async (e: any) => {
      if (e) {
         if (e.target) {
            if (e.target.files) {
               if (e.target.files.length > 0) {
                  // if(field == "1"){
                  const file = e.target.files[0];
                  if (file) {
                     downloadFileName = file.name
                     downloadFileType = file.type
                     if(downloadFileType == "text/plain"){
                        selectedLanguage = listdata[0].value;
                     }
                     else{
                        const result = listdata.find(obj => obj.mimetype == downloadFileType);
                        if(result){
                           selectedLanguage = result.value
                        }
                     }
                     const reader = new FileReader();

                     reader.onload = function (e1: any) {
                        const content = e1.target.result;
                        if (editor1) {
                           editor1.setValue(content);
                        }
                     };

                     reader.onerror = function (e) {
                        console.error("Error reading file:", e);
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
   };

   // --- Clear Function ---
   function clearEditors() {
      if (editor1) {
         editor1.setValue("");
         downloadFileName = ""
         // selectedLanguage = listdata[0].value;
      }
      let file1: any = document.getElementById("editorfile");
      if (file1) {
         file1.value = "";
      }
   }

   const downloadFile = () => {
      // Simplest approach: Always use text/plain
      const textContent = editor1?.getValue();
      if(textContent){
         const blob = new Blob([textContent], { type: 'text/plain' }); 
         // ... rest of the download function (creates <a>, sets download="my_data.custom_ext", clicks)
         // --- Standard download procedure ---
         const url = URL.createObjectURL(blob);
         const a = document.createElement('a');

         a.download = downloadFileName;
         a.href = url;

         document.body.appendChild(a);
         a.click();

         // Clean up
         document.body.removeChild(a);
         URL.revokeObjectURL(url);
      }
      else{
         toast.info("There is no data!")
      }
   }

   const CopytoClipboard = () => {
      const textContent = editor1?.getValue() || "";
      navigator.clipboard.writeText(textContent)
      toast.success("Copied successfully!")
   }

   function isBase64Image(base64:string) {
      if (!base64.startsWith("data:")) return false;

      const mime = base64.substring(
         base64.indexOf(":") + 1,
         base64.indexOf(";")
      );

      return mime.startsWith("image/");
   }

   const PreviewData = () => {
      const textContent = editor1?.getValue() || "";
      if(textContent){
         // const blob = new Blob([textContent], { type: 'text/html' }); 
         // const url = URL.createObjectURL(blob);
         // window.open(url,"_blank")
         let newtab = window.open("","_blank")
         let htmlcode = ""
         if(selectedLanguage == "base64"){
            // htmlcode = `<html>
            //    <embed src="${textContent}" style="height:100%;width:100%"/>
            // </html>`
            if(isBase64Image(textContent)){
               htmlcode = `<html>
                  <div style="width:100%;height:100%;display:flex;align-items:center;justify-content:center">
                     <embed src="${textContent}" style="max-height:100%;max-width:100%;height:100%;width:auto" />
                  </div>
               </html>`
               newtab?.document.write(htmlcode)
               newtab?.document.close()
            }
            else{
               htmlcode = `<html>
                  <div style="width:100%;height:100%;display:flex;align-items:center;justify-content:center">
                     <embed src="${textContent}" style="max-height:100%;max-width:100%;height:100%;width:100%" />
                  </div>
               </html>`
               newtab?.document.write(htmlcode)
               newtab?.document.close()
            }
         }
         else{
            htmlcode = textContent
            newtab?.document.write(htmlcode)
            newtab?.document.close()
         }
      }
   }
</script>

<CommonHeader />
<div
   class="grid w-full"
   style="height: 80vh;max-height: 80vh;"
   bind:this={cardhtml}
>
   <InputGroup.Root>
      <InputGroup.Addon align="block-start" class="border-b">
         <InputGroup.Text class="font-mono font-medium">
            <!-- script.js -->
         </InputGroup.Text>
         <!-- svelte-ignore a11y_consider_explicit_label -->
          <div class="grid gap-3">
            <Searchdropdown listdata={listdata} bind:selectedValue={selectedLanguage}/>
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
         <button
            class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive shrink-0 justify-center whitespace-nowrap font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 hover:bg-accent hover:text-accent-foreground dark:hover:bg-accent/50 flex items-center gap-2 text-sm shadow-none size-6 rounded-[calc(var(--radius)-5px)] p-0 has-[>svg]:p-0"
            style="cursor: pointer;"
            on:click={() => {document.getElementById("editorfile")?.click()}}
         >
            <Upload />
         </button>
         <!-- <InputGroup.Button class="ms-auto" size="icon-xs">
            <RefreshCwIcon />
            </InputGroup.Button> -->
         <button
            style="cursor: pointer;"
            class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive shrink-0 justify-center whitespace-nowrap font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 hover:bg-accent hover:text-accent-foreground dark:hover:bg-accent/50 flex items-center gap-2 text-sm shadow-none size-6 rounded-[calc(var(--radius)-5px)] p-0 has-[>svg]:p-0 ms-auto"
            on:click={clearEditors}
         >
            <RefreshCwIcon />
         </button>
         <button
            class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive shrink-0 justify-center whitespace-nowrap font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 hover:bg-accent hover:text-accent-foreground dark:hover:bg-accent/50 flex items-center gap-2 text-sm shadow-none size-6 rounded-[calc(var(--radius)-5px)] p-0 has-[>svg]:p-0"
            style="cursor: pointer;"
            on:click={CopytoClipboard}
         >
            <CopyIcon />
         </button>
         {#if ShowPreviewList.includes(selectedLanguage)}
            <button
               class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive shrink-0 justify-center whitespace-nowrap font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 hover:bg-accent hover:text-accent-foreground dark:hover:bg-accent/50 flex items-center gap-2 text-sm shadow-none size-6 rounded-[calc(var(--radius)-5px)] p-0 has-[>svg]:p-0"
               style="cursor: pointer;"
               on:click={PreviewData}
            >
               <Presentation />
            </button>
         {/if}
      </InputGroup.Addon>
      <!-- <InputGroup.Textarea style="max-height:70vh"
         placeholder=""
         class="editor-area"
      /> -->
      <div
         bind:this={editorWrapper}
         class="border rounded min-w-0 w-full h-full"
         style=""
      ></div>

      <InputGroup.Addon align="block-end" class="border-t">
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
         <div class="grid gap-3">
         <Input
            id="AnotherID"
            type="text"
            class="w-50"
            placeholder="FileName..."
            bind:value={downloadFileName}
            autocomplete="off"
         />
         </div>
         <button class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive shrink-0 justify-center whitespace-nowrap font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 bg-primary text-primary-foreground hover:bg-primary/90 py-2 flex items-center text-sm shadow-none h-8 gap-1.5 rounded-md px-2.5 has-[>svg]:px-2.5 ms-auto" style="cursor: pointer;" on:click={downloadFile}>
            Download <CornerDownLeftIcon />
         </button>
      </InputGroup.Addon>
   </InputGroup.Root>
</div>
<input type="file" hidden on:input={(e)=>{filechange(e)}} id="editorfile"/>
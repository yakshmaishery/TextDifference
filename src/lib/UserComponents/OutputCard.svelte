<script lang="ts">
   export let LeftText: string = "";
   export let RightText: string = "";
   import * as Tabs from "$lib/components/ui/tabs/index.js";
   import * as Card from "$lib/components/ui/card/index.js";
   import { ArrowBigUp, ArrowBigDown } from "@lucide/svelte";
   import "$lib/Styles/OutputCardCSS.css";
   import { diffLines, diffWords } from "diff";
   let diffs: { line: number; html: string; modifiedLine: string }[] = [];
   let modificationLines: number[] = [];
   let Currentposition: number = 0;
   function escapeHtml(str: string): string {
      return str
         .replace(/&/g, "&amp;")
         .replace(/</g, "&lt;")
         .replace(/>/g, "&gt;");
   }

   function compareText() {
      const lineDiff = diffLines(LeftText, RightText);
      diffs = [];
      modificationLines = [];
      let currentLine = 1;

      for (let i = 0; i < lineDiff.length; i++) {
         const part = lineDiff[i];
         const lines = part.value.split("\n");

         for (let j = 0; j < lines.length; j++) {
            if (
               j === lines.length - 1 &&
               lines[j] === "" &&
               part.value.endsWith("\n")
            )
               continue;

            const line = lines[j];

            // Look ahead to see if it's a removed line followed by an added line (i.e., a modified line)
            const next = lineDiff[i + 1];
            const nextLines = next?.value?.split("\n") ?? [];
            const nextLine = nextLines[j] ?? "";

            const isModifiedLine =
               part.removed && next?.added && nextLines.length === lines.length;

            if (isModifiedLine) {
               const wordDiff = diffWords(line, nextLine);
               let html = wordDiff
                  .map((d) => {
                     const color = d.added
                        ? "bg-green-500 text-black-500"
                        : d.removed
                          ? "bg-red-500 line-through text-black-500"
                          : "";
                     return `<span class="${color}">${escapeHtml(d.value)}</span>`;
                  })
                  .join("");

               diffs.push({ line: currentLine, html: html, modifiedLine: "1" });
               if (!modificationLines.includes(currentLine)) {
                  modificationLines.push(currentLine);
               }
               i++; // skip the "added" part we just used
            } else if (part.added || part.removed) {
               // Pure added or removed line
               const color = part.added
                  ? "bg-green-500 text-black-500"
                  : "bg-red-500 line-through text-black-500";

               const visible =
                  line === ""
                     ? '<em class="text-gray-400">[blank line]</em>'
                     : escapeHtml(line);

               diffs.push({
                  line: currentLine,
                  html: `<span class="${color}">${visible}</span>`,
                  modifiedLine: "1",
               });
               if (!modificationLines.includes(currentLine)) {
                  modificationLines.push(currentLine);
               }
            } else {
               // Unchanged line
               const visible =
                  line === ""
                     ? '<em class="text-gray-400">[blank line]</em>'
                     : escapeHtml(line);

               diffs.push({
                  line: currentLine,
                  html: `<span>${visible}</span>`,
                  modifiedLine: "0",
               });
            }

            currentLine++;
         }
      }
      console.warn(modificationLines);
   }

   function clearAll() {
      LeftText = "";
      RightText = "";
      diffs = [];
      modificationLines = [];
   }

   const goPosition = (flag: string) => {
      if (modificationLines.length === 0) return;

      if (flag === "down") {
         Currentposition = (Currentposition + 1) % modificationLines.length;
      } else if (flag === "up") {
         Currentposition =
            (Currentposition - 1 + modificationLines.length) %
            modificationLines.length;
      }

      const ID = modificationLines[Currentposition];
      const div = document.getElementById(`${ID}`);

      if (div) {
         div.scrollIntoView({ behavior: "smooth", block: "center" });
      }
   };
</script>

<div class="p-4">
   <Card.Root>
      <Card.Header>
         <!-- <Card.Title>Input</Card.Title> -->
         <!-- <Card.Description>Card Description</Card.Description> -->
         <div style="width: 10%;display:flex;gap:20px">
            <button
               on:click={compareText}
               class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive inline-flex shrink-0 items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 bg-background shadow-xs hover:bg-accent hover:text-accent-foreground dark:bg-input/30 dark:border-input dark:hover:bg-input/50 border h-9 px-4 py-2 has-[>svg]:px-3 w-full"
               style="cursor:pointer"
               disabled={LeftText == "" || RightText == ""}>Compare</button
            >
            <button
               on:click={clearAll}
               class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive inline-flex shrink-0 items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 bg-background shadow-xs hover:bg-accent hover:text-accent-foreground dark:bg-input/30 dark:border-input dark:hover:bg-input/50 border h-9 px-4 py-2 has-[>svg]:px-3 w-full"
               style="cursor:pointer">Clear</button
            >
         </div>
      </Card.Header>

      <Card.Content class="flex gap-4" style="overflow-x:auto">
         <Tabs.Root value="Difference" class="w-full">
            <Tabs.List>
               <Tabs.Trigger value="Difference"
                  >Difference in output</Tabs.Trigger
               >
               <Tabs.Trigger value="Details">Details</Tabs.Trigger>
            </Tabs.List>
            <Tabs.Content value="Difference">
               <div
                  class="p-4 space-y-2"
                  style="display: grid;grid-template-columns: max-content auto;gap:10px"
               >
                  {#each diffs as diff}
                     <!-- <div class="flex items-start"> -->
                     <div
                        id={`${diff.line}`}
                        class={`w-8 text-right pr-2 bg-gray-20 ${diff.modifiedLine == "1" ? "text-red-500" : "text-gray-500"}`}
                     >
                        {diff.line}
                     </div>
                     <pre
                        class="flex-1 whitespace-pre-wrap break-words">{@html diff.html}</pre>
                     <!-- </div> -->
                  {/each}
               </div>
            </Tabs.Content>
            <Tabs.Content value="Details">
               <div class="p-4 space-y-2">
                  <Card.Title>Modification Lines</Card.Title>
                  <Card.Description>{modificationLines}</Card.Description>
               </div>
            </Tabs.Content>
         </Tabs.Root>
      </Card.Content>

      <Card.Footer>
         <!-- <p>Card Footer</p> -->
      </Card.Footer>
   </Card.Root>
</div>
{#if modificationLines.length > 0}
   <div id="navdiv">
      <button
         on:click={() => {
            goPosition("up");
         }}
         class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive inline-flex shrink-0 items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 bg-background shadow-xs hover:bg-accent hover:text-accent-foreground dark:bg-input/30 dark:border-input dark:hover:bg-input/50 border h-9 px-4 py-2 has-[>svg]:px-3 w-full"
         style="cursor:pointer"><ArrowBigUp /></button
      >
      <button
         on:click={() => {
            goPosition("down");
         }}
         class="focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive inline-flex shrink-0 items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium outline-none transition-all focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0 bg-background shadow-xs hover:bg-accent hover:text-accent-foreground dark:bg-input/30 dark:border-input dark:hover:bg-input/50 border h-9 px-4 py-2 has-[>svg]:px-3 w-full"
         style="cursor:pointer"><ArrowBigDown /></button
      >
   </div>
{/if}

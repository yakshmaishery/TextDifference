<script lang="ts">
   import CommonHeader from "$lib/UserComponents/CommonHeader.svelte";
   import InputCard from "$lib/UserComponents/InputCard.svelte";
   import OutputCard from "$lib/UserComponents/OutputCard.svelte";
   let LeftText:string = ""
   let RightText:string = ""
</script>
<svelte:head>
   <title>Text Difference</title>
</svelte:head>
<CommonHeader/>
<InputCard bind:LeftText bind:RightText/>
<OutputCard bind:LeftText bind:RightText/>
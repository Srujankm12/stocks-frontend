<script>
    import { goto } from "$app/navigation";
    import Header from "$lib/header.svelte";
  </script>
  
  <div class="min-h-screen flex flex-col bg-white">

    <Header />
  

    <div class="relative h-screen bg-white text-black">
      <div class="container mx-auto px-6 py-24">
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 transition-all duration-300 z-0">
  
          <div class="p-4">
            <div class="border-b-[5px] border-l-[5px]  rounded-2xl p-6 bg-white flex flex-col h-72 duration-75 hover:border-l-0 hover:border-b-0 shadow-lg">
              <span class="text-start mt-2 text-2xl font-semibold">Material Inward</span>
              <div class="flex-grow"></div>
              <button
                class="text-xl p-3 rounded-lg bg-black text-white font-bold self-end"
                on:click={() => goto("/inwardpage")}
              >
                Manage
              </button>
            </div>
          </div>
  

          <div class="p-4">
            <div class="border-b-[5px] border-l-[5px] rounded-2xl p-6 bg-white flex flex-col h-72 duration-75 hover:border-l-0 hover:border-b-0 shadow-lg">
              <span class="text-start mt-2 text-2xl font-semibold">Material Outward</span>
              <div class="flex-grow"></div>
              <button
                class="text-xl p-3 rounded-lg bg-black text-white font-bold self-end"
                on:click={() => goto("/outwardpage")}
              >
                Manage
              </button>
            </div>
          </div>
          <!-- <div class="p-4">
            <div class="border-b-[5px] border-l-[5px]rounded-2xl p-6 bg-white flex flex-col h-72 duration-75 hover:border-l-0 hover:border-b-0 shadow-lg">
              <span class="text-start mt-2 text-2xl font-semibold">Material Stock</span>
              <div class="flex-grow"></div>
              <button
                class="text-xl p-3 rounded-lg bg-black text-white font-bold self-end"
                on:click={() => goto("/materialstock")}
              >
                Manage
              </button>
            </div>
          </div> -->
          
         
          
        
        
         
        </div>
      </div>
    </div>
  

    <footer class="bg-black text-white py-3 text-center text-sm">
      &copy; 2024 SRA BAO. All Rights Reserved.
    </footer>
  </div>
  
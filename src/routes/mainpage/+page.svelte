<script>

	import { goto } from "$app/navigation";

</script>
<div class="min-h-screen flex flex-col bg-white">
    <!-- Header -->
    <header class="bg-black text-white shadow-md">
        <div class="container mx-auto px-6 py-4 flex items-center">
            <img src="/logo.jpeg" alt="SRA BAO Logo" class="w-12 h-12 rounded-full mr-4" />
            <div>
                <h1 class="text-3xl font-extralight text-white">SR Automation</h1>               
            </div>
        </div>
    </header>

    <!-- Content -->
    <div class="relative h-screen bg-white text-black">
        <div class="container mx-auto px-6 py-8">

            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 transition-all duration-300 z-0">
                <!-- Static PLC List Grid -->
                <div class="p-4">
                    <div class="border-b-[5px] border-l-[5px] border-black rounded-2xl p-6 bg-white flex flex-col h-72 duration-75 hover:border-l-0 hover:border-b-0 shadow-lg">
                        <span class="text-start mt-2 text-2xl font-semibold">Material Inward</span>
                        <div class="flex-grow"></div>
                        <button class="text-xl p-3 rounded-lg bg-black text-white font-bold self-end" 
                        onclick={()=> goto("/inwardpage")}>
                            Manage
                        </button>
                    </div>
                </div>
                <div class="p-4">
                    <div class="border-b-[5px] border-l-[5px] border-black rounded-2xl p-6 bg-white flex flex-col h-72 duration-75 hover:border-l-0 hover:border-b-0 shadow-lg">
                        <span class="text-start mt-2 text-2xl font-semibold">Material Outward</span>
                        <div class="flex-grow"></div>
                        <button class="text-xl p-3 rounded-lg bg-black text-white font-bold self-end"
                            onclick={() => goto("/outwardpage")}
                        >
                            Manage
                        </button>
                    </div>
                </div>
             
           
            </div>
        </div>

        <!-- Modal Button -->
     

     

        <!-- Success and Error Messages -->
  
    </div>

    <!-- Footer -->
    <footer class="bg-black text-white py-3 text-center text-sm">
        &copy; 2024 SRA BAO. All Rights Reserved.
    </footer>
</div>

<style>
    .center {
        display: flex;
        justify-content: center;
        align-items: center;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        height: 100%;
    }
</style>

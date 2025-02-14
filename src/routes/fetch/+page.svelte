<script>
    import Header from "$lib/header.svelte";
    import { onMount } from "svelte";
    
    let materials = [];
    let loading = true;
    let error = "";
    let searchQuery = "";

    async function fetchStockData() {
        loading = true;
        error = "";

        try {
            const response = await fetch("https://stocks-backend-t2bh.onrender.com/material", {
                method: "POST"
            });

            if (!response.ok) {
                throw new Error("Failed to fetch stock data");
            }

            materials = await response.json();
        } catch (err) {
            error = err.message;
        } finally {
            loading = false;
        }
    }

    function searchTable() {
        const query = searchQuery.toLowerCase().trim();
        return materials.filter(mat => mat.part_code.toLowerCase().includes(query));
    }

    onMount(fetchStockData);
</script>

<div class="min-h-screen flex flex-col bg-white ">
    <Header />


    <main class="flex-grow py-28 px-2">
   

        <div class="flex justify-end mb-4">
            <input
                type="text"
                bind:value={searchQuery}
                placeholder="Search by Part Code"
                class="px-3 py-2 w-80 bg-white text-black shadow-md border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
            />
        </div>

        {#if loading}
            <p class="text-gray-400 text-center">Loading stock data...</p>
        {:else if error}
            <p class="text-red-400 text-center">Error: {error}</p>
        {:else if materials.length === 0}
            <p class="text-gray-400 text-center">No stock data available.</p>
        {:else}
        <div class="overflow-x-auto bg-gray-100 shadow-lg rounded-lg">
            <table class="w-full border-collapse text-sm text-black">
                <thead class="bg-black text-white">
                        <tr>
                            <th class="py-3 px-4 text-center">Part Code</th>
                            <th class="py-3 px-4 text-center">Received</th>
                            <th class="py-3 px-4 text-center">Issued</th>
                            <th class="py-3 px-4 text-center">Stock</th>
                            <th class="py-3 px-4 text-center">Reorder Status</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#each searchTable() as mat}
                        <tr class="border-t border-gray-300 hover:bg-gray-200">
                                <td class="py-3 px-4 text-center">{mat.part_code}</td>
                                <td class="py-3 px-4 text-center">{mat.received}</td>
                                <td class="py-3 px-4 text-center">{mat.issued}</td>
                                <td class="py-3 px-4 text-center">{mat.stock}</td>
                                <td class="py-3 px-4 text-center">
                                    {#if mat.reorder_status}
                                        <span class="text-red-400 font-bold">⚠️ Reorder Needed</span>
                                    {:else}
                                        <span class="text-green-500">✔ In Stock</span>
                                    {/if}
                                </td>
                            </tr>
                        {/each}
                    </tbody>
                </table>
            </div>
        {/if}
    </main>
</div>

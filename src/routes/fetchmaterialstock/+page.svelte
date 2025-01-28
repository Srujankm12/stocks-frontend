<script>
    import { onMount } from "svelte";
    import Header from "$lib/header.svelte";
    let data = [];
    let isLoading = true;
    let expandedRow = null;

    const apiUrl = "http://localhost:8000/materialstockdata";

    function convertToIST(timestamp) {
        if (!timestamp) {
            console.error("Invalid timestamp:", timestamp);
            return "Invalid Date";
        }

        const date = new Date(timestamp);
        if (isNaN(date.getTime())) {
            console.error("Invalid Date:", timestamp);
            return "Invalid Date";
        }

        const options = {
            timeZone: "Asia/Kolkata",
            year: "numeric",
            month: "numeric",
            day: "numeric",
            hour: "2-digit",
            minute: "2-digit",
            second: "2-digit",
            hour12: true,
        };

        return new Intl.DateTimeFormat("en-IN", options)
            .format(date)
            .replace(/(AM|PM)/g, (match) => ` ${match}`);
    }

    onMount(async () => {
        try {
            const response = await fetch(apiUrl);
            if (response.ok) {
                const jsonResponse = await response.json();
                data = Array.isArray(jsonResponse) ? jsonResponse : [jsonResponse];
            } else {
                console.error("Failed to fetch data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        } finally {
            isLoading = false;
        }
    });
</script>

<div class="min-h-screen flex flex-col bg-white">
    <Header />
  
    <main class="flex-grow py-28 px-2">
        {#if isLoading}
            <div class="flex justify-center items-center h-full">
                <div class="animate-spin h-12 w-12 rounded-full border-t-4 border-gray-800 border-gray-300"></div>
            </div>
        {:else}
            <div class="overflow-x-auto bg-gray-100 shadow-lg rounded-lg">
                <table class="w-full border-collapse text-sm text-black">
                    <thead class="bg-black text-white">
                        <tr>
                            <th class="py-3 px-4">Timestamp</th>
                            <th class="py-3 px-4">Supplier</th>
                            <th class="py-3 px-4">Category</th>
                            <th class="py-3 px-4">Lead Time</th>
                            <th class="py-3 px-4">Std/Non-Std</th>
                            <th class="py-3 px-4">Part Code</th>
                            <th class="py-3 px-4">Unit</th>
                            <th class="py-3 px-4">Rate</th>
                            <th class="py-3 px-4">Min Retain</th>
                            <th class="py-3 px-4">Max Retain</th>
                            <th class="py-3 px-4">Received</th>
                            <th class="py-3 px-4">Issue</th>
                            <th class="py-3 px-4">Reserved Stock</th>
                            <th class="py-3 px-4">Stock</th>
                            <th class="py-3 px-4">Value</th>
                            <th class="py-3 px-4">Reorder Status</th>
                            <th class="py-3 px-4">Excess Stock</th>
                            <th class="py-3 px-4">Excess Stock Value</th>
                            <th class="py-3 px-4">Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#if data.length === 0}
                            <tr>
                                <td colspan="18" class="py-4 text-center text-gray-600">No data available</td>
                            </tr>
                        {:else}
                            {#each data as row, index}
                                <tr class="border-t border-gray-300 hover:bg-gray-200">
                                    <td class="py-3 px-4 whitespace-nowrap">{convertToIST(row.timestamp)}</td>
                                    <td class="py-3 px-4 text-center">{row.supplier}</td>
                                    <td class="py-3 px-4 text-center">{row.category}</td>
                                    <td class="py-3 px-4 text-center">{row.lead_time}</td>
                                    <td class="py-3 px-4 text-center">{row.std_non_std}</td>
                                    <td class="py-3 px-4 text-center">{row.part_code}</td>
                                    <td class="py-3 px-4 text-center">{row.unit}</td>
                                    <td class="py-3 px-4 text-center">₹{row.rate}</td>
                                    <td class="py-3 px-4 text-center">{row.minimum_retain}</td>
                                    <td class="py-3 px-4 text-center">{row.maximum_retain}</td>
                                    <td class="py-3 px-4 text-center">{row.received}</td>
                                    <td class="py-3 px-4 text-center">{row.issue}</td>
                                    <td class="py-3 px-4 text-center">{row.reserved_stock}</td>
                                    <td class="py-3 px-4 text-center">{row.stock}</td>
                                    <td class="py-3 px-4 text-center">₹{row.value}</td>
                                    <td class="py-3 px-4 text-center">
                                        {row.reorder_status === "true" ? "Reorder Needed" : "In Stock"}
                                    </td>
                                    <td class="py-3 px-4 text-center">{row.excess_stock}</td>
                                    <td class="py-3 px-4 text-center">₹{row.excess_stock_value}</td>
                                    <td class="py-3 px-4 text-center">
                                        <button
                                            class="bg-black text-white text-xs px-2 py-1 rounded hover:bg-gray-800 transition"
                                            on:click={() => expandedRow = expandedRow === index ? null : index}>
                                            {expandedRow === index ? "Hide" : "View"}
                                           
                                        </button>
                                    </td>
                                </tr>
                                {#if expandedRow === index}
                                    <tr class="bg-gray-200">
                                        <td colspan="18" class="py-4 px-6 text-sm text-gray-700">
                                            <p><strong>Supplier:</strong> {row.supplier}</p>
                                            <p><strong>Category:</strong> {row.category}</p>
                                            <p><strong>Part Code:</strong> {row.part_code}</p>
                                            <p><strong>Unit:</strong> {row.unit}</p>
                                            <p><strong>Rate:</strong> ₹{row.rate}</p>
                                            <p><strong>Stock:</strong> {row.stock}</p>
                                            <p><strong>Reorder Status:</strong> {row.reorder_status === "true" ? "Reorder Needed" : "In Stock"}</p>
                                            <p><strong>Excess Stock:</strong> {row.excess_stock}</p>
                                            <p><strong>Excess Stock Value:</strong> ₹{row.excess_stock_value}</p>
                                        </td>
                                    </tr>
                                {/if}
                            {/each}
                        {/if}
                    </tbody>
                </table>
            </div>
        {/if}
    </main>

    <footer class="bg-black text-white py-3 text-center text-sm">
        &copy; 2024 SRA BAO. All Rights Reserved.
    </footer>
</div>

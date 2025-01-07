<script>
    import { onMount } from "svelte";

    let data = [];
    let isLoading = true;
    let expandedRow = null;

    const apiUrl = "http://localhost:8000/getlist";

    // Function to convert timestamp to IST format
    function convertToIST(timestamp) {
        if (!timestamp) return "Invalid Date";

        const date = new Date(timestamp);

        if (isNaN(date.getTime())) return "Invalid Date";

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
            .replace(/(AM|PM)/g, (match) => ` ${match}`); // Add a space before AM/PM
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
    <!-- Header -->
    <header class="bg-black text-white shadow-md">
        <div class="container mx-auto px-6 py-4 flex items-center">
            <img src="/logo.jpeg" alt="SRA BAO Logo" class="w-12 h-12 rounded-full mr-4" />
            <div>
                <h1 class="text-3xl font-extralight text-white">SR Automation</h1>               
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-grow p-6">
        <h1 class="text-3xl font-bold text-center text-black mb-6">Material Inward Table</h1>

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
                            <th class="py-3 px-4">Buyer</th>
                            <th class="py-3 px-4">Part Code</th>
                            <th class="py-3 px-4">Serial Number</th>
                            <th class="py-3 px-4">Quantity</th>
                            <th class="py-3 px-4">PO Number</th>
                            <th class="py-3 px-4">PO Date</th>
                            <th class="py-3 px-4">Invoice Number</th>
                            <th class="py-3 px-4">Invoice Date</th>
                            <th class="py-3 px-4">Received Date</th>
                            <th class="py-3 px-4">Unit Price</th>
                            <th class="py-3 px-4">Category</th>
                            <th class="py-3 px-4">Warranty</th>
                            <th class="py-3 px-4">Warranty Due Days</th>
                            <th class="py-3 px-4">Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#if data.length === 0}
                            <tr>
                                <td colspan="16" class="py-4 text-center text-gray-600">No data available</td>
                            </tr>
                        {:else}
                            {#each data as row, index}
                                <tr class="border-t border-gray-300 hover:bg-gray-200">
                                    <td class="py-3 px-4 whitespace-nowrap">{convertToIST(row.timestamp)}</td>
                                    <td class="py-3 px-4 text-center">{row.supplier}</td>
                                    <td class="py-3 px-4 text-center">{row.buyer}</td>
                                    <td class="py-3 px-4 text-center">{row.partcode}</td>
                                    <td class="py-3 px-4 text-center">{row.serial_number}</td>
                                    <td class="py-3 px-4 text-center">{row.qty}</td>
                                    <td class="py-3 px-4 text-center">{row.po_no}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.po_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">{row.invoice_no}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.invoice_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.received_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">₹{row.unit_price_per_qty.toFixed(2)}</td>
                                    <td class="py-3 px-4 text-center">{row.category}</td>
                                    <td class="py-3 px-4 text-center">{row.warranty} Days</td>
                                    <td class="py-3 px-4 text-center">
                                        {row.warranty_due_days > 0 ? `${row.warranty_due_days} days remaining` : "Expired"}
                                    </td>
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
                                        <td colspan="16" class="py-4 px-6 text-sm text-gray-700">
                                            <p><strong>Supplier:</strong> {row.supplier}</p>
                                            <p><strong>Buyer:</strong> {row.buyer}</p>
                                            <p><strong>Part Code:</strong> {row.partcode}</p>
                                            <p><strong>Serial Number:</strong> {row.serial_number}</p>
                                            <p><strong>Quantity:</strong> {row.qty}</p>
                                            <p><strong>Unit Price:</strong> ₹{row.unit_price_per_qty.toFixed(2)}</p>
                                            <p><strong>Category:</strong> {row.category}</p>
                                            <p><strong>Warranty:</strong> {row.warranty} Days</p>
                                            <p><strong>Warranty Due Days:</strong> {row.warranty_due_days}</p>
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

    <!-- Footer -->
    <footer class="bg-black text-white py-3 text-center text-sm">
        &copy; 2024 SRA BAO. All Rights Reserved.
    </footer>
</div>

<script>
    import { onMount } from "svelte";

    let data = [];
    let isLoading = true;
    let expandedRow = null;

    const apiUrl = "http://localhost:8000/getlist";

    function convertToIST(timestamp) {
        if (!timestamp) {
            return "Invalid Date";
        }

        const regex = /(\d{2})\/(\d{2})\/(\d{4}), (\d{1,2}):(\d{2}):(\d{2}) (am|pm)/;
        const match = timestamp.match(regex);

        if (!match) {
            return "Invalid Date";
        }

        const [_, day, month, year, hourRaw, minute, second, period] = match;
        let hour = parseInt(hourRaw, 10);
        if (period.toLowerCase() === "pm" && hour < 12) hour += 12;
        if (period.toLowerCase() === "am" && hour === 12) hour = 0;

        const date = new Date(year, month - 1, day, hour, minute, second);

        if (isNaN(date.getTime())) return "Invalid Date";

        return new Intl.DateTimeFormat("en-IN", {
            year: "numeric",
            month: "long",
            day: "numeric",
            hour: "numeric",
            minute: "numeric",
            second: "numeric",
            hour12: true,
        }).format(date);
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

<div class="min-h-screen flex flex-col bg-gray-50">
    <!-- Header -->
    <header class="bg-gradient-to-r from-orange-500 to-orange-600 text-white shadow-md">
        <div class="container mx-auto px-6 py-4 flex items-center">
            <img src="/logo.jpeg" alt="SRA BAO Logo" class="w-12 h-12 rounded-full mr-4" />
            <div>
                <h1 class="text-xl font-bold">SRA BAO</h1>
                <p class="text-sm">Daily Reporting System</p>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-grow p-6">
        <h1 class="text-2xl font-bold text-center text-orange-600 mb-6">Material Inward Table</h1>

        {#if isLoading}
            <div class="flex justify-center items-center h-full">
                <div class="animate-spin h-12 w-12 rounded-full border-t-4 border-orange-500 border-gray-300"></div>
            </div>
        {:else}
            <div class="overflow-x-auto bg-white shadow-md rounded-lg">
                <table class="w-full border-collapse text-sm">
                    <thead class="bg-orange-500 text-white text-left">
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
                                <td colspan="16" class="py-4 text-center text-gray-500">No data available</td>
                            </tr>
                        {:else}
                            {#each data as row, index}
                                <tr class="border-t border-gray-200 hover:bg-gray-50">
                                    <td class="py-3 px-4">{convertToIST(row.timestamp)}</td>
                                    <td class="py-3 px-4">{row.supplier}</td>
                                    <td class="py-3 px-4">{row.buyer}</td>
                                    <td class="py-3 px-4">{row.partcode}</td>
                                    <td class="py-3 px-4">{row.serial_number}</td>
                                    <td class="py-3 px-4">{row.qty}</td>
                                    <td class="py-3 px-4">{row.po_no}</td>
                                    <td class="py-3 px-4">{new Date(row.po_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4">{row.invoice_no}</td>
                                    <td class="py-3 px-4">{new Date(row.invoice_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4">{new Date(row.received_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4">{row.unit_price_per_qty.toFixed(2)}</td>
                                    <td class="py-3 px-4">{row.category}</td>
                                    <td class="py-3 px-4">{row.warranty} Days</td>
                                    <td class="py-3 px-4">
                                        {row.warranty_due_days > 0 ? `${row.warranty_due_days} days remaining` : "Expired"}
                                    </td>
                                    <td class="py-3 px-4 text-center">
                                        <button
                                            class="bg-orange-500 text-white text-xs px-2 py-1 rounded hover:bg-orange-600"
                                            on:click={() => expandedRow = expandedRow === index ? null : index}>
                                            {expandedRow === index ? "Hide" : "View"}
                                        </button>
                                    </td>
                                </tr>
                                {#if expandedRow === index}
                                    <tr class="bg-gray-50">
                                        <td colspan="16" class="py-4 px-6 text-sm text-gray-700">
                                            <p><strong>Supplier:</strong> {row.supplier}</p>
                                            <p><strong>Buyer:</strong> {row.buyer}</p>
                                            <p><strong>Part Code:</strong> {row.partcode}</p>
                                            <p><strong>Serial Number:</strong> {row.serial_number}</p>
                                            <p><strong>Quantity:</strong> {row.qty}</p>
                                            <p><strong>Unit Price:</strong> {row.unit_price_per_qty.toFixed(2)}</p>
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
    <footer class="bg-gray-800 text-white py-3 text-center text-sm">
        &copy; 2024 SRA BAO. All Rights Reserved.
    </footer>
</div>

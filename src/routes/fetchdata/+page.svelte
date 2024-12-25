<script>
    import { onMount } from "svelte";

    let data = [];
    let isLoading = true; // Add loading state
    const apiUrl = "http://localhost:8000/getlist"; // Your API endpoint

    // State for expanded row details
    let expandedRow = null;

   // Function to convert UTC to Bengaluru time (IST)
 function convertToIST(timestamp) {
    if (!timestamp) {
        return "Invalid Date";
    }

    // Regex to match the timestamp format
    const regex = /(\d{2})\/(\d{2})\/(\d{4}), (\d{1,2}):(\d{2}):(\d{2}) (am|pm)/;
    const match = timestamp.match(regex);

    if (!match) {
        return "Invalid Date";
    }

    // Extract components from the timestamp
    const day = parseInt(match[1], 10);
    const month = parseInt(match[2], 10) - 1; // Zero-indexed month
    const year = parseInt(match[3], 10);
    let hour = parseInt(match[4], 10);
    const minute = parseInt(match[5], 10);
    const second = parseInt(match[6], 10);
    const period = match[7].toLowerCase();

    // Adjust hour for AM/PM format
    if (period === "pm" && hour < 12) {
        hour += 12;
    } else if (period === "am" && hour === 12) {
        hour = 0;
    }

    // Create the Date object (without adjusting for time zone)
    const date = new Date(year, month, day, hour, minute, second);

    if (isNaN(date.getTime())) {
        return "Invalid Date";
    }

    // Format the date and time for display in Bengaluru time (IST)
    const options = {
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
        second: "numeric",
        hour12: true,
    };

    const bengaluruTime = new Intl.DateTimeFormat("en-IN", options).format(date);
    return bengaluruTime;
}


    // Fetch data from the API
    onMount(async () => {
        try {
            const response = await fetch(apiUrl);
            if (response.ok) {
                const jsonResponse = await response.json();
                console.log(jsonResponse); // Log the response to inspect the structure
                data = Array.isArray(jsonResponse) ? jsonResponse : [jsonResponse]; // Ensure it's an array
            } else {
                console.error("Failed to fetch data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        } finally {
            isLoading = false; // Set loading state to false after data is fetched
        }
    });
</script>

<div class="min-h-screen flex flex-col">
    <!-- Header -->
    <header class="bg-gradient-to-r from-orange-600 to-orange-500 text-white shadow-lg">
        <div class="container mx-auto px-6 py-4 flex items-center justify-between">
          <!-- Logo and Branding -->
          <div class="flex items-center space-x-4">
            <img src="/logo.jpeg" alt="SRA BAO Logo" class="w-16 h-16 rounded-full border-4 border-white shadow-md" />
            <div>
              <h1 class="text-3xl font-extrabold leading-tight tracking-wide">SRA BAO</h1>
              <p class="text-sm font-medium opacity-90">Daily Reporting System</p>
            </div>
          </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-grow bg-gradient-to-br from-orange-100 via-white to-orange-50 p-8">
        <h1 class="text-4xl font-extrabold text-orange-700 mb-8 text-center">Purchase Order Table</h1>

        <!-- Loading Spinner -->
        {#if isLoading}
            <div class="fixed inset-0 flex items-center justify-center z-40 bg-white bg-opacity-75">
                <div class="w-16 h-16 border-6 border-t-8 border-orange-500 border-solid rounded-full animate-spin"></div>
            </div>
        {:else}
            <div class="overflow-x-auto rounded-lg shadow-2xl">
                <table class="w-full text-left border-collapse bg-white rounded-lg">
                    <!-- Table Header -->
                    <thead class="bg-orange-600 text-white">
                        <tr>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Timestamp</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Supplier</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Buyer</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Partcode</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Serial Number</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Quantity</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">PO Number</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">PO Date</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Invoice Number</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Invoice Date</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Received Date</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Unit Price</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Category</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Warranty</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Warranty Due Days</th>
                            <th class="px-6 py-4 text-sm font-semibold uppercase text-center">Actions</th>
                        </tr>
                    </thead>
                    <!-- Table Body -->
                    <tbody>
                        {#if data.length === 0}
                            <tr>
                                <td colspan="16" class="px-6 py-4 text-center text-gray-500">
                                    No data available
                                </td>
                            </tr>
                        {:else}
                            {#each data as row, index}
                                <tr class="even:bg-orange-100 odd:bg-orange-50 transition-all duration-300">
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">
                                        {convertToIST(row.timestamp)}
                                    </td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.supplier}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.buyer}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.partcode}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.serial_number}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.qty}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.po_no}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{new Date(row.po_date).toLocaleDateString()}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.invoice_no}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{new Date(row.invoice_date).toLocaleDateString()}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{new Date(row.received_date).toLocaleDateString()}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.unit_price_per_qty}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.category}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.warranty}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center text-gray-700">{row.warranty_due_days}</td>
                                    <td class="px-6 py-4 border-b text-sm text-center">
                                        <button 
                                            class="px-4 py-2 bg-orange-500 text-white rounded hover:bg-orange-600"
                                            on:click={() => expandedRow = expandedRow === index ? null : index}>
                                            {expandedRow === index ? "Hide" : "View"}
                                        </button>
                                    </td>
                                </tr>
                                <!-- Expanded Row Details -->
                                {#if expandedRow === index}
                                    <tr class="bg-orange-50">
                                        <td colspan="16" class="px-6 py-4">
                                            <p><strong>Supplier:</strong> {row.supplier}</p>
                                            <p><strong>Buyer:</strong> {row.buyer}</p>
                                            <p><strong>Partcode:</strong> {row.partcode}</p>
                                            <p><strong>Serial Number:</strong> {row.serial_number}</p>
                                            <p><strong>Quantity:</strong> {row.qty}</p>
                                            <p><strong>PO Number:</strong> {row.po_no}</p>
                                            <p><strong>PO Date:</strong> {row.po_date}</p>
                                            <p><strong>Invoice Number:</strong> {row.invoice_no}</p>
                                            <p><strong>Invoice Date:</strong> {row.invoice_date}</p>
                                            <p><strong>Received Date:</strong> {row.received_date}</p>
                                            <p><strong>Unit Price:</strong> {row.unit_price_per_qty}</p>
                                            <p><strong>Category:</strong> {row.category}</p>
                                            <p><strong>Warranty:</strong> {row.warranty}</p>
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
    <footer class="bg-orange-600 text-white py-4 shadow-inner">
        <div class="container mx-auto px-4 text-center text-sm">
            <p>&copy; 2024 SRA BAO. All Rights Reserved.</p>
        </div>
    </footer>
</div>

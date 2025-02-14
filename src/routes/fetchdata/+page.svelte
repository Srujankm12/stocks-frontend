<script>
    import { onMount } from "svelte";
    import Header from "$lib/header.svelte";
    import { fade } from "svelte/transition";

    let data = [];
    let isLoading = true;
    let expandedRow = null;
    let showDownloadMessage = false; 
    let filteredData =[];
    let searchQuery = "";

    const apiUrl = "https://stocks-backend-t2bh.onrender.com/getlist";
    const downlaodUrl = "https://stocks-backend-t2bh.onrender.com/downloadinward"
     const downloadexcel = async () => {
        try {
            const response = await fetch(downlaodUrl, {
                method: "GET",
               
            });
            if (response.ok) {
                const blob = await response.blob();
                const url = window.URL.createObjectURL(blob);
                const a = document.createElement("a");
                a.href = url;
                a.download = "inwarddata.xlsx";
                a.click();
                window.URL.revokeObjectURL(url);
            } else {
                console.error("Failed to download Excel file:", response.statusText);
            }
        } catch (error) {
            console.error("Error downloading Excel file:", error);
        }

        showDownloadMessage = true;
        setTimeout(() => {
            showDownloadMessage = false;
        }, 3000);
    };


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
            .replace(/(AM|PM)/g, (match) => ` ${match}`); 
    }
 
    onMount(async () => {
        try {
            const response = await fetch(apiUrl);
            if (response.ok) {
                const jsonResponse = await response.json();
                data = Array.isArray(jsonResponse) ? jsonResponse : [jsonResponse];
                filteredData =[...data]
                console.log(filteredData)
            } else {
                console.error("Failed to fetch data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        } finally {
            isLoading = false;
        }
    });
 
function searchTable() {
    const query = searchQuery.toLowerCase().trim();
    if (!query) {
        filteredData = [...data]; // Reset when search query is empty
        return;
    }

    filteredData = data.filter(row => {
        const supplierName = row.supplier?.toLowerCase() || ""; 
        const buyerName = row.buyer?.toLowerCase() || ""; 
        return supplierName.includes(query) || buyerName.includes(query);
    });
}
</script>


<div class="min-h-screen flex flex-col bg-white ">
    <Header />


    <main class="flex-grow py-28 px-2">
        <div class="relative w-full flex justify-end bg-white">
            <input
              type="text"
              bind:value={searchQuery}
              on:input={searchTable}
              placeholder="Search by supplier or buyer"
              class="px-3 py-1  w-80 mb-4 shadow-md border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none "
            />
          </div>

        {#if isLoading}
            <div class="flex justify-center items-center h-full">
                <div class="animate-spin h-12 w-12 rounded-full border-t-4 border-gray-800 border-gray-300"></div>
            </div>
        {:else}
            <div class="overflow-x-auto bg-gray-100 shadow-lg rounded-lg">
                <table class="w-full border-collapse text-sm text-black">
                    <thead class="bg-black text-white">
                        <tr>
                            <th class="py-3 px-4">S.No</th>
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
                            {#each filteredData as row, index(row.id)  }
                                <tr class="border-t border-gray-300 hover:bg-gray-200">
                              <td class="py-3 px-4 text-center">{index + 1}</td>
                                    <td class="py-3 px-4 whitespace-nowrap">{row.timestamp}</td>
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
            <div class="py-4 flex justify-center items-center fixed bottom-0 left-0 right-0 text-center">
                <button class="text-white bg-black rounded-md px-9 py-2"
                on:click={downloadexcel}
                 >
                
               
                    Download
                </button>
            </div>
            
        {/if}
    </main>
    {#if showDownloadMessage}
    <div class="fixed bottom-12 right-4 bg-black text-white font-semibold p-4 rounded-md shadow-2xl duration-300" transition:fade>
        Excel downloaded successfully!
    </div>
{/if}

</div>

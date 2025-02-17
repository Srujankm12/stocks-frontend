<script>
    import { onMount } from "svelte";
    import Header from "$lib/header.svelte";
    import { fade } from "svelte/transition";

    let data = [];
    let filteredData = [];
    let isLoading = true;
    let searchQuery = "";
    let showDownloadMessage = false;
    let expandedRow = null;
    let showUpdateModal = false;
   
    let selectedRow = null;
    const updateData = {
      
};


    const apiUrl = "https://stocks-backend-t2bh.onrender.com/fetchoutward";
    const downloadexcelmo = "https://stocks-backend-t2bh.onrender.com/downloadoutward";

    async function fetchData() {
        try {
            const response = await fetch(apiUrl);
            if (response.ok) {
                const jsonResponse = await response.json();
                data = Array.isArray(jsonResponse) ? jsonResponse : [jsonResponse];
                filteredData = [...data]; 
            } else {
                console.error("Failed to fetch data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        } finally {
            isLoading = false;
        }
    }

    async function downloadExcelMaterialOutward() {
        try {
            const response = await fetch(downloadexcelmo, { method: "GET" });
            if (response.ok) {
                const blob = await response.blob();
                const url = window.URL.createObjectURL(blob);
                const a = document.createElement("a");
                a.href = url;
                a.download = "outwarddata.xlsx";
                a.click();
                window.URL.revokeObjectURL(url);
            } else {
                console.error("Failed to download Excel file:", response.statusText);
            }
        } catch (error) {
            console.error("Error downloading Excel file:", error);
        }
    }

    function convertToIST(timestamp) {
        if (!timestamp) return "Invalid Date";
        const date = new Date(timestamp);
        if (isNaN(date.getTime())) return "Invalid Date";

        return new Intl.DateTimeFormat("en-IN", {
            timeZone: "Asia/Kolkata",
            year: "numeric",
            month: "numeric",
            day: "numeric",
            hour: "2-digit",
            minute: "2-digit",
            second: "2-digit",
            hour12: true,
        }).format(date).replace(/(AM|PM)/g, (match) => ` ${match}`);
    }
    function openUpdateModal(row) {
     
     selectedRow = row; 
     showUpdateModal = true;
 }

    function searchTable() {
        const query = searchQuery.toLowerCase().trim();
        if (!query) {
            filteredData = [...data]; 
            return;
        }

        filteredData = data.filter(row => {
            const sellerName = row.seller?.toLowerCase() || "";
            const customerName = row.customer?.toLowerCase() || "";
            return sellerName.includes(query) || customerName.includes(query);
        });
    }

    onMount(fetchData);

  
</script>

<div class="min-h-screen flex flex-col bg-white">
    <Header />
  

    <main class="flex-grow py-24 px-2">
        <div class="relative w-full flex justify-end bg-white">
            <input
              type="text"
              bind:value={searchQuery}
              on:input={searchTable}
              placeholder="Search by seller or customer"
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
                            <th class="py-3 px-4">ID</th>
                            <th class="py-3 px-4">Timestamp</th>
                            <th class="py-3 px-4">Seller</th>
                            <th class="py-3 px-4">Customer</th>
                            <th class="py-3 px-4">Part Code</th>
                            <th class="py-3 px-4">Serial Number</th>
                            <th class="py-3 px-4">Quantity</th>
                            <th class="py-3 px-4">PO Number</th>
                            <th class="py-3 px-4">PO Date</th>
                            <th class="py-3 px-4">Invoice Number</th>
                            <th class="py-3 px-4">Invoice Date</th>
                            <th class="py-3 px-4">Delivery Date</th>
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
                            {#each filteredData as row, index}
                                <tr class="border-t border-gray-300 hover:bg-gray-200">
                                    <td class="py-3 px-4 text-center">{index + 1}</td>
                                    <td class="py-3 px-4 whitespace-nowrap">
                                      {row.timestamp}</td>
                                    <td class="py-3 px-4 text-center">{row.seller}</td>
                                    <td class="py-3 px-4 text-center">{row.customer}</td>
                                    <td class="py-3 px-4 text-center">{row.partcode}</td>
                                    <td class="py-3 px-4 text-center">{row.serial_number}</td>
                                    <td class="py-3 px-4 text-center">{row.qty}</td>
                                    <td class="py-3 px-4 text-center">{row.cus_po_no}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.cus_po_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">{row.cus_invoice_no}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.cus_invoice_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.delivery_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">₹{row.unit_price_per_qty.toFixed(2)}</td>
                                    <td class="py-3 px-4 text-center">{row.category}</td>
                                    <td class="py-3 px-4 text-center">{row.warranty} Days</td>
                                    <td class="py-3 px-4 text-center">
                                        {row.warranty_due_days > 0 ? `${row.warranty_due_days} days remaining` : "Expired"}
                                    </td>
                                    <td class="py-3 px-4 text-center">
                                        <!-- svelte-ignore a11y_consider_explicit_label -->
                                     <button
                                            class="text-xl font-medium rounded-full flex items-center justify-center text-center"
                                            on:click={() => openUpdateModal(row)}
                                        >
                                            <i class="fas fa-edit"></i>
                                        </button>
                                    </td>
                                </tr>
                              
                            {/each}
                        {/if}
                        
                    </tbody>
                </table>
            </div>
            <div class="py-4 flex justify-center items-center fixed bottom-0 left-0 right-0 text-center">
                <button class="text-white bg-black rounded-md px-9 py-2" 
                on:click={downloadExcelMaterialOutward}
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
{#if showUpdateModal}
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center" transition:fade>
            <div class="w-full max-w-2xl mx-4 bg-white rounded-lg p-6 shadow-md relative">
                <h1 class="text-center text-xl py-2 mb-6 font-medium text-gray-900">Update InwardData</h1>

             
                            
                       
          
            
                 <button
                    class="absolute top-4 right-4 text-gray-500"
                    on:click={() => showUpdateModal = false}
                >
                    <i class="fas fa-times"></i>
                </button>
            </div>
        </div>
    {/if}

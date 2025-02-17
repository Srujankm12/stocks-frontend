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
    let isUpdating = false;

    let sellers = [];
    let issuesAgainst = [];
    let branchRegions = [];
    let categories = [];

    const updateData = {
        id: "",
        customer: "",
        seller: "",
        branch_region: "",
        partcode: "",
        serial_number: "",
        qty : "",
        cus_po_no: "",
        cus_po_date: "",
        cus_invoice_no: "",
        cus_invoice_date: "",
       delivery_date: "",
        unit_price_per_qty: "",
        issue_against: "",
        notes: "",
        category: "",
        warranty: "",
};
function openUpdateModal(row) {
      selectedRow = row;
      showUpdateModal = true;
      updateData.id = row.id;
      updateData.customer = row.customer || "";
      updateData.seller = row.seller || "";
      updateData.branch_region = row.branch_region || "";
      updateData.partcode = row.partcode || "";
      updateData.serial_number = row.serial_number || "";
      updateData.qty = row.qty|| "";
      updateData.cus_po_no = row.cus_po_no || "";
      updateData.cus_po_date = row.cus_po_date ? new Date(row.cus_po_date).toISOString().split('T')[0]: "";
      updateData.cus_invoice_no = row.cus_invoice_no || "";
      updateData.cus_invoice_date = row.cus_invoice_date ? new Date(row.cus_invoice_date).toISOString().split('T')[0]: "";
      updateData.delivery_date = row.delivery_date ? new Date(row.delivery_date).toISOString().split('T')[0]: "";
      updateData.unit_price_per_qty = row.unit_price_per_qty || "";
      updateData.issue_against = row.issue_against || "";
      updateData.notes = row.notes || "";
      updateData.category = row.category || "";
      updateData.warranty = row.warranty || "";
      updateData.issue_against = row.issue_against || "";

    }

    const apiUrl = "https://stocks-backend-t2bh.onrender.com/fetchoutward";
    const downloadexcelmo = "https://stocks-backend-t2bh.onrender.com/downloadoutward";
    const updateurl = "https://stocks-backend-t2bh.onrender.com/updateoutward";

    async function updateOutwardData(event) {
        event.preventDefault();
        isUpdating = true;
        try {
            const updatedFields = {};

            Object.keys(updateData).forEach(key => {
                if (updateData[key] !== selectedRow[key]) {
                    updatedFields[key] = updateData[key];
                }
            });

            if (Object.keys(updatedFields).length === 0) {
                console.log("No fields updated.");
                return;
            }

            const updatedData = {
                ...selectedRow, 
                ...updatedFields, 
            };

            console.log("Updated Data:", updatedData);

            const response = await fetch(updateurl, {
                method: "POST",
              
                body: JSON.stringify(updatedData),
            });

            if (response.ok) {
                console.log("OutwardData updated successfully");
                showUpdateModal = false;
                selectedRow = null;
                await fetchData(); 
            } else {
                const errorMessage = await response.text();  
                console.error("Failed to update Outwarddata:", errorMessage);
            }
        } catch (error) {
            console.error("Error updating outwarddata:", error);
        } finally {
            isUpdating = false;
        }
    }

    async function fetchDropdownData() {
        isLoading = true;
        try {
            const response = await fetch('https://stocks-backend-t2bh.onrender.com/outwardDropdown', { method: 'GET' });
            if (response.ok) {
                const data = await response.json();
                sellers = [...new Set(data.map(item => item.seller))];
                branchRegions = [...new Set(data.map(item => item.branch_region))];
                issuesAgainst = [...new Set(data.map(item => item.issue_against))];
                categories = [...new Set(data.map(item => item.category))];
            } else {
                console.error("Failed to fetch dropdown data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching dropdown data:", error);
        } finally {
            isLoading = false;
        }
    }

    async function fetchData() {
        try {
            const response = await fetch(apiUrl);
            if (response.ok) {
                const jsonResponse = await response.json();
                data = Array.isArray(jsonResponse) ? jsonResponse : [jsonResponse];
                filteredData = [...data]; 
                console.log("Fetched data:", data);
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
                showDownloadMessage = true;
                setTimeout(() => {
                    showDownloadMessage = false;
                }, 3000);
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

    onMount(async () => {
        await fetchData();
        await fetchDropdownData();
    });


  
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
                <div class="animate-spin h-12 w-12 rounded-full border-t-4 border-gray-800"></div>
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
                            <th class="py-3 px-4">Branch Region</th>
                            <th class="py-3 px-4">Part Code</th>
                            <th class="py-3 px-4">Serial Number</th>
                            <th class="py-3 px-4">Quantity</th>
                            <th class="py-3 px-4">PO Number</th>
                            <th class="py-3 px-4">PO Date</th>
                            <th class="py-3 px-4">Invoice Number</th>
                            <th class="py-3 px-4">Invoice Date</th>
                            <th class="py-3 px-4">Delivery Date</th>
                            <th class="py-3 px-4">Unit Price</th>
                            <th class ="py-3 px-4">Issue Against</th>
                            <th class="py-3 px-4">Notes</th>
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
                                    <td class="py-3 px-4 text-center">{row.branch_region}</td>
                                    <td class="py-3 px-4 text-center">{row.partcode}</td>
                                    <td class="py-3 px-4 text-center">{row.serial_number}</td>
                                    <td class="py-3 px-4 text-center">{row.qty}</td>
                                    <td class="py-3 px-4 text-center">{row.cus_po_no}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.cus_po_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">{row.cus_invoice_no}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.cus_invoice_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">{new Date(row.delivery_date).toLocaleDateString()}</td>
                                    <td class="py-3 px-4 text-center">₹{row.unit_price_per_qty.toFixed(2)}</td>
                                    <td class="py-3 px-4 text-center">{row.issue_against}</td>
                                    <td class ="py-3 px-4 text-center">{row.notes}</td>
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
<div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 transition-opacity">
    <div class="w-full max-w-4xl mx-4 bg-white rounded-lg p-4 shadow-xl relative">
        <h1 class="text-center text-xl py-2 mb-6 font-semibold text-gray-900">Update Outward Data</h1>

        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
            <div>
                <label for="supplier" class="block text-sm font-medium text-gray-700">Seller</label>
                <select id="supplier" bind:value={updateData.seller} class="w-full p-2 border border-gray-300 rounded-md">
                    <option value="">Select Seller</option>
                    {#each sellers as seller}
                        <option value={seller}>{seller}</option>
                    {/each}
                </select>
            </div>
           <div>
            <label for="customer" class="block text-sm font-medium text-gray-700">Customer</label>
            <input type="text" id="customer" bind:value={updateData.customer} class="w-full p-2 border border-gray-300 rounded-md" />
           </div>

           <div>
            <label for ="branch_region" class="block text-sm font-medium text-gray-700">Branch Region</label>
            <select id="branch_region" bind:value={updateData.branch_region} class="w-full p-2 border border-gray-300 rounded-md">
                <option value="">Select Branch Region</option>
                {#each branchRegions as region}
                <option value={region}>{region}</option>
                {/each}
                </select>
                </div>
            
            <div>
                <label for = "part_code" class="block text-sm font-medium text-gray-700">Part Code</label>
                <input type="text" id="part_code" bind:value={updateData.partcode} class="w-full p-2 border border-gray-300 rounded-md" />

                </div>
                <div>
                    <label for ="serial_number" class="block text-sm font-medium text-gray-700">Serial Number</label>
                    <input type="text" id="serial_number" bind:value={updateData.serial_number } class="w-full p-2 border border-gray-300 rounded-md" />
                 </div>

            <div>
                <label for ="qty" class="block text-sm font-medium text-gray-700">Qty</label>
                <input type="number" id="qty" bind:value={updateData.qty} class="w-full p-2 border border-gray-300 rounded-md" />
                </div>
             <div>
                    <label for = "cus_po_no" class="block text-sm font-medium text-gray-700">PO Number</label>
                    <input type="text" id="cus_po_no" bind:value={updateData.cus_po_no} class="w-full p-2 border border-gray-300 rounded-md" />
                    </div>
                <div>
                        <label for = "cus_po_date" class="block text-sm font-medium text-gray-700">PO Date</label>
                        <input type="date" id="cus_po_date" bind:value={updateData.cus_po_date} class="w-full p-2 border border-gray-300 rounded-md" />
                </div>

                <div>
                    <label for ="cus_invoice_no" class="block text-sm font-medium text-gray-700">Invoice Number</label>
                    <input type="text" id="cus_invoice_no" bind:value={updateData.cus_invoice_no} class="w-full p-2 border border-gray-300 rounded-md" />
                </div>

                <div>
                    <label for ="cus_invoice_date" class="block text-sm font-medium text-gray-700">Invoice Date</label>
                    <input type="date" id="cus_invoice_date" bind:value={updateData.cus_invoice_date} class="w-full p-2 border border-gray-300 rounded-md" />
                </div>

                <div>
                    <label for = "delivery_date" class="block text-sm font-medium text-gray-700">Delivery Date</label>
                    <input type="date" id="delivery_date" bind:value={updateData.delivery_date } class="w-full p-2 border border-gray-300 rounded-md" />

                </div>
                <div>
                    <label for ="unit_price_per_qty" class="block text-sm font-medium text-gray-700">Unit Price Per Qty</label>
                    <input type="number" id="unit_price_per_qty" bind:value={updateData.unit_price_per_qty} class="w-full p-2 border border-gray-300 rounded-md" />
                </div>

                <div>
                    <label for ="issue_against" class="block text-sm font-medium text-gray-700">Issue Against</label>
                 <select  id="issue_against" bind:value={updateData. issue_against} class="w-full p-2 border border-gray-300 rounded-md">
                    <option value="">Select Issue Against</option>
                    {#each issuesAgainst as issue}
                    <option value={issue}>{issue}</option>
                    {/each}
                 </select>
                 </div>

                <div>
                    <label for = "notes" class="block text-sm font-medium text-gray-700">Notes</label>
                    <textarea id="notes" bind:value={updateData.notes} class="w-full p-2 border border-gray-300 rounded-md"></textarea>
                </div>
                <div>
                    <label for ="category" class="block text-sm font-medium text-gray-700">Category</label>
                    <select id="category" bind:value={updateData.category} class="w-full p-2 border border-gray-300 rounded-md">
                        <option value="">Select Category</option>
                        {#each categories as category}
                        <option value={category}>{category}</option>
                        {/each}
                    </select>
                    </div>


            </div>
       
            <div class="mt-6 flex justify-end gap-4">
            <button on:click={() => showUpdateModal = false} class="px-4 py-2 bg-gray-300 text-gray-800 rounded-md hover:bg-gray-400">Cancel</button>
       <button on:click={updateOutwardData} class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700">Save</button> 
        </div>

        <!-- Close Button -->
        <button
            class="absolute top-4 right-4 text-gray-500 hover:text-gray-700"
            on:click={() => showUpdateModal = false}>
            ✖
        </button>
    </div>
    </div>

{/if}

<script>
    import { onMount } from "svelte";
    import Header from "$lib/header.svelte";
    import { fade } from "svelte/transition";

    let data = [];
    let isLoading = true;
    let expandedRow = null;
    let showDownloadMessage = false;
    let filteredData = [];
    let searchQuery = "";
    let showUpdateModal = false;
    let selectedRow = null;
    let isUpdating = false;

    const updateData = {
        id: "",
        supplier: "",
        buyer: "",
        category: "",
        partcode: "",
        serial_number: "",
        quantity: "",
        unit_price_per_qty: "",
        po_no: "",
        po_date: "",
        invoice_no: "",
        invoice_date: "",
        received_date: "",
        warranty: ""
    };

    const apiUrl = "https://stocks-backend-t2bh.onrender.com/getlist";
    const downloadUrl = "https://stocks-backend-t2bh.onrender.com/downloadinward";
    const updateUrl = "https://stocks-backend-t2bh.onrender.com/update";

    let suppliers = [];
    let buyers = [];
    let categories = [];

    // Function to fetch data from API
    async function fetchData() {
        isLoading = true;
        try {
            const response = await fetch(apiUrl);
            if (response.ok) {
                const jsonResponse = await response.json();
                data = Array.isArray(jsonResponse) ? jsonResponse : [jsonResponse];
                filteredData = [...data]; // Update filtered data
                console.log("Fetched Data:", filteredData);
            } else {
                console.error("Failed to fetch data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        } finally {
            isLoading = false;
        }
    }

    async function updateInwardData(event) {
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

            const response = await fetch(updateUrl, {
                method: "POST",
             
                body: JSON.stringify(updatedData),
            });

            if (response.ok) {
                console.log("Inward updated successfully");
                showUpdateModal = false;
                selectedRow = null;
                await fetchData(); // Fetch updated data
            } else {
                const errorMessage = await response.text();
                console.error("Failed to update inwarddata:", errorMessage);
            }
        } catch (error) {
            console.error("Error updating Inwarddata:", error);
        } finally {
            isUpdating = false;
        }
    }

    const fetchSuppliersAndBuyers = async () => {
        try {
            const response = await fetch('https://stocks-backend-t2bh.onrender.com/inward', { method: 'GET' });

            if (response.ok) {
                const data = await response.json();
                const uniqueSuppliers = [...new Set(data.map(item => item.supplier))];
                const uniqueBuyers = [...new Set(data.map(item => item.buyer))];
                const uniqueCategories = [...new Set(data.map(item => item.category))];

                suppliers = uniqueSuppliers;
                buyers = uniqueBuyers;
                categories = uniqueCategories;

                console.log('Suppliers:', uniqueSuppliers);
                console.log('Buyers:', uniqueBuyers);
                console.log('Categories:', uniqueCategories);
            } else {
                const errorData = await response.json();
                console.error('Error fetching data:', errorData.message);
            }
        } catch (error) {
            console.error('Error fetching data:', error);
        }
    };

    onMount(async () => {
        await fetchSuppliersAndBuyers();
        await fetchData();
    });

    const downloadExcel = async () => {
        try {
            const response = await fetch(downloadUrl, { method: "GET" });
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

    function openUpdateModal(row) {
        selectedRow = row;
        showUpdateModal = true;
        updateData.id = row.id;
        updateData.supplier = row.supplier || "";
        updateData.buyer = row.buyer || "";
        updateData.category = row.category || "";
        updateData.partcode = row.partcode || "";
        updateData.serial_number = row.serial_number || "";
        updateData.quantity = row.quantity || "";
        updateData.unit_price_per_qty = row.unit_price_per_qty || "";
        updateData.po_date = row.po_date ? new Date(row.po_date).toISOString().split('T')[0] : "";
        updateData.invoice_date = row.invoice_date ? new Date(row.invoice_date).toISOString().split('T')[0] : "";
        updateData.received_date = row.received_date ? new Date(row.received_date).toISOString().split('T')[0] : "";
        updateData.po_no = row.po_no || "";
        updateData.invoice_no = row.invoice_no || "";
        updateData.warranty = row.warranty || "";
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
                <div class="animate-spin h-12 w-12 rounded-full border-t-4 border-gray-800"></div>
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
{#if showUpdateModal}
<div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 transition-opacity">
    <div class="w-full max-w-4xl mx-4 bg-white rounded-lg p-4 shadow-xl relative">
        <h1 class="text-center text-xl py-2 mb-6 font-semibold text-gray-900">Update InwardData</h1>

        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
            <!-- Column 1 -->
            <div>
                <label for="supplier" class="block text-sm font-medium text-gray-700">Supplier</label>
                <select id="supplier" bind:value={updateData.supplier} class="w-full p-2 border border-gray-300 rounded-md">
                    <option value="">Select Supplier</option>
                    {#each suppliers as supplier}
                        <option value={supplier}>{supplier}</option>
                    {/each}
                </select>
            </div>
            <div>
                <label for="buyer" class="block text-sm font-medium text-gray-700">Buyer</label>
                <select id="buyer" bind:value={updateData.buyer} class="w-full p-2 border border-gray-300 rounded-md">
                    <option value="">Select Supplier</option>
                    {#each buyers as buyer}
                        <option value={buyer}>{buyer}</option>
                    {/each}
                </select>
            </div>

            <div>
                <label for="category" class="block text-sm font-medium text-gray-700">Category</label>
                <select id="category" 
                        bind:value={updateData.category} 
                        class="w-full p-2 border border-gray-300 rounded-md " required>
                    <option value="" disabled>Select Category</option>
                    {#each categories as category}
                        <option value={category}>{category}</option>
                    {/each}
                </select>
            </div>

            <div>
                <label for="partcode" class="block text-sm font-medium text-gray-700">Part Code</label>
                <input id="partcode" type="text" bind:value={updateData.partcode} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>

            <div>
                <label for="serial_number" class="block text-sm font-medium text-gray-700">Serial Number</label>
                <input id="serial_number" type="text" bind:value={updateData.serial_number} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>

            <div>
                <label for="quantity" class="block text-sm font-medium text-gray-700">Quantity</label>
                <input id="quantity" type="number" bind:value={updateData.qty} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>

            <div>
                <label for="unit_price_per_qty" class="block text-sm font-medium text-gray-700">Unit Price Per Qty</label>
                <input id="unit_price_per_qty" type="number" step="0.01" bind:value={updateData.unit_price_per_qty} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>

            <div>
                <label for="cus_po_no" class="block text-sm font-medium text-gray-700">PO No.</label>
                <input id="cus_po_no" type="text" bind:value={updateData.po_no} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>

            <div>
                <label for="po_date" class="block text-sm font-medium text-gray-700">PO Date</label>
                <input id="po_date" type="date" bind:value={updateData.po_date} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>
            <div>
                <label for="cus_invoice_no" class="block text-sm font-medium text-gray-700">Invoice No.</label>
                <input id="cus_invoice_no" type="text" bind:value={updateData.invoice_no} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>

            <div>
                <label for="cus_invoice_date" class="block text-sm font-medium text-gray-700">Invoice Date</label>
                <input id="cus_invoice_date" type="date" bind:value={updateData.invoice_date} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>

         

         

            <div>
                <label for="received_date" class="block text-sm font-medium text-gray-700">Recieved Date</label>
                <input id="received_date" type="date" bind:value={updateData.received_date} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>
           

            <div>
                <label for="warranty" class="block text-sm font-medium text-gray-700">Warranty</label>
                <input id="warranty" type="text" bind:value={updateData.warranty} class="w-full p-2 border border-gray-300 rounded-md" />
            </div>

          
        </div>

        <div class="mt-6 flex justify-end gap-4">
            <button on:click={() => showUpdateModal = false} class="px-4 py-2 bg-gray-300 text-gray-800 rounded-md hover:bg-gray-400">Cancel</button>
       <button on:click={updateInwardData} class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700">Save</button> 
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



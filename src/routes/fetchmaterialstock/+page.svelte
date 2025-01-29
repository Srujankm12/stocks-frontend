<script>
    import { onMount } from "svelte";
    import Header from "$lib/header.svelte";
    import { fade } from "svelte/transition";

    let data = [];
    let isLoading = true;
    let selectedRow = null;
    let showUpdateModal = false;
    let updateData = {
        id: "",
        supplier: "",
        category: "",
        lead_time: "",
        std_non_std: "",
        part_code: "",
        unit: "",
        rate: "",
        minimum_retain: "",
        maximum_retain: "",
        received: "",
        issue: "",
        reserved_stock: ""
    };
  
    let isUpdating = false;

    const apiUrl = "http://localhost:8000/materialstockdata";
    const updateUrl = "http://localhost:8000/materialupdate";


    let suppliers = [];
    let categories = [];
    let units = [];

    const fetchDropdownData = async () => {
        try {
            const response = await fetch('http://localhost:8000/materialstockdropdown');
            if (response.ok) {
                const data = await response.json();

                suppliers = Array.from(new Set(data.map(item => item.supplier)));
                categories = Array.from(new Set(data.map(item => item.category)));
                units = Array.from(new Set(data.map(item => item.unit)));
            } else {
                console.error('Error fetching dropdown data:', response.statusText);
            }
        } catch (error) {
            console.error('Error fetching dropdown data:', error);
        }
    };
    fetchDropdownData();
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

    function openUpdateModal(row) {
        // Populate updateData with the selected row values
        updateData = {
            id: row.id,
            supplier: row.supplier,
            category: row.category,
            lead_time: row.lead_time,
            std_non_std: row.std_non_std,
            part_code: row.part_code,
            unit: row.unit,
            rate: row.rate,
            minimum_retain: row.minimum_retain,
            maximum_retain: row.maximum_retain,
            received: row.received,
            issue: row.issue,
            reserved_stock: row.reserved_stock
        };
        selectedRow = row; // Set selected row
        showUpdateModal = true;
    }

    async function updateStock(event) {
        event.preventDefault();
        isUpdating = true;
        try {
            // Send only the updated field (e.g., part_code)
            const updatedFields = {};
            // Only include the fields that are changed
            Object.keys(updateData).forEach(key => {
                if (updateData[key] !== selectedRow[key]) {
                    updatedFields[key] = updateData[key];
                }
            });

            // If no fields were updated, do nothing
            if (Object.keys(updatedFields).length === 0) {
                console.log("No fields updated.");
                return;
            }

            // Construct the payload with the unchanged data plus the updated field
            const updatedData = {
                ...selectedRow, // Keep the existing data
                ...updatedFields, // Override with updated fields
            };

            // Send the full data with the updated field
            const response = await fetch(updateUrl, {
                method: "POST", // Use POST method for updates
                body: JSON.stringify(updatedData), // Send all fields with the updated data
               
            });

            if (response.ok) {
                console.log("Stock updated successfully");
                showUpdateModal = false;
                selectedRow = null; // Clear selected row
                await fetchData(); // Optionally fetch the updated data here
            } else {
                console.error("Failed to update stock:", response.statusText);
            }
        } catch (error) {
            console.error("Error updating stock:", error);
        } finally {
            isUpdating = false;
        }
    }

    async function fetchData() {
        try {
            const response = await fetch(apiUrl);
            if (response.ok) {
                const jsonResponse = await response.json();
                data = Array.isArray(jsonResponse) ? jsonResponse : [jsonResponse];
                console.log(data);
            } else {
                console.error("Failed to fetch data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        } finally {
            isLoading = false;
        }
    }

    onMount(fetchData);
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
                            <th class="py-3 px-4">ID</th>
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
                            {#each data as row}
                                <tr class="border-t border-gray-300 hover:bg-gray-200">
                                    <td class="py-3 px-4 whitespace-nowrap">{row.id}</td>
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
                                        {#if row.reorder_status === true || row.reorder_status === "true"}
                                            Reorder Needed
                                        {:else}
                                            In Stock
                                        {/if}
                                    </td>
                                    <td class="py-3 px-4 text-center">{row.excess_stock}</td>
                                    <td class="py-3 px-4 text-center">₹{row.excess_stock_value}</td>
                                    <td class="py-3 px-4 text-center">
                                        <!-- svelte-ignore a11y_consider_explicit_label -->
                                        <button
                                            class="text-xl font-medium rounded-full flex items-center justify-center"
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
        {/if}           
    </main>

    {#if showUpdateModal}
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center" transition:fade>
            <div class="w-full max-w-2xl mx-4 bg-white rounded-lg p-6 shadow-md relative">
                <h1 class="text-center text-xl py-2 mb-6 font-medium text-gray-900">Update Stock</h1>

                <form on:submit|preventDefault={updateStock}>
                    <div class="grid grid-cols-2 gap-4 mb-6">
                        <div>
                            <label for="supplier" class="block text-sm font-medium text-gray-700">Supplier</label>
                            <select
                                
                                bind:value={updateData.supplier}
                                class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                                required
                            >
                                <option value="" disabled>Select a supplier</option>
                                {#each suppliers as supplier}
                                    <option value={supplier}>{supplier}</option>
                                {/each}
                            </select>
                        </div>
                        <div>
                            <label for="category" class="block text-sm font-medium text-gray-700">Category</label>
                            <select
                                id="category"
                                bind:value={updateData.category}
                                class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                                required
                            >
                                <option value="" disabled>Select a category</option>
                                {#each categories as category}
                                    <option value={category}>{category}</option>
                                {/each}
                            </select>
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Lead Time</label>
                            <input type="number" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.lead_time} required />
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Std/Non-Std</label>
                            <input type="text" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.std_non_std} required />
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Part Code</label>
                            <input type="text" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.part_code} required />
                        </div>
                        <div>
                            <label for="unit" class="block text-sm font-medium text-gray-700">Unit</label>
                            <select
                                id="unit"
                                bind:value={updateData.unit}
                                class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                                required
                            >
                                <option value="" disabled>Select a Unit</option>
                                {#each units as unit}
                                    <option value={unit}>{unit}</option>
                                {/each}
                            </select>
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Rate</label>
                            <input type="number" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.rate} required />
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Minimum Retain</label>
                            <input type="number" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.minimum_retain} required />
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Maximum Retain</label>
                            <input type="number" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.maximum_retain} required />
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Received</label>
                            <input type="number" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.received} required />
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Issue</label>
                            <input type="number" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.issue} required />
                        </div>
                        <div>
                            <!-- svelte-ignore a11y_label_has_associated_control -->
                            <label class="block text-gray-700">Reserved Stock</label>
                            <input type="number" class="w-full px-4 py-2 border rounded-md" bind:value={updateData.reserved_stock} required />
                        </div>
                    </div>
                    <div class="flex justify-end">
                        <button type="submit" class="px-6 py-2 bg-black text-white rounded-md" disabled={isUpdating}>
                            {#if isUpdating}
                                Updating...
                            {:else}
                                Update Stock
                            {/if}
                        </button>
                    </div>
                </form>
                <!-- svelte-ignore a11y_consider_explicit_label -->
                <button
                    class="absolute top-4 right-4 text-gray-500"
                    on:click={() => showUpdateModal = false}
                >
                    <i class="fas fa-times"></i>
                </button>
            </div>
        </div>
    {/if}
</div>

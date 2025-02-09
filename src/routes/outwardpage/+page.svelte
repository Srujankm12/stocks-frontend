<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import Header from '$lib/header.svelte';

  let dropdownData = writable({
    sellers: [],
    branchRegions: [],
    issuesAgainst: [],
    categories: [],
  });

  let formData = writable({
    timestamp: '',
    customer: '',
    seller: '',
    branch_region: '',
    partcode: '',
    serial_number: '',
    qty: 0,
    cus_po_no: '',
    cus_po_date: '',
    cus_invoice_no: '',
    cus_invoice_date: '',
    delivery_date: '',
    unit_price_per_qty: 0.0,
    issue_against: '',
    notes: '',
    category: '',
    warranty: 0,
    warranty_due_days: 0,
  });

  let isLoading = false;
  let errorMessage = '';

  const fetchDropdownData = async () => {
    isLoading = true;
    try {
      const response = await fetch('https://stocks-backend-t2bh.onrender.com/outwardDropdown', { method: 'GET' });
      if (response.ok) {
        const data = await response.json();
        dropdownData.set({
          sellers: [...new Set(data.map((item) => item.seller))],
          branchRegions: [...new Set(data.map((item) => item.branch_region))],
          issuesAgainst: [...new Set(data.map((item) => item.issue_against))],
          categories: [...new Set(data.map((item) => item.category))],
        });
      } else {
        errorMessage = `Failed to fetch dropdown data: ${response.statusText}`;
      }
    } catch (error) {
      errorMessage = `Error fetching dropdown data: ${error.message}`;
    } finally {
      isLoading = false;
    }
  };

  const setTimestamp = () => {
    const currentTime = new Date().toLocaleString('en-IN', { timeZone: 'Asia/Kolkata' });
    formData.update((data) => ({ ...data, timestamp: currentTime }));
  };

  const handleSubmit = async (event) => {
    event.preventDefault();
    setTimestamp();
    const payload = $formData;

    isLoading = true;
    try {
      const response = await fetch('https://stocks-backend-t2bh.onrender.com/submitoutward', {
        method: 'POST',

        body: JSON.stringify(payload),
      });

      if (response.ok) {
        console.log('Form submitted successfully:', await response.json());
        formData.set({
          timestamp: '',
          customer: '',
          seller: '',
          branch_region: '',
          partcode: '',
          serial_number: '',
          qty: 0,
          cus_po_no: '',
          cus_po_date: '',
          cus_invoice_no: '',
          cus_invoice_date: '',
          delivery_date: '',
          unit_price_per_qty: 0.0,
          issue_against: '',
          notes: '',
          category: '',
          warranty: 0,
          warranty_due_days: 0,
        });
        errorMessage = '';
      } else {
        errorMessage = `Form submission failed: ${await response.text()}`;
      }
    } catch (error) {
      errorMessage = `Error during form submission: ${error.message}`;
    } finally {
      isLoading = false;
    }
  };

  onMount(() => {
    fetchDropdownData();
  });
</script>
<div class="min-h-screen flex flex-col bg-white">
  <Header />
  

  <main class="flex-grow container mx-auto px-6 py-20 grid lg:grid-cols-2 gap-10">
    <div class="bg-white shadow-lg rounded-lg p-8 flex flex-col justify-start items-start space-y-6">
  <div class="w-full overflow-hidden rounded-lg">
    <img src="image.webp" alt="Material Tracking" class="w-full h-auto mb-4 object-cover transition-transform duration-500 hover:scale-105" />
  </div>
  <h2 class="text-3xl font-semibold text-gray-900">Why Choose Us?</h2>
  <ul class="space-y-4 text-gray-700 text-lg">
    <li class="flex items-start">
      <span class="text-black font-bold mr-3">•</span>
      Automate data collection for inventory accuracy.
    </li>
    <li class="flex items-start">
      <span class="text-black font-bold mr-3">•</span>
      Centralize supplier and buyer details for quick access.
    </li>
    <li class="flex items-start">
      <span class="text-black font-bold mr-3">•</span>
      Ensure compliance with precise record keeping.
    </li>
    <li class="flex items-start">
      <span class="text-black font-bold mr-3">•</span>
      Optimize cost tracking with unit price details.
    </li>
  </ul>
</div>

    <form on:submit={handleSubmit} class="bg-white shadow-xl rounded-lg p-8 space-y-8">
      <h2 class="text-3xl font-semibold text-gray-900 mb-8">Enter Material Outward Details</h2>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label for="customer" class="block text-sm font-medium text-gray-700">Customer</label>
          <input id="customer" 
          placeholder="Enter Customer Name"
          type="text" bind:value={$formData.customer} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
        <div>
          <label for="seller" class="block text-sm font-medium text-gray-700">Seller</label>
          <select id="seller" bind:value={$formData.seller} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required>
            <option value="" disabled>Select a seller</option>
            {#each $dropdownData.sellers as seller}
              <option value={seller}>{seller}</option>
            {/each}
          </select>
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label for="branch_region" class="block text-sm font-medium text-gray-700">Branch Region</label>
          <select id="branch_region" bind:value={$formData.branch_region} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required>
            <option value="" disabled>Select a branch region</option>
            {#each $dropdownData.branchRegions as region}
              <option value={region}>{region}</option>
            {/each}
          </select>
        </div>
        <div>
          <label for="partcode" class="block text-sm font-medium text-gray-700">Partcode</label>
          <input id="partcode"
          placeholder="Enter Partcode"
           type="text" bind:value={$formData.partcode} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label for="serial_number" class="block text-sm font-medium text-gray-700">Serial Number</label>
          <input id="serial_number" 
          placeholder="Enter Serial Number"
          type="text" bind:value={$formData.serial_number} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
        <div>
          <label for="qty" class="block text-sm font-medium text-gray-700">Quantity</label>
          <input id="qty" type="number" bind:value={$formData.qty} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label for="cus_po_no" class="block text-sm font-medium text-gray-700">Customer PO Number</label>
          <input id="cus_po_no" 
          placeholder="Enter Customer PO Number"
          type="text" bind:value={$formData.cus_po_no} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
        <div>
          <label for="cus_po_date" class="block text-sm font-medium text-gray-700">Customer PO Date</label>
          <input id="cus_po_date"
          placeholder="Enter Customer PO Date"
           type="date" bind:value={$formData.cus_po_date} 
          class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
        <div>
          <label for="cus_invoice_no" class="block text-sm font-medium text-gray-700">Customer Invoice Number</label>
          <input id="cus_invoice_no" 
          placeholder="Enter Customer Invoice Number"
          type="text" bind:value={$formData.cus_invoice_no} 
          class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
        <div>
          <label for="cus_invoice_date" class="block text-sm font-medium text-gray-700">Customer Invoice Date</label>
          <input id="cus_invoice_date" type="date" bind:value={$formData.cus_invoice_date} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg" required />
        </div>
        <div>
          <label for="delivery_date" class="block text-sm font-medium text-gray-700">Delivery Date</label>
          <input id="delivery_date" type="date" bind:value={$formData.delivery_date}
          class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
        <div>
          <label for="unit_price_per_qty" class="block text-sm font-medium text-gray-700">Unit Price Per Quantity</label>
          <input id="unit_price_per_qty" type="number" step="0.01" bind:value={$formData.unit_price_per_qty} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg" required />
        </div>
  
        <div>
          <label for="issue_against" class="block text-sm font-medium text-gray-700">Issue Against</label>
          <select id="issue_against" bind:value={$formData.issue_against} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg" required>
            <option value="" disabled>Select an issue type</option>
            {#each $dropdownData.issuesAgainst as issue}
              <option value={issue}>{issue}</option>
            {/each}
          </select>
        </div>
        <div>
          <label for="notes" class="block text-sm font-medium text-gray-700">Notes</label>
          <textarea id="notes" bind:value={$formData.notes} 
          placeholder="Enter Notes"
          class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg" rows="3"></textarea>
        </div>
        
        <div>
          <label for="category" class="block text-sm font-medium text-gray-700">Category</label>
          <select
              id="category"
              bind:value={$formData.category}
              class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
              required
          >
              <option value="" disabled>Select a category</option>
              {#each $dropdownData.categories as category}
                  <option value={category}>{category}</option>
              {/each}
          </select>
      </div>
  
        <div>
          <label for="warranty" class="block text-sm font-medium text-gray-700">Warranty</label>
          <input id="warranty" type="number" bind:value={$formData.warranty} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg" />
        </div>
  
        {#if $formData.warranty === 0}
          <div>
            <label for="warranty_due_days" class="block text-sm font-medium text-gray-700">Warranty Due Days</label>
            <input id="warranty_due_days" type="number" bind:value={$formData.warranty_due_days} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg" />
          </div>
        {/if}
  
      
      </div>
   
     

      <div class="text-center">
        <button
        type="submit"
        class="px-8 py-3 bg-black text-white text-xl font-semibold rounded-lg hover:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-black transition duration-300"
    >
        Submit
    </button>
      </div>
      </form>
  </main>
</div>

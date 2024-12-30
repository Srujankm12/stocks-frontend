<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  // Example of creating a writable store
  let count = writable(0);

  // Function to increment the count
  function increment() {
    count.update(n => n + 1);
  }

  // Store to hold form data
  let formData = writable({
    id: null,
    timestamp: '',
    supplier: '',
    buyer: '',
    partcode: '',
    serial_number: '',
    qty: 0,
    po_no: '',
    po_date: '',
    invoice_no: '',
    invoice_date: '',
    received_date: '',
    unit_price_per_qty: 0.0,
    category: '',
    warranty: 0,
    warranty_due_days: 0,
  });

  // Function to fetch suppliers and buyers (optional, depending on your form)
  let suppliers = writable([]);
  let buyers = writable([]);

  const fetchSuppliersAndBuyers = async () => {
    try {
      const response = await fetch('http://localhost:8000/inward', { method: 'GET' });

      if (response.ok) {
        const data = await response.json();
        const uniqueSuppliers = [...new Set(data.map(item => item.supplier))];
        const uniqueBuyers = [...new Set(data.map(item => item.buyer))];

        suppliers.set(uniqueSuppliers);
        buyers.set(uniqueBuyers);

        console.log('Suppliers:', uniqueSuppliers);
        console.log('Buyers:', uniqueBuyers);
      } else {
        const errorData = await response.json();
        console.error('Error fetching data:', errorData.message);
      }
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  };

  // Function to set timestamp when the form is submitted
  const setTimestamp = () => {
    // Get current date in Bengaluru time zone (Asia/Kolkata)
    const bengaluruTime = new Date().toLocaleString('en-IN', { timeZone: 'Asia/Kolkata' });

    formData.update((data) => ({
      ...data,
      timestamp: bengaluruTime, // Set the timestamp in Bengaluru time
    }));
  };

  // Submit handler to set timestamp before submitting and clear form on success
  const handleSubmit = async (event) => {
    event.preventDefault(); // Prevent default form submission
    setTimestamp(); // Set the timestamp before submission

    // Get the current form data from the store
    const formPayload = $formData;

    try {
      const response = await fetch('http://localhost:8000/submit', {
        method: 'POST',
       
        body: JSON.stringify(formPayload),
      });

      if (response.ok) {
        const responseData = await response.json();
        console.log('Data submitted successfully:', responseData);

        // Clear the form data after successful submission
        formData.set({
          id: null,
          timestamp: '',
          supplier: '',
          buyer: '',
          partcode: '',
          serial_number: '',
          qty: 0,
          po_no: '',
          po_date: '',
          invoice_no: '',
          invoice_date: '',
          received_date: '',
          unit_price_per_qty: 0.0,
          category: '',
          warranty: 0,
          warranty_due_days: 0,
        });

        // Optionally, reset other relevant stores (e.g., suppliers, buyers) if needed
        suppliers.set([]);
        buyers.set([]);
      } else {
        const errorData = await response.json();
        console.error('Error submitting data:', errorData);
      }
    } catch (error) {
      console.error('Error during submission:', error);
    }
  };

  onMount(() => {
    fetchSuppliersAndBuyers();
  });
</script>


<div class="min-h-screen flex flex-col bg-white">
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
  <main class="flex-grow container mx-auto px-6 py-12 grid lg:grid-cols-2 gap-10">
    <!-- Left Section: Info and Image -->
    <div class="bg-white shadow-lg rounded-lg p-8 flex flex-col justify-center items-start">
      <img src="image.webp" alt="Material Tracking" class="w-full h-full mb-4 rounded-lg shadow-lg object-cover hover:scale-105 transition-transform duration-500" />
      <h2 class="text-3xl font-semibold text-gray-900 mb-6">Why Choose Us?</h2>
      <ul class="space-y-4 text-gray-700 text-lg">
        <li class="flex items-start">
          <span class="text-orange-500 font-bold mr-3">•</span>
          Automate data collection for inventory accuracy.
        </li>
        <li class="flex items-start">
          <span class="text-orange-500 font-bold mr-3">•</span>
          Centralize supplier and buyer details for quick access.
        </li>
        <li class="flex items-start">
          <span class="text-orange-500 font-bold mr-3">•</span>
          Ensure compliance with precise record keeping.
        </li>
        <li class="flex items-start">
          <span class="text-orange-500 font-bold mr-3">•</span>
          Optimize cost tracking with unit price details.
        </li>
      </ul>
    </div>

    <form on:submit={handleSubmit} class="bg-white shadow-xl rounded-lg p-8 space-y-8 transition-shadow duration-300">
      <h2 class="text-3xl font-semibold text-gray-900 mb-8">Enter Material Details</h2>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div>
          <label for="supplier" class="block text-sm font-medium text-gray-800">Supplier</label>
          <select id="supplier" bind:value={$formData.supplier} class="mt-2 block w-full rounded-lg border-orange-50 shadow-sm " required>
            <option value="" disabled>Select a supplier</option>
            {#each $suppliers as supplier}
              <option value={supplier}>{supplier}</option>
            {/each}
          </select>
        </div>
        <div>
          <label for="buyer" class="block text-sm font-medium text-gray-800">Buyer</label>
          <select id="buyer" bind:value={$formData.buyer} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm " required>
            <option value="" disabled>Select a buyer</option>
            {#each $buyers as buyer}
              <option value={buyer}>{buyer}</option>
            {/each}
          </select>
        </div>
      </div>


      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div>
          <label for="partcode" class="block text-sm font-medium text-gray-800">Partcode</label>
          <input 
            id="partcode" 
            type="text" 
            bind:value={$formData.partcode} 
            class="mt-2 block w-full rounded-lg  shadow-sm focus:ring-2 focus:ring-orange-500 focus:border-orange-500" 
            required  
          />
        </div>
        <div>
          <label for="serial_number" class="block text-sm font-medium text-gray-800">Serial Number</label>
          <input 
            id="serial_number" 
            type="text" 
            bind:value={$formData.serial_number} 
            class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500 focus:border-orange-500" 
            required 
          />
        </div>
      </div>
      
      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div>
          <label for="qty" class="block text-sm font-medium text-gray-800">Quantity</label>
          <input id="qty" type="number" bind:value={$formData.qty} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" required />
        </div>
        <div>
          <label for="unit_price_per_qty" class="block text-sm font-medium text-gray-800">Unit Price (per Qty)</label>
          <input id="unit_price_per_qty" type="number" step="0.01" bind:value={$formData.unit_price_per_qty} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" required />
        </div>
      </div>
  
      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div>
          <label for="po_no" class="block text-sm font-medium text-gray-800">P.O. Number</label>
          <input id="po_no" type="text" bind:value={$formData.po_no} class="mt-2 block w-full rounded-lg border-orange-300 shadow-sm focus:ring-2 focus:ring-orange-500" />
        </div>
        <div>
          <label for="po_date" class="block text-sm font-medium text-gray-800">P.O. Date</label>
          <input id="po_date" type="date" bind:value={$formData.po_date} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" />
        </div>
        <div>
          <label for="invoice_no" class="block text-sm font-medium text-gray-800">Invoice Number</label>
          <input id="invoice_no" type="text" bind:value={$formData.invoice_no} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" />
        </div>
        <div>
          <label for="invoice_date" class="block text-sm font-medium text-gray-800">Invoice Date</label>
          <input id="invoice_date" type="date" bind:value={$formData.invoice_date} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" />
        </div>
      </div>


      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div>
          <label for="received_date" class="block text-sm font-medium text-gray-800">Received Date</label>
          <input id="received_date" type="date" bind:value={$formData.received_date} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" />
        </div>
        <div>
          <label for="category" class="block text-sm font-medium text-gray-800">Category</label>
          <input id="category" type="text" bind:value={$formData.category} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" />
        </div>
      </div>

 
      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div>
          <label for="warranty" class="block text-sm font-medium text-gray-800">Warranty</label>
          <input id="warranty" type="number" bind:value={$formData.warranty} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" />
        </div>
        {#if $formData.warranty === 0}
          <div>
            <label for="warranty_due_days" class="block text-sm font-medium text-gray-800">Warranty Due (days)</label>
            <input id="warranty_due_days" type="number" bind:value={$formData.warranty_due_days} class="mt-2 block w-full rounded-lg border-gray-300 shadow-sm focus:ring-2 focus:ring-orange-500" />
          </div>
        {/if}
      </div>

      <!-- Submit Button -->
      <div class="text-center">
        <button type="submit" class="px-6 py-2 bg-gradient-to-r from-orange-600 to-orange-500 text-white rounded-lg shadow-lg hover:bg-gradient-to-r hover:from-orange-700 hover:to-orange-600">
          Submit
        </button>
      </div>
    </form>
  </main>
</div>



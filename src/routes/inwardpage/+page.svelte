<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import Header from '$lib/header.svelte';
	import { Ear } from 'lucide-svelte';

  let count = writable(0);

  function increment() {
    count.update(n => n + 1);
  }

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

  let suppliers = writable([]);
  let buyers = writable([]);
  let categories = writable([]);

  const fetchSuppliersAndBuyers = async () => {
    try {
      const response = await fetch('http://localhost:8000/inward', { method: 'GET' });

      if (response.ok) {
        const data = await response.json();
        const uniqueSuppliers = [...new Set(data.map(item => item.supplier))];
        const uniqueBuyers = [...new Set(data.map(item => item.buyer))];
        const uniqueCategories = [...new Set(data.map(item => item.category))];

        suppliers.set(uniqueSuppliers);
        buyers.set(uniqueBuyers);
        categories.set(uniqueCategories);

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

  const setTimestamp = () => {
    const bengaluruTime = new Date().toLocaleString('en-IN', { timeZone: 'Asia/Kolkata' });

    formData.update((data) => ({
      ...data,
      timestamp: bengaluruTime, 
    }));
  };

  const handleSubmit = async (event) => {
    event.preventDefault();
    setTimestamp();

  
    const formPayload = $formData;

    try {
      const response = await fetch('http://localhost:8000/submit', {
        method: 'POST',
       
        body: JSON.stringify(formPayload),
      });

      if (response.ok) {
        const responseData = await response.json();
        console.log('Data submitted successfully:', responseData);

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

  

    <form on:submit={handleSubmit} class="bg-white shadow-xl rounded-lg p-8 space-y-8 transition-shadow duration-300">
      <h2 class="text-3xl font-semibold text-gray-900 mb-8">Enter Material Inward Details</h2>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label for="supplier" class="block text-sm font-medium text-gray-700">Supplier</label>
          <select id="supplier" bind:value={$formData.supplier} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none">
            <option value="" disabled>Select a supplier</option>
            {#each $suppliers as supplier}
              <option value={supplier}>{supplier}</option>
            {/each}
          </select>
        </div>
        <div>
          <label for="buyer" class="block text-sm font-medium text-gray-700">Buyer</label>
          <select id="buyer" bind:value={$formData.buyer} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none">
            <option value="" disabled>Select a buyer</option>
            {#each $buyers as buyer}
              <option value={buyer}>{buyer}</option>
            {/each}
          </select>
        </div>
     </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label for="partcode" class="block text-sm font-medium text-gray-700">Partcode</label>
          <input id="partcode"
           type="text"
           placeholder="Enter Partcode"

           bind:value={$formData.partcode} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
        <div>
          <label for="serial_number" class="block text-sm font-medium text-gray-700">Serial Number</label>
          <input id="serial_number" type="text"
          placeholder="Enter Serial Number"
           bind:value={$formData.serial_number} class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
      </div>
      
      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div>
          <label for="qty" class="block text-sm font-medium text-gray-800">Quantity</label>
          <input 
            id="qty" 
            type="number" 
            bind:value={$formData.qty} 
            class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
            required 
          />
        </div>
      
        <div>
          <label for="unit_price_per_qty" class="block text-sm font-medium text-gray-800">Unit Price (per Qty)</label>
          <input 
            id="unit_price_per_qty" 
            type="number" 
            step="0.01" 
            bind:value={$formData.unit_price_per_qty} 
            class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
            required 
          />
        </div>
      </div>
      
  
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-2 gap-8">
        <div>
          <label for="po_no" class="block text-sm font-medium text-gray-800">P.O. Number</label>
          <input 
            id="po_no" 
            placeholder="Enter P.O. Number"
            type="text" 
            bind:value={$formData.po_no} 
            class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
          />
        </div>
      
        <div>
          <label for="po_date" class="block text-sm font-medium text-gray-800">P.O. Date</label>
          <input 
            id="po_date" 
            type="date" 
            placeholder="Enter P.O. Date"
            bind:value={$formData.po_date} 
            class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
          />
        </div>
      
        <div>
          <label for="invoice_no" class="block text-sm font-medium text-gray-800">Invoice Number</label>
          <input 
            id="invoice_no" 
            type="text" 
            placeholder="Enter Invoice Number"
            bind:value={$formData.invoice_no} 
            class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
          />
        </div>
      
        <div>
          <label for="invoice_date" class="block text-sm font-medium text-gray-800">Invoice Date</label>
          <input 
            id="invoice_date" 
            type="date" 
            bind:value={$formData.invoice_date} 
            class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
          />
        </div>
      </div>
      


      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-2 gap-8">
        <div>
          <label for="received_date" class="block text-sm font-medium text-gray-800">Received Date</label>
          <input 
            id="received_date" 
            type="date" 
            bind:value={$formData.received_date} 
            class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
          />
        </div>
      
        <div>
          <label for="category" class="block text-sm font-medium text-gray-700">Category</label>
          <select
            id="category"
            bind:value={$formData.category}
            class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
          >
            <option value="" disabled>Select a category</option>
            {#each $categories as category}
              <option value={category}>{category}</option>
            {/each}
          </select>
        </div>
        
      </div>


      

      
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-2 gap-8">
        <div>
          <label for="warranty" class="block text-sm font-medium text-gray-800">Warranty</label>
          <input 
            id="warranty" 
            type="number" 
            bind:value={$formData.warranty} 
            class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
          />
        </div>
      
        {#if $formData.warranty === 0}
          <div>
            <label for="warranty_due_days" class="block text-sm font-medium text-gray-800">Warranty Due (days)</label>
            <input 
              id="warranty_due_days" 
              type="number" 
              bind:value={$formData.warranty_due_days} 
              class="mt-2 block w-full rounded-lg border border-gray-300 shadow-sm focus:ring-2 focus:ring-black focus:outline-none text-gray-800 px-4 py-2" 
              />
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



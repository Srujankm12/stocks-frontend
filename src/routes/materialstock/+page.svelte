<script>
    import Header from '$lib/header.svelte';
    import { writable } from 'svelte/store';

   
    let formData = writable({
        supplier: '',
        category: '',
        lead_time: '',
        std_non_std: '',
        part_code: '',
        unit: '',
        rate: '',
        minimum_retain: '',
        maximum_retain: '',
        received: '',
        issue: '',
        reserved_stock: '',
    });


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

 
    const handleSubmit = async (event) => {
        event.preventDefault();  

        console.log('Form Submitted', $formData);  

        try {
      
            const response = await fetch('http://localhost:8000/materialstock', {
                method: 'POST',
                
             
                body: JSON.stringify($formData),
            });

            if (response.ok) {
             
                formData.set({
                    supplier: '',
                    category: '',
                    lead_time: '',
                    std_non_std: '',
                    part_code: '',
                    unit: '',
                    rate: '',
                    minimum_retain: '',
                    maximum_retain: '',
                    received: '',
                    issue: '',
                    reserved_stock: '',
                });

                const responseData = await response.json();
                console.log('Data submitted successfully:', responseData);
            } else {
                const errorData = await response.json();
                console.error('Error submitting form:', errorData);
            }
        } catch (error) {
            console.error('Error submitting form:', error);
        }
    };
</script>

<div class="min-h-screen flex flex-col bg-white">
    <Header />

    <main class="flex-grow container mx-auto px-6 py-20 grid lg:grid-cols-2 gap-10">
        <div class="bg-white shadow-lg rounded-lg p-8 flex flex-col justify-start items-start space-y-6">
            <div class="w-full overflow-hidden rounded-lg">
                <img
                    src="image.webp"
                    alt="Material Tracking"
                    class="w-full h-auto mb-4 object-cover transition-transform duration-500 hover:scale-105"
                />
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
            <h2 class="text-3xl font-semibold text-gray-900 mb-8">Enter Material Stock</h2>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        
                <div>
                    <label for="supplier" class="block text-sm font-medium text-gray-700">Supplier</label>
                    <select
                        id="supplier"
                        bind:value={$formData.supplier}
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
                        bind:value={$formData.category}
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
                    <label for="lead_time" class="block text-sm font-medium text-gray-700">Lead Time</label>
                    <input
                        placeholder="Enter lead time"
                        id="lead_time"
                        type="number"
                        bind:value={$formData.lead_time}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

   
                <div>
                    <label for="std_non_std" class="block text-sm font-medium text-gray-700">Std/Non-Std</label>
                    <input
                        placeholder="Enter Std/Non-Std"
                        id="std_non_std"
                        type="text"
                        bind:value={$formData.std_non_std}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

      
                <div>
                    <label for="part_code" class="block text-sm font-medium text-gray-700">Part Code</label>
                    <input
                        placeholder="Enter Part Code"
                        id="part_code"
                        type="text"
                        bind:value={$formData.part_code}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

   
                <div>
                    <label for="unit" class="block text-sm font-medium text-gray-700">Unit</label>
                    <select
                        placeholder="Enter Unit"
                        id="unit"
                        bind:value={$formData.unit}
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
                    <label for="rate" class="block text-sm font-medium text-gray-700">Rate</label>
                    <input
                        placeholder="Enter Rate"
                        id="rate"
                        type="number"
                        bind:value={$formData.rate}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="minimum_retain" class="block text-sm font-medium text-gray-700">Minimum Retain</label>
                    <input
                        placeholder="Enter Minimum Retain"
                        id="minimum_retain"
                        type="number"
                        bind:value={$formData.minimum_retain}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="maximum_retain" class="block text-sm font-medium text-gray-700">Maximum Retain</label>
                    <input
                        placeholder="Enter Maximum Retain"
                        id="maximum_retain"
                        type="number"
                        bind:value={$formData.maximum_retain}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

               
                <div>
                    <label for="received" class="block text-sm font-medium text-gray-700">Received</label>
                    <input
                        placeholder="Enter Received"
                        id="received"
                        type="number"
                        bind:value={$formData.received}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

                <div>
                    <label for="issue" class="block text-sm font-medium text-gray-700">Issue</label>
                    <input
                        placeholder="Enter Issue"
                        id="issue"
                        type="number"
                        bind:value={$formData.issue}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>

       
                <div>
                    <label for="reserved_stock" class="block text-sm font-medium text-gray-700">Reserved Stock</label>
                    <input
                        placeholder="Enter Reserved Stock"
                        id="reserved_stock"
                        type="number"
                        bind:value={$formData.reserved_stock}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                        required
                    />
                </div>
            </div>

         
            <div class="flex justify-center mt-6">
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

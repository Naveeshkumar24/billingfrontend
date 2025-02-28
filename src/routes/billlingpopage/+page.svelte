<script>
    import { onMount } from 'svelte';
    import { writable, get } from 'svelte/store';
    import Header from '$lib/header.svelte';

    let formData = writable({
        engg_Name: '',
        supplier: '',
        bill_No: '',
        bill_Date: '',
        customer_Name: '',
        customer_Po_No: '',
        customer_Po_Date: '',
        item_Description: '',
        billed_Qty: '',
        unit: '',
        net_Value: '',
        cgst: '',
        igst: '',
        dispatch_through: ''
    });

    let enggNames = writable([]);
    let suppliers = writable([]);
    let customers = writable([]);
    let units = writable([]);

    // Fetch dropdown data
    const fetchDropdownData = async () => {
        try {
            const response = await fetch('https://srbilling.onrender.com/dropdown');
            if (response.ok) {
                const data = await response.json();
                enggNames.set([...new Set(data.map(item => item.engg_name))]);
                suppliers.set([...new Set(data.map(item => item.supplier_name))]);
                customers.set([...new Set(data.map(item => item.customer_name))]);
                units.set([...new Set(data.map(item => item.unit_name))]);
            } else {
                console.error('Error fetching dropdown data:', response.statusText);
            }
        } catch (error) {
            console.error('Error fetching dropdown data:', error);
        }
    };

    onMount(fetchDropdownData);

    // Handle form submission
    const handleSubmit = async (event) => {
        event.preventDefault();
        let formDataValue = get(formData);

        try {
            const response = await fetch('https://srbilling.onrender.com/submit', {
                method: 'POST', 
               
                body: JSON.stringify(formDataValue),
            });

            if (response.ok) {
                formData.set({
                    engg_Name: '',
                    supplier: '',
                    bill_No: '',
                    bill_Date: '',
                    customer_Name: '',
                    customer_Po_No: '',
                    customer_Po_Date: '',
                    item_Description: '',
                    billed_Qty: '',
                    unit: '',
                    net_Value: '',
                    cgst: '',
                    igst: '',
                    dispatch_through: ''
                });
                console.log('Data submitted successfully');
            } else {
                console.error('Error submitting form:', await response.json());
            }
        } catch (error) {
            console.error('Error submitting form:', error);
        }
    };
</script>

<div class="min-h-screen flex flex-col bg-white">
    <Header />
    <main class="flex-grow container mx-auto px-6 py-24 grid lg:grid-cols-1 gap-10">
        <form on:submit={handleSubmit} class="bg-white shadow-xl rounded-lg p-8 space-y-8">
            <h2 class="text-2xl font-semibold text-gray-900 mb-8">Billing PO Form</h2>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                
                <div>
                    <label for="engg_name" class="block text-sm font-medium text-gray-700">Engineer Name</label>
                    <select bind:value={$formData.engg_Name} required class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none">
                        <option value="" disabled>Select Engineer Name</option>
                        {#each $enggNames as name}
                            <option value={name}>{name}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="supplier" class="block text-sm font-medium text-gray-700">Supplier</label>
                    <select bind:value={$formData.supplier} required class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none">
                        <option value="" disabled>Select Supplier</option>
                        {#each $suppliers as supplier}
                            <option value={supplier}>{supplier}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="bill_No" class="block text-sm font-medium text-gray-700">Bill No</label>
                    <input type="text" 
                    id="bill_no"
                    bind:value={$formData.bill_No} 
                    placeholder="Enter Bill No"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required/>
                </div>

                <div>
                    <label for="bill_Date" class="block text-sm font-medium text-gray-700">Bill Date</label>
                    <input type="date" 
                    id="bill_date"
                    bind:value={$formData.bill_Date} 
                    placeholder="Enter Bill Date"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required/>
                </div>

                <div>
                    <label for="customer_Name" class="block text-sm font-medium text-gray-700">Customer Name</label>
                    <select bind:value={$formData.customer_Name} required class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none">
                        <option value="" disabled>Select Customer</option>
                        {#each $customers as customer}
                            <option value={customer}>{customer}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="customer_Po_No" class="block text-sm font-medium text-gray-700">Customer PO No</label>
                    <input type="text" id="customer_Po_no" bind:value={$formData.customer_Po_No} placeholder="Enter Customer Po No"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required/>
                </div>

                <div>
                    <label for="customer_Po_Date" class="block text-sm font-medium text-gray-700">Customer PO Date</label>
                    <input type="date" 
                    id="customer_Po_Date"
                    bind:value={$formData.customer_Po_Date} placeholder="Enter Customer PO Date"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required/>
                </div>

                <div>
                    <label for="item_Description" 
                     class="block text-sm font-medium text-gray-700">Item Description</label>
                    <input type="text"
                    id="item_Description"
                    bind:value={$formData.item_Description} 
                    placeholder="Enter Item Description"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required/>
                </div>

                <div>
                    <label for="billed_Qty" class="block text-sm font-medium text-gray-700">Billed Quantity</label>
                    <input type="number"
                    id="billed_Qty"
                     bind:value={$formData.billed_Qty}
                     placeholder="Enter Billed Quantity"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required/>
                </div>

                <div>
                    <label for="unit" class="block text-sm font-medium text-gray-700">Unit</label>
                    <select 
                    id="unit"
                    bind:value={$formData.unit}
                     placeholder="Enter Unit"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                    required>
                        <option value="" disabled>Select Unit</option>
                        {#each $units as unit}
                            <option value={unit}>{unit}</option>
                        {/each}
                    </select>
                </div>

                <div>
                    <label for="net_value" class="block text-sm font-medium text-gray-700">Net Value</label>
                    <input type="number" 
                    id="net_value"
                    bind:value={$formData.net_Value} 
                    placeholder="Enter Net Value"
                   class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                   required/>
                </div>

                <div>
                    <label for="cgst" class="block text-sm font-medium text-gray-700">CGST</label>
                    <input type="number" 
                    id="cgst"
                    bind:value={$formData.cgst} 
                    placeholder="Enter CGST "
                   class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                   required/>
                </div>

                <div>
                    <label for="igst"class="block text-sm font-medium text-gray-700">IGST</label>
                    <input type="number"
                    id="igst"
                    bind:value={$formData.igst} 
                    placeholder="Enter IGST"
                   class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                   required/>
                </div>


                

                <div>
                    <label for="dispatch_through" class="block text-sm font-medium text-gray-700">Dispatch Through</label>
                    <input type="text" 
                    id="dispatch_through"
                    bind:value={$formData.dispatch_through} 
                    placeholder="Enter Dispatach Through"
                   class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none"
                   required/>
                </div>

            </div>
            <div class="flex justify-center mt-6">
            <button type="submit" class="px-8 py-3 bg-black text-white text-xl font-semibold rounded-lg hover:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-black transition duration-300"
            >Submit</button>
        </form>
    </main>
</div>


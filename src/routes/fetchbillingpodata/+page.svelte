<script>
    import { onMount } from "svelte";
    import Header from "$lib/header.svelte";
    import { fade } from "svelte/transition";

    let data = [];
    let isLoading = true;
    let expandedRow = null;
    let showDownloadMessage = false;
    let showUpdateModal = false;
    let selectedRow = null;
    let customers = [];
    let bsNumbers = [];
    let suppliers=[];
    let enggNames=[];
    let units = [];
    let poStatuses = [];
    let concernsOnOrders = [];
    let filteredData = [];
    let searchQuery = "";
    let showDeleteModal = false;
    let rowToDelete = null;
    let UpdateData = {
        engg_name: '',
        supplier: '',
        bill_no: '',
        bill_date: '',
        customer_name: '',
        customer_po_no: '',
        customer_po_date: '',
        item_description: '',
        billed_qty: '',
        unit: '',
        net_value: '',
        cgst: '',
        igst: '',
        dispatch_through: '',
        gross:'',
        total_tax:'',

    };

    let isUpdating = false;
    const fetchurl = "https://srbilling.onrender.com/fetch";
    const downloadexcelurl = "https://srbilling.onrender.com/download";
    const updateurl = "https://srbilling.onrender.com/update";

    async function fetchDropdownData() {
        try {
            const response = await fetch('https://srbilling.onrender.com/dropdown');
            if (response.ok) {
                const data = await response.json();
                enggNames = [...new Set(data.map(item => item.engg_name))];
                suppliers=[...new Set(data.map(item => item.supplier_name))];
                customers=[...new Set(data.map(item => item.customer_name))];
                units=[...new Set(data.map(item => item.unit_name))];
            } else {
                console.error('Error fetching dropdown data:', response.statusText);
            }
        } catch (error) {
            console.error('Error fetching dropdown data:', error);
        }
    }

    async function fetchData() {
        try {
            const response = await fetch(fetchurl);
            if (response.ok) {
               
                data = await response.json();
                isLoading = false;
                filteredData = [...data];   
                console.log(filteredData)
            } else {
                console.error("Error fetching data:", response.statusText);
            }
        } catch (error) {
            console.error("Error fetching data:", error);
        }
    }

    onMount(() => {
        fetchDropdownData();
        fetchData();
    });

    function openUpdateModal(row) {
        UpdateData = {
    engg_name: row.engg_name || "",
    supplier: row.supplier || "",
    bill_no: row.bill_no || "",
    bill_date:formatToInputDate( row.bill_date )|| "",
    customer_name: row.customer_name || "",
    customer_po_no: row.customer_po_no || "",
    customer_po_date: formatToInputDate(row.customer_po_date) || "",
    item_description: row.item_description || "",
    billed_qty: row.billed_qty || "",
    unit: row.unit || "",
    net_value: row.net_value || "",
    cgst:row.cgst|| "", // No corresponding value provided in the original structure
    igst: row.igst||"", // No corresponding value provided in the original structure
    dispatch_through: row.dispatch_through||"" // No corresponding value provided in the original structure
    };
        selectedRow = row;
        showUpdateModal = true;
    }

    async function updateBilling(event) {
        event.preventDefault();
        isUpdating = true;

        try {
            const updatedFields = {};
            Object.keys(UpdateData).forEach((key) => {
                if (UpdateData[key] !== selectedRow[key]) {
                    updatedFields[key] = UpdateData[key];
                }
            });

            if (Object.keys(updatedFields).length === 0) {
                console.log("No fields updated.");
                isUpdating = false;
                return;
            }

            const updatedData = { ...selectedRow, ...updatedFields };

            const response = await fetch('https://srbilling.onrender.com/update', {
                method: "POST",
                body: JSON.stringify(updatedData),
            });

            if (response.ok) {
                console.log("Customer PO updated successfully.");
                showUpdateModal = false;
                selectedRow = null;
                await fetchData(); 
                console.log(data)
            } else {
                console.error("Failed to update customer PO:", response.statusText);
            }
        } catch (error) {
            console.error("Error updating customer PO:", error);
        } finally {
            isUpdating = false;
        }
    }

    async function downloadExcelBPO() {
        try {
            const response = await fetch('https://srbilling.onrender.com/download');
            if (response.ok) {
                const blob = await response.blob();
                const url = window.URL.createObjectURL(blob);
                const a = document.createElement("a");
                a.href = url;
                a.download = "billingpo.xlsx";
                a.click();
                window.URL.revokeObjectURL(url);
            } else {
                console.error("Error downloading Excel:", response.statusText);
            }
        } catch (error) {
            console.error("Error downloading Excel:", error);
        }
    }
    
    function confirmDelete(row) {
        rowToDelete = row;
        showDeleteModal = true;
    }

    async function deleteBill() {
        if (!rowToDelete) return;

        try {
            const response = await fetch(`https://srbilling.onrender.com/delete/${rowToDelete.id}`, {
                method: "POST",
            });

            if (response.ok) {
                console.log("Billing PO deleted successfully.");
                showDeleteModal = false;
                rowToDelete = null;
                await fetchData(); 
            } else {
                console.error("Failed to delete Billing PO:", response.statusText);
            }
        } catch (error) {
            console.error("Error deleting Billinng PO:", error);
        }
    }

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
        filteredData = [...data]; 
        return;
    }

    filteredData = data.filter(row => {
        const engineerName = row.enggNames?.toLowerCase() || ""; 
        const customerName = row.customerName?.toLowerCase() || ""; 
        return engineerName.includes(query) || customerName.includes(query);
    });
}
function formatToInputDate(timestamp) {
  if (!timestamp) return "";
  const date = new Date(timestamp);
  return date.toISOString().split("T")[0]; // Returns "YYYY-MM-DD"
}

  function formatDateToDayMonthYear(dateString) {
    if (!dateString) return "N/A";
    const date = new Date(dateString);
    return date.toLocaleDateString("en-GB", {
      day: "numeric",
      month: "long",
      year: "numeric"
    });
  }




</script>

<div class="min-h-screen flex flex-col bg-white">
    <Header />

    <main class="flex-grow py-24 px-2">
        <div class="relative w-full flex justify-end bg-white">
            <input
              type="text"
              bind:value={searchQuery}
              on:input={searchTable}
              placeholder="Search by Engineer or Customer Name"
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
                            <th class="py-3 px-4">Engineer Name</th>
                            <th class="py-3 px-4">Supplier</th>
                            <th class="py-3 px-4">Bill No</th>
                            <th class="py-3 px-4">Bill Date</th>
                            <th class="py-3 px-4">Customer Name</th>
                            <th class="py-3 px-4">Customer PO No</th>
                            <th class="py-3 px-4">Customer PO Date</th>
                            <th class="py-3 px-4">Item Description</th>
                            <th class="py-3 px-4">Billed Quantity</th>
                            <th class="py-3 px-4">Unit</th>
                            <th class="py-3 px-4">Net Value</th>
                            <th class="py-3 px-4">CGST</th>
                            <th class="py-3 px-4">IGST</th>
                            <th class="py-3 px-4">Dispatch Through</th>
                            <th class="py-3 px-4">Gross</th>
                            <th class="py-3 px-4">Total Tax</th>
                            <th class="py-3 px-4">Update</th>
                            <th class="py-3 px-4">Delete</th>

                            
                        </tr>
                    </thead>
                    <tbody>
                        {#if data.length === 0}
                            <tr>
                                <td colspan="30" class="py-4 text-center text-gray-600">No data available</td>
                            </tr>
                        {:else}
                            {#each filteredData as row, index}
                            <tr class="border-t border-gray-300 hover:bg-gray-200">
                            <td class="py-3 px-4 text-center">{index+1}</td>
                            <td class="py-3 px-4 text-center">{row.engg_name}</td>
                            <td class="py-3 px-4 text-center">{row.supplier}</td>
                            <td class="py-3 px-4 text-center">{row.bill_no}</td>
                            <td class="py-3 px-4 text-center">
                                {formatDateToDayMonthYear(row.bill_date)}
                              </td>
                            <td class="py-3 px-4 text-center">{row.customer_name}</td>
                            <td class="py-3 px-4 text-center">{row.customer_po_no}</td>
                            <td class="py-3 px-4 text-center">{formatDateToDayMonthYear(row.customer_po_date)}</td>
                            <td class="py-3 px-4 text-center">{row.item_description}</td>
                            <td class="py-3 px-4 text-center">{row.billed_qty}</td>
                            <td class="py-3 px-4 text-center">{row.unit}</td>
                            <td class="py-3 px-4 text-center">₹{row.net_value ? row.net_value.toFixed(2) : "0.00"}</td>
                            <td class="py-3 px-4 text-center">₹{row.cgst ? row.cgst.toFixed(2) : "0.00"}</td>
                            <td class="py-3 px-4 text-center">₹{row.igst ? row.igst.toFixed(2) : "0.00"}</td>
                            <td class="py-3 px-4 text-center">{row.dispatch_through ?? "N/A"}</td>
                            <td class="py-3 px-4 text-center">{row.gross ?? "0"}</td>
                            <td class="py-3 px-4 text-center">{row.total_tax ?? "0"}</td>
                            <td class="py-3 px-4 text-center">
                                <!-- svelte-ignore a11y_consider_explicit_label -->
                                 <button
                                    class="text-xl font-medium rounded-full flex items-center justify-center text-center"
                                    on:click={() => openUpdateModal(row)}>
                                        <i class="fas fa-edit"></i>
                                </button>
       
                            </td>
                            <td class="py-3 px-4 text-center">
    
                                <button class="bg-red-500 text-white px-3 py-1 rounded ml-2" on:click={() => confirmDelete(row)}>Delete</button>

                            </td>

                        
                             
                                </tr>
                            {/each}
                        {/if}
                    </tbody>
                </table>
            </div>
            <div class="py-4 flex justify-center items-center fixed bottom-0 left-0 right-0 text-center">
                <button class="text-white bg-black rounded-md px-9 py-2" 
                on:click={downloadExcelBPO}
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
  

{#if showUpdateModal}
<div class="fixed inset-0 bg-black opacity-50 z-40"></div> <!-- Background Overlay -->
<div class="fixed inset-0 flex items-center justify-center z-50">
    <div class="w-full max-w-4xl mx-4 bg-white rounded-lg p-4 shadow-lg relative">
        <form on:submit={updateBilling} class="bg-white shadow-xl rounded-lg p-8 space-y-8">

        <h1 class="text-center text-xl py-2 mb-6 font-semibold text-gray-900">Update</h1>

        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <!-- Engineer Name -->
                <div>
                    <label for="unit" class="block text-sm font-medium text-gray-700">Engineer Name</label>
                    <select id="unit" bind:value={UpdateData.engg_name}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required>
                        <option value="" disabled>Select a Unit</option>
                        {#each enggNames as engg_name}
                            <option value={engg_name}>{engg_name}</option>
                        {/each}
                    </select>
                </div>
        
            <!-- Supplier -->
            <div>
                <label for="unit" class="block text-sm font-medium text-gray-700">Supplier Name</label>
                <select id="unit" bind:value={UpdateData.supplier}
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required>
                    <option value="" disabled>Select a Unit</option>
                    {#each suppliers as supplier}
                        <option value={supplier}>{supplier}</option>
                    {/each}
                </select>
            </div>
          <!-- Bill Number -->
          <div>
            <label for="bill_No" class="block text-sm font-medium text-gray-700">Bill Number</label>
            <input type="text" id="bill_No" bind:value={UpdateData.bill_no} placeholder="Enter Bill No"
                class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
          <!-- Bill Date -->
          <div>
            <label for="bill_Date" class="block text-sm font-medium text-gray-700">Bill Date</label>
            <input type="date" id="bill_Date" bind:value={UpdateData.bill_date}
                class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
        </div>
    
    
            <!-- Customer Name -->
            <div>
                    <label for="unit" class="block text-sm font-medium text-gray-700">Customer Name</label>
                    <select id="unit" bind:value={UpdateData.customer_name}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required>
                        <option value="" disabled>Select a Unit</option>
                        {#each customers as customer}
                            <option value={customer}>{customer}</option>
                        {/each}
                    </select>
                </div>
                <div>
                    <label for="customer_Po_No" class="block text-sm font-medium text-gray-700">Customer Po No</label>
                    <input type="text" id="billed_Qty" bind:value={UpdateData.customer_po_no} placeholder="Enter Customer Po No"
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
                </div>
                <div>
                    <label for="customer_Po_Date" class="block text-sm font-medium text-gray-700">Customer Po Date</label>
                    <input type="date" id="bill_Date" bind:value={UpdateData.customer_po_date}
                        class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
                </div>
        
          
          
            <!-- Item Description -->
            <div>
                <label for="item_Description" class="block text-sm font-medium text-gray-700">Item Description</label>
                <input type="text" id="item_Description" bind:value={UpdateData.item_description} placeholder="Enter Item Description"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
            </div>
        
            <!-- Quantity -->
            <div>
                <label for="billed_Qty" class="block text-sm font-medium text-gray-700">Quantity</label>
                <input type="number" id="billed_Qty" bind:value={UpdateData.billed_qty} placeholder="Enter Quantity"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
            </div>
        
            <!-- Unit -->
            <div>
                <label for="unit" class="block text-sm font-medium text-gray-700">Unit</label>
                <select id="unit" bind:value={UpdateData.unit}
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required>
                    <option value="" disabled>Select a Unit</option>
                    {#each units as unit}
                        <option value={unit}>{unit}</option>
                    {/each}
                </select>
            </div>
        
            <!-- Net Value -->
            <div>
                <label for="net_Value" class="block text-sm font-medium text-gray-700">Net Value</label>
                <input type="number" id="net_Value" bind:value={UpdateData.net_value} placeholder="Enter Net Value"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" required />
            </div>
        
            <!-- CGST -->
            <div>
                <label for="cgst" class="block text-sm font-medium text-gray-700">CGST</label>
                <input type="number" id="cgst" bind:value={UpdateData.cgst} placeholder="Enter CGST"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" />
            </div>
        
            <!-- IGST -->
            <div>
                <label for="igst" class="block text-sm font-medium text-gray-700">IGST</label>
                <input type="number" id="igst" bind:value={UpdateData.igst} placeholder="Enter IGST"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" />
            </div>
        
            <!-- Dispatch Through -->
            <div>
                <label for="dispatch_through" class="block text-sm font-medium text-gray-700">Dispatch Through</label>
                <input type="text" id="dispatch_through" bind:value={UpdateData.dispatch_through} placeholder="Enter Dispatch Info"
                    class="mt-1 block w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-black focus:outline-none" />
            </div>
        
            <!-- Month of Delivery Scheduled -->
           
        </div>
        
        <!-- Submit Button -->
        <div class="flex justify-end mt-4">
            <button type="submit" class="px-6 py-2 bg-black text-white rounded-md transition duration-300 ease-in-out hover:bg-gray-800 disabled:opacity-50" disabled={isUpdating}>
                {#if isUpdating} Updating... {:else} Update Stock {/if}
            </button>
        </div>
        
        

        <!-- svelte-ignore a11y_consider_explicit_label -->
        <button
            class="absolute top-4 right-4 text-gray-500"
            on:click={() => showUpdateModal = false}
        >
            <i class="fas fa-times"></i>
        </button>
   </form>
    </div>
</div>
{/if}
<!-- Delete Confirmation Modal -->
{#if showDeleteModal}
<div class="fixed inset-0 flex items-center justify-center z-10">
    <!-- Background Overlay -->
    <div class="absolute inset-0 bg-black opacity-60"></div>

    <!-- Modal Box -->
    <div class="relative bg-white p-6 rounded-lg shadow-xl z-20">
        <h2 class="text-lg font-semibold text-black">Confirm Deletion</h2>
        <p class="text-black">Are you sure you want to delete this record?</p>
        <div class="mt-4 flex justify-end">
            <button class="bg-black text-white px-4 py-2 rounded mr-2" on:click={() => showDeleteModal = false}>Cancel</button>
            <button class="bg-red-500 text-white px-4 py-2 rounded" on:click={deleteBill}>Delete</button>
        </div>
    </div>
</div>
{/if}

</div>


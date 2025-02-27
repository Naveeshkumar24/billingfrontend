<script>
    import { goto } from "$app/navigation";
    import Header from "$lib/header.svelte";

    let isloading=false
</script>
<div class=",in-h-screen flex flex-col bg-white">
    <Header/>
    {#if isloading}
    <div class="flex justify-center items-center h-full">
        <div class="animate-spin h-12 w-12 rounded-full border-t-4 border-gray-800 "></div>
    </div>
{:else}
    <div class="relative h-screen bg-white text-black">
        <div class="container mx-auto px-6 py-24">
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 transition-all duration-300">
                <div class="p-4">
                    <div class="rounded-2xl p-6 bg-white flex flex-col h-72 duration-200 hover:shadow-xl border border-gray-200">
                        <span class="text-start mt-2 text-2xl font-semibold">Billing PO</span>
                        <div class="flex-grow"></div>
                        <button
                            class="text-xl p-3 rounded-lg bg-black text-white font-bold self-end hover:bg-gray-800 transition"
                            on:click={() => goto("/billlingpopage")}
                        >
                            Manage
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
{/if}
<footer class="bg-black text-white py-3 text-center text-sm">
    &copy; 2024 SRA BAO. All Rights Reserved.
</footer>
</div>
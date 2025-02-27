<script>
    import { goto } from "$app/navigation";

    let isDrawerOpen = false;

    function toggleDrawer() {
        isDrawerOpen = !isDrawerOpen;
    }
</script>

<header class="bg-black text-white fixed w-full z-50 shadow-md">

    <div class="container  flex items-center px-6 py-3">

        <!-- svelte-ignore a11y_consider_explicit_label -->
        <button
            class="text-3xl bg-black text-white shadow-md focus:outline-none"
            on:click={toggleDrawer}
        >
            <i class="fa-solid fa-bars"></i>
        </button>

      
        <div class="flex items-center justify-center flex-grow">
            <img src="/logo.jpeg" alt="SRA BAO Logo" class="w-12 h-12 rounded-full mr-4" />
            <h1 class="text-2xl font-light">SR Automation</h1>
        </div>


        <div class="flex items-center justify-end"></div>
    </div>


    {#if isDrawerOpen}
        <!-- svelte-ignore a11y_no_static_element_interactions -->
        <!-- svelte-ignore a11y_click_events_have_key_events -->
        <div class="fixed inset-0 z-40 bg-black opacity-30" on:click={toggleDrawer}></div>
        <div class="fixed top-0 left-0 h-full w-64 bg-white shadow-lg rounded-lg z-50 transform transition-transform duration-500 ease-in-out">
            <div class="p-6 flex flex-col h-full">
                <h2 class="text-xl font-bold mb-4">Menu</h2>
                <ul class="space-y-4">
                    <li>
                        <button
                            class="w-full text-left text-lg font-medium text-gray-700 hover:bg-gray-100 px-4 py-2 rounded-lg"
                            on:click={() => { goto('/mainpage'); toggleDrawer(); }}
                        >
                            <i class="fa-solid fa-house mr-3"></i> Home
                        </button>
                    </li>
                    <li>
                        <button
                            class="w-full text-left text-lg font-medium text-gray-700 hover:bg-gray-100 px-4 py-2 rounded-lg"
                            on:click={() => { goto('/fetchbillingpodata'); toggleDrawer(); }}
                        >
                            <i class="fa-solid fa-user-cog mr-3"></i> CPO
                        </button>
                    </li>
                  
                
                    <li>
                        <button
                            class="w-full text-left text-lg font-medium text-gray-700 hover:bg-gray-100 px-4 py-2 rounded-lg"
                            on:click={() => { goto('/'); toggleDrawer(); }}
                        >
                            <i class="fa-solid fa-sign-out-alt mr-3"></i> Logout
                        </button>
                    </li>
                </ul>

                <div class="mt-auto text-sm text-gray-500">
                    <p>&copy; 2024 SR Automation. All Rights Reserved.</p>
                </div>
            </div>
        </div>
    {/if}
</header>

<style>
    header {
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }
    button:focus {
        outline: none;
    }
</style>
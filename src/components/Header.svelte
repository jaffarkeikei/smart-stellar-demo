<script lang="ts">
    import { onMount } from "svelte";
    import { keyId } from "../store/keyId";
    import { contractId } from "../store/contractId";
    import { account, server } from "../utils/passkey-kit";
    import { truncate } from "../utils/base";

    let creating = false;
    let loading = false;
    let error: string | null = null;
    let menuOpen = false;

    onMount(async () => {
        if (localStorage.hasOwnProperty("ssd:keyId")) {
            keyId.set(localStorage.getItem("ssd:keyId")!);
        }
    });

    keyId.subscribe(async (value) => {
        if (value) {
            try {
                loading = true;
                const { contractId: cid } = await account.connectWallet({
                    keyId: value,
                });

                contractId.set(cid);
            } catch (e) {
                console.error("Error connecting wallet:", e);
                error = "Failed to connect wallet";
            } finally {
                loading = false;
            }
        }
    });

    async function login() {
        try {
            loading = true;
            error = null;
            const { keyIdBase64, contractId: cid } = await account.connectWallet();

            keyId.set(keyIdBase64);
            localStorage.setItem("ssd:keyId", keyIdBase64);

            contractId.set(cid);
        } catch (e) {
            console.error("Error logging in:", e);
            error = "Failed to login";
        } finally {
            loading = false;
        }
    }

    async function signUp() {
        try {
            creating = true;
            error = null;
            
            const {
                keyIdBase64,
                contractId: cid,
                signedTx,
            } = await account.createWallet("Smart Stellar Demo", "Smart Stellar Demo User");

            await server.send(signedTx);

            keyId.set(keyIdBase64);
            localStorage.setItem("ssd:keyId", keyIdBase64);

            contractId.set(cid);
        } catch (e) {
            console.error("Error creating account:", e);
            error = "Failed to create account";
        } finally {
            creating = false;
        }
    }

    function logout() {
        keyId.set(null);
        contractId.set(null);

        Object.keys(localStorage).forEach((key) => {
            if (key.includes("ssd:")) {
                localStorage.removeItem(key);
            }
        });

        Object.keys(sessionStorage).forEach((key) => {
            if (key.includes("ssd:")) {
                sessionStorage.removeItem(key);
            }
        });

        location.reload();
    }

    function toggleMenu() {
        menuOpen = !menuOpen;
    }
</script>

<header class="relative bg-lime-950 text-lime-100 shadow-md">
    <div class="container mx-auto px-4 py-3">
        <div class="flex items-center justify-between">
            <div class="flex items-center space-x-4">
                <a href="/" class="flex items-center space-x-2">
                    <svg class="w-8 h-8 text-lime-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
                        <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"></path>
                    </svg>
                    <span class="text-xl font-bold hidden sm:inline">Smart Stellar Demo</span>
                    <span class="text-xl font-bold sm:hidden">Stellar Demo</span>
                </a>
            </div>

            {#if error}
                <div class="absolute top-16 right-4 left-4 md:left-auto md:w-72 bg-red-100 border-l-4 border-red-500 text-red-700 p-3 rounded shadow-md z-50">
                    <div class="flex">
                        <div class="py-1">
                            <svg class="w-5 h-5 mr-2" fill="currentColor" viewBox="0 0 20 20">
                                <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zm-1 9a1 1 0 01-1-1v-4a1 1 0 112 0v4a1 1 0 01-1 1z" clip-rule="evenodd"></path>
                            </svg>
                        </div>
                        <div>
                            <p class="text-sm">{error}</p>
                        </div>
                        <button 
                            class="ml-auto" 
                            on:click={() => error = null}
                            aria-label="Close error message"
                        >
                            <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                                <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"></path>
                            </svg>
                        </button>
                    </div>
                </div>
            {/if}

            <div class="hidden md:flex items-center space-x-3">
                {#if $contractId}
                    <a
                        class="font-mono text-sm bg-lime-900 px-3 py-1 rounded hover:bg-lime-800 transition-colors"
                        href="https://stellar.expert/explorer/public/contract/{$contractId}"
                        target="_blank"
                        rel="noopener noreferrer"
                    >
                        <span class="flex items-center">
                            <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1"></path>
                            </svg>
                            {truncate($contractId, 4)}
                        </span>
                    </a>
                    <button
                        class="bg-red-600 hover:bg-red-700 text-white px-3 py-1 rounded transition-colors"
                        on:click={logout}
                    >
                        Logout
                    </button>
                {:else}
                    <button 
                        class="text-lime-300 hover:text-lime-100 transition-colors px-3 py-1"
                        on:click={login}
                        disabled={loading || creating}
                    >
                        {#if loading}
                            <span class="flex items-center">
                                <svg class="animate-spin w-4 h-4 mr-1" fill="none" viewBox="0 0 24 24">
                                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                                </svg>
                                Loading...
                            </span>
                        {:else}
                            Login
                        {/if}
                    </button>
                    <button
                        class="bg-lime-500 hover:bg-lime-600 text-lime-950 px-3 py-1 rounded transition-colors disabled:bg-gray-400 disabled:text-gray-700"
                        on:click={signUp}
                        disabled={creating || loading}
                    >
                        {#if creating}
                            <span class="flex items-center">
                                <svg class="animate-spin w-4 h-4 mr-1" fill="none" viewBox="0 0 24 24">
                                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                                </svg>
                                Creating...
                            </span>
                        {:else}
                            Create New Account
                        {/if}
                    </button>
                {/if}
            </div>

            <!-- Mobile menu button -->
            <button 
                class="md:hidden focus:outline-none"
                on:click={toggleMenu}
                aria-label="Menu"
            >
                <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 20 20">
                    {#if menuOpen}
                        <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"></path>
                    {:else}
                        <path fill-rule="evenodd" d="M3 5a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM3 10a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM3 15a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z" clip-rule="evenodd"></path>
                    {/if}
                </svg>
            </button>
        </div>

        <!-- Mobile menu -->
        {#if menuOpen}
            <div class="md:hidden mt-2 py-2 border-t border-lime-800">
                {#if $contractId}
                    <a
                        class="block py-2 px-4 font-mono text-sm hover:bg-lime-900 rounded"
                        href="https://stellar.expert/explorer/public/contract/{$contractId}"
                        target="_blank"
                        rel="noopener noreferrer"
                    >
                        <span class="flex items-center">
                            <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1"></path>
                            </svg>
                            View Contract ({truncate($contractId, 4)})
                        </span>
                    </a>
                    <button
                        class="block w-full text-left py-2 px-4 hover:bg-lime-900 rounded text-red-300"
                        on:click={logout}
                    >
                        <span class="flex items-center">
                            <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"></path>
                            </svg>
                            Logout
                        </span>
                    </button>
                {:else}
                    <button 
                        class="block w-full text-left py-2 px-4 hover:bg-lime-900 rounded"
                        on:click={login}
                        disabled={loading || creating}
                    >
                        {#if loading}
                            <span class="flex items-center">
                                <svg class="animate-spin w-4 h-4 mr-2" fill="none" viewBox="0 0 24 24">
                                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                                </svg>
                                Loading...
                            </span>
                        {:else}
                            <span class="flex items-center">
                                <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 16l-4-4m0 0l4-4m-4 4h14m-5 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h7a3 3 0 013 3v1"></path>
                                </svg>
                                Login
                            </span>
                        {/if}
                    </button>
                    <button
                        class="block w-full text-left py-2 px-4 mt-1 text-lime-950 bg-lime-500 hover:bg-lime-600 rounded disabled:bg-gray-400 disabled:text-gray-700"
                        on:click={signUp}
                        disabled={creating || loading}
                    >
                        {#if creating}
                            <span class="flex items-center">
                                <svg class="animate-spin w-4 h-4 mr-2" fill="none" viewBox="0 0 24 24">
                                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                                </svg>
                                Creating...
                            </span>
                        {:else}
                            <span class="flex items-center">
                                <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z"></path>
                                </svg>
                                Create New Account
                            </span>
                        {/if}
                    </button>
                {/if}
            </div>
        {/if}
    </div>
</header>
<script lang="ts">
    import { contractId } from "../store/contractId";
    import { truncate } from "../utils/base";
    import { onDestroy, onMount } from "svelte";
    import { keyId } from "../store/keyId";
    import { chat } from "../utils/chat";
    import { account, server } from "../utils/passkey-kit";
    import { getMessages } from "../utils/zettablocks";
    import { getEvents, rpc } from "../utils/rpc";

    let interval: NodeJS.Timeout;

    let msg: string = "";
    let msgs: ChatEvent[] = [];

    let sending: boolean = false;
    let loading: boolean = true;
    let error: string | null = null;

    onMount(async () => {
        try {
            loading = true;
            await callGetMessages();

            const { sequence } = await rpc.getLatestLedger();
            await callGetEvents(sequence - 17_280); // last 24 hrs

            interval = setInterval(async () => {
                try {
                    const { sequence } = await rpc.getLatestLedger();
                    await callGetEvents(sequence - 17_280); // last 24 hrs
                } catch (e) {
                    console.error('Error fetching new messages:', e);
                }
            }, 12_000); // 5 times per minute
        } catch (e) {
            console.error('Error initializing chat:', e);
            error = 'Failed to load messages. Please try again later.';
        } finally {
            loading = false;
        }
    });

    onDestroy(() => {
        if (interval) clearInterval(interval);
    });

    async function callGetMessages() {
        msgs = await getMessages();
        msgs = msgs.sort(
            (a, b) => a.timestamp.getTime() - b.timestamp.getTime(),
        );
    }

    async function callGetEvents(
        limit: number | string,
        found: boolean = false,
    ) {
        msgs = await getEvents(msgs, limit, found);
        msgs = msgs.sort(
            (a, b) => a.timestamp.getTime() - b.timestamp.getTime(),
        );
    }

    async function send() {
        if (!$contractId || !$keyId || !msg.trim()) return;

        try {
            sending = true;
            error = null;

            let at = await chat.send({
                addr: $contractId,
                msg,
            });

            at = await account.sign(at, { keyId: $keyId });

            await server.send(at);

            msg = "";
        } catch (e) {
            console.error('Error sending message:', e);
            error = 'Failed to send message. Please try again.';
        } finally {
            sending = false;
        }
    }

    function formatTimestamp(timestamp: Date): string {
        const now = new Date();
        const diff = now.getTime() - timestamp.getTime();
        
        // If less than 24 hours ago, show time
        if (diff < 24 * 60 * 60 * 1000) {
            return timestamp.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
        }
        
        // Otherwise show date and time
        return timestamp.toLocaleDateString([], { month: 'short', day: 'numeric' }) + 
               ' ' + timestamp.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }
</script>

<div class="flex flex-col min-w-full items-center my-10">
    {#if $contractId}
        <div class="w-full max-w-xl px-4">
            <div class="bg-white rounded-lg shadow-md overflow-hidden">
                <div class="bg-lime-950 text-white py-3 px-4 flex items-center">
                    <h2 class="text-lg font-semibold">Stellar Chat</h2>
                    <div class="ml-auto text-xs bg-lime-600 px-2 py-1 rounded-full">
                        {msgs.length} messages
                    </div>
                </div>

                {#if error}
                    <div class="bg-red-50 text-red-700 px-4 py-2 text-sm">
                        {error}
                    </div>
                {/if}

                <div class="p-4 h-[400px] overflow-y-auto bg-gray-50">
                    {#if loading}
                        <div class="flex justify-center items-center h-full">
                            <div class="animate-pulse flex flex-col items-center">
                                <div class="w-12 h-12 bg-lime-200 rounded-full mb-2"></div>
                                <div class="text-sm text-gray-500">Loading messages...</div>
                            </div>
                        </div>
                    {:else if msgs.length === 0}
                        <div class="flex justify-center items-center h-full text-gray-500">
                            <p>No messages yet. Start the conversation!</p>
                        </div>
                    {:else}
                        <ul class="space-y-3">
                            {#each msgs as event}
                                <li class="transition-all duration-200 hover:translate-x-1">
                                    <div class="flex flex-col">
                                        <div class="flex items-baseline mb-1">
                                            <a
                                                class="text-sm font-medium text-lime-700 hover:text-lime-900"
                                                target="_blank"
                                                href="https://stellar.expert/explorer/public/tx/{event.txHash}"
                                            >
                                                {truncate(event.addr, 4)}
                                            </a>
                                            <time
                                                class="ml-2 text-xs text-gray-500"
                                                datetime={event.timestamp.toUTCString()}
                                            >
                                                {formatTimestamp(event.timestamp)}
                                            </time>
                                        </div>
                                        <div 
                                            class="py-2 px-3 bg-white rounded-lg border border-gray-200 shadow-sm break-words"
                                        >
                                            {event.msg}
                                        </div>
                                    </div>
                                </li>
                            {/each}
                        </ul>
                    {/if}
                </div>

                <div class="border-t border-gray-200 p-4">
                    <form class="flex flex-col" on:submit|preventDefault={send}>
                        <textarea
                            class="border px-3 py-2 mb-3 border-gray-300 rounded-lg focus:ring-2 focus:ring-lime-500 focus:border-lime-500 focus:outline-none resize-none"
                            rows="3"
                            name="msg"
                            id="msg"
                            placeholder="Type your message..."
                            bind:value={msg}
                            disabled={sending || loading}
                        ></textarea>

                        <div class="flex items-center justify-between">
                            <div class="text-xs text-gray-500">
                                {#if sending}
                                    Sending message...
                                {:else}
                                    Messages are stored on the Stellar blockchain
                                {/if}
                            </div>
                            <button
                                class="bg-lime-700 hover:bg-lime-800 text-white px-4 py-2 rounded-lg font-medium text-sm transition-colors duration-200 disabled:bg-gray-400 disabled:cursor-not-allowed flex items-center"
                                type="submit"
                                disabled={sending || loading || !msg.trim()}
                            >
                                {#if sending}
                                    <svg class="animate-spin -ml-1 mr-2 h-4 w-4 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                                    </svg>
                                    Sending
                                {:else}
                                    Send Message
                                {/if}
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    {:else}
        <div class="bg-white rounded-lg shadow-md p-8 max-w-md text-center">
            <h1 class="text-2xl font-bold mb-4">Welcome to Stellar Chat</h1>
            <p class="text-gray-600 mb-6">Please login or create a new account to start chatting on the Stellar blockchain.</p>
            <div class="flex justify-center">
                <svg class="w-16 h-16 text-lime-600" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                    <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-6-3a2 2 0 11-4 0 2 2 0 014 0zm-2 4a5 5 0 00-4.546 2.916A5.986 5.986 0 0010 16a5.986 5.986 0 004.546-2.084A5 5 0 0010 11z" clip-rule="evenodd"></path>
                </svg>
            </div>
        </div>
    {/if}
</div>

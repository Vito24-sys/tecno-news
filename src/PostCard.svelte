<script>
    let {
        id,
        title,
        subtitle,
        imageUrl,
        createdAt,
        userId,
        currentUserId,
        onDelete
    } = $props();

    function formatTime(dateString){
        if (!dateString) return '';
        const date = new Date(dateString)
        const hora = date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
        const fecha = date.toLocaleDateString();
        return `${hora} hs (${fecha})`
    }
</script>

<article class="bg-slate-800 rounded-xl border border-slate-700 p-5 shadow-lg flex flex-col justify-between transition-all hover:border-cyan-500/50 realtive">
    <div class="space-y-3">
        <h3 class="text-xl font-bold text-slate-100 leading-snug hover:text-cyan-400 transition-colors">{title}</h3>

        <p class="text-slate-200/90 text-base leading-7">{subtitle}</p>

        {#if imageUrl}
            <div class="pt-2">
                <img
                 src={imageUrl}
                 alt={title}
                 class="w-full h-52 object-cover rounded-lg border border-slate-700"
                 loading="lazy"
                />
            </div>
        {/if}
    </div>

    <footer class="pt-4 border-t border-slate-700/60 flex items-center justify-between text-xs text-slate-400">
        <span class="inline-flex items-center gap-1.5">
            <svg class="w-3.5 h-3.5 text-cyan-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
            </svg>
            {formatTime(createdAt)}
        </span>

        {#if currentUserId && currentUserId === userId}
            <button
              onclick={() => onDelete(id, imageUrl)}
              class="inline-flex items-center gap-1 text-rose-400 hover:text-rose-300 bg-rose-500/10 hover:bg-rose-500/20 px-2.5 py-1 rounded transition-colors"
              title="Eliminar publicación"
            >
                <svg class="w-3.5 h-3.5 text-cyan-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1 1v3M4 7h16"/>
                </svg>
                Eliminar    
            </button>
        {/if}
    </footer>
</article>
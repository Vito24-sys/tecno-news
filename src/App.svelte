<script>
  import { onMount } from "svelte";
  import { supabase } from './supabaseClient';
  import PostCard from "./PostCard.svelte";

  let user = null;
  let authEmail = '';
  let authPassword = '';
  let isRegistering = false

  let posts = [];
  let title = '';
  let subtitle = '';
  let file = null;
  let loading = false;
  let uploading = false;

  onMount(async () => {
    const { data: { session } } = await supabase.auth.getSession();
    user = session?.user ?? null;

    supabase.auth.onAuthStateChange((_event, session) => {
      user = session?.user ?? null;
    });

    fetchPosts();
  });

  async function fetchPosts() {
    loading = true;
    const { data, error } = await supabase
      .from('posts')
      .select('*')
      .order('created_at', { ascending: false });

    if (!error) posts = data;
    loading = false;
  }

  async function handleAuth(e){
    e.preventDefault();
    try {
      if (isRegistering) {
        const { error } = await supabase.auth.signUp({
          email: authEmail,
          password: authPassword,
        });
        if (error) throw error;
        alert('Registro exitoso. Revisa tu correo o inicia sesión');
      } else {
        const { error } = await supabase.auth.signInWithPassword({
          email: authEmail,
          password: authPassword,
        });
        if (error) throw error;
      }
    } catch (err) {
      alert(err.message);
    }
  }

  async function handleLogout(){
    await supabase.auth.signOut();
  }

  function handleFileChange(event) {
    if (event.target.files && event.target.files.length > 0) {
      file = event.target.files[0];
    }
  }

  async function handleCreatePost(event){
    event.preventDefault();
    if (!user) {
      alert('Debes iniciar sesión para publicar.');
      return;
    }

    uploading = true;
    let imageUrl = null;

    try {
      if (file) {
        const fileExt = file.name.split('.').pop();
        const fileName = `${Date.now()}-${Math.random().toString(36).substring(2)}.${fileExt}`;

        const { error: uploadError } = await supabase.storage
          .from('post-images')
          .upload(fileName, file);

        if (uploadError) throw uploadError;

        const { data } = supabase.storage
          .from('post-images')
          .getPublicUrl(fileName);

        imageUrl = data.publicUrl;
      }

      const { error: insertError } = await supabase
        .from('posts')
        .insert([{
          title,
          subtitle,
          image_url: imageUrl,
          user_id: user.id
        }]);
      
      if (insertError) throw insertError;

      title = '';
      subtitle = '';
      file = null;
      event.target.reset();
      await fetchPosts();
    } catch (err) {
      alert(`Error al guardar: ${err.message}`);
    } finally {
      uploading = false;
    }
  }

  async function handleDeletePost(postId, imageUrl){
    const confirmDelete = confirm('¿Estás seguro de eliminar esta publicación?');
    if (!confirmDelete) return;

    try {
      const { error } = await supabase
        .from('posts')
        .delete()
        .eq('id', postId);

      if (error) throw error;

      if (imageUrl) {
        const fileName = imageUrl.split('/').pop();
        await supabase.storage.from('post-images').remove([fileName]);
      }

      posts = posts.filter((p) => p.id !== postId);
    } catch (err) {
      alert(`Error al eliminar: ${err.message}`);
    }
  }
</script>

<main class="min-h-screen bg-slate-900 text-slate-100 p-6 md:p-12">
  <div class="max-w-4xl mx-auto space-y-10">

    <header class="flex flex-col md:flex-row justify-between items-center pb-6 border-b border-slate-800 gap-4">
      <div>
        <h1 class="text-4xl font-extrabold text-cyan-400">Tecno News</h1>
        <p class="text-slate-400 text-sm mt-1">Novedades y actualidad en informática.</p>
      </div>

      <div>
        {#if user}
          <div class="flex items-center gap-3 bg-slate-800 px-4 py-2 rounded-lg border border-slate-700">
            <span class="text-xs text-slate-300 truncate max-w-[150px]">{user.email}</span>
            <button onclick={handleLogout} class="text-xs text-rose-400 hover:underline">
              Cerrar sesión
            </button>
          </div>
        {/if}
      </div>
    </header>

    {#if !user}
      <section class="bg-slate-800 p-6 rounded-xl border border-slate-700 max-w-md mx-auto">
        <h2 class="text-lg font-bold text-slate-200 mb-4 text-center">
          {isRegistering ? 'Crear una cuenta' : 'Inicia sesión para publicar'}
        </h2>
        <form onsubmit={handleAuth} class="space-y-3">
          <input
           type="email"
           bind:value={authEmail}
           placeholder="Correo electrónico"
           class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-3 text-sm text-white focus:outline-none focus:border-cyan-500"
           required
          >
          <input
           type="password"
           bind:value={authPassword}
           placeholder="Contraseña"
           class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-3 text-sm text-white focus:outline-none focus:border-cyan-500"
           required
          >
          <button
            type="submit"
            class="w-full bg-cyan-500 hover:bg-cyan-600 text-slate-950 font-semibold py-2 rounded-lg text-sm transition-colors"
          >
            {isRegistering ? 'Registrarse' : 'Ingresar'}
          </button>
        </form>
        <button 
          onclick={() => isRegistering = !isRegistering}
          class="w-full text-center text-xs text-slate-400 mt-3 hover:text-cyan-400"
        >
          {isRegistering ? '¿Ya tienes cuenta? Inicia sesión' : '¿No tienes cuenta? Regístrate'}
        </button>
      </section>
    {:else}
      <section class="bg-slate-800 p-6 rounded-xl border border-slate-700 shadow-xl">
        <h2 class="text-xl font-bold mb-4 text-slate-200">Publicar una noticia</h2>
        <form onsubmit={handleCreatePost} class="space-y-4">
          <input
           type="text"
           bind:value={title}
           placeholder="Título del artículo"
           class="w-full bg-slate-900 border border-slate-700 rounded-lg px-4 py-2 text-white focus:outline-none focus:border-cyan-500"
           required
          >
          <div>
            <label for="post-subtitle" class="block text-sm font-medium text-slate-300 mb-1">
              Subtítulo / Resumen
            </label>
            <textarea
              id="post-subtitle"
              bind:value={subtitle}
              rows="4"
              placeholder="Escribe aquí el resumen o desarrollo de la noticia..."
              class="w-full bg-slate-900 border border-slate-700 rounded-lg px-4 py-2.5 text-white placeholder-slate-500 focus:outline-none focus:border-cyan-500 resize-y transition-colors leading-relaxed"
              required
            ></textarea>
          </div>
          <input
           type="file"
           accept="image/*"
           onchange={handleFileChange}
           class="w-full text-sm text-slate-400 file:mr-4 file:py-2 file:px-4 file:rounded-md file:border-0 file:bg-cyan-600 file:text-white hover:file:bg-cyan-700 cursor-pointer"
          >
          <button
            type="submit"
            disabled={uploading}
            class="w-full bg-cyan-500 hover:bg-cyan-600 text-slate-950 font-bold py-2 rounded-lg transition-colors disabled:opacity-50"
          >
            {uploading ? 'Publicando...' : 'Publicar noticia'}
          </button>
        </form>
      </section>
    {/if}

    <section>
      <h2 class="text-2xl font-bold mb-6 text-slate-200">Últimas publicaciones</h2>

      {#if loading}
        <div class="h-6 w-6 border-4 border-slate-700 rounded-full border-t-4 border-t-blue-500 animate-spin m-auto"></div>
      {:else if posts.length === 0}
        <p class="text-slate-500">No hay publicaciones disponibles.</p>
      {:else}
        <div class="grid gap-6 md:grid-cols-2 items-start">
          {#each posts as post (post.id)}
            <PostCard
              id={post.id}
              title={post.title}
              subtitle={post.subtitle}
              imageUrl={post.image_url}
              createdAt={post.created_at}
              userId={post.user_id}
              currentUserId={user?.id}
              onDelete={handleDeletePost}
            />
          {/each}
        </div>
      {/if}
    </section>

  </div>
</main>

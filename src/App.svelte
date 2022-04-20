<script>
	import EditorPage from "./Page/EditorPage.svelte";


    if (import.meta.hot) {
        import.meta.hot.on(
            "vite:beforeUpdate",
            () => console.clear()
        );
    }

</script>

<main>
	<EditorPage></EditorPage>
</main>

<style>
	main {
        margin: 0;
        padding: 0;
	}
</style>
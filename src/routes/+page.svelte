<script lang="ts">
    import Tesseract from "tesseract.js";

    import "../app.css";

    let file: File | null = null;
    let imageUrl = "";
    let result = "";
    let fileInput: HTMLInputElement | null = null;

    const handleFile = (e: Event) => {
        const target = e.target as HTMLInputElement;
        if (target.files && target.files[0]) {
            file = target.files[0] as File;
            imageUrl = URL.createObjectURL(file);
        } else {
            alert("No file selected!");
        }
    };

    const handleExtract = () => {
        if (!file) {
            result = "No file selected for OCR.";
            return;
        }

        result = "Processing...";

        const reader = new FileReader();
        reader.onload = () => {
            if (reader.result) {
                Tesseract.recognize(reader.result as string, "eng", {
                    logger: (info) => console.log(info),
                })
                    .then(({ data: { text } }) => {
                        result = text;
                    })
                    .catch((error) => {
                        console.error("Error OCR: ", error);
                        result = "Error processing the image.";
                    });
            }
        };

        reader.readAsDataURL(file);
    };

    const handleRemove = () => {
        file = null;
        imageUrl = "";
        result = "";

        if (fileInput) {
            fileInput.value = "";
        }
    };
</script>

<svelte:head>
    <title>OCR - Svelte</title>
</svelte:head>
<main class="flex items-center justify-center min-h-screen bg-gray-100">
    <div class="text-center">
        <div class="my-5">
            <h1 class="font-bold text-3xl text-blue-600">OCR + Svelte</h1>
        </div>

        <div class="flex justify-center gap-2">
            <div class="mb-4">
                <input
                    type="file"
                    bind:this={fileInput}
                    onchange={handleFile}
                />
            </div>

            {#if file}
                <div class="mb-4">
                    <button
                        class="bg-blue-600 p-2 rounded-md text-white"
                        onclick={handleExtract}
                    >
                        Extract
                    </button>
                    <button
                        class="bg-red-600 p-2 rounded-md text-white"
                        onclick={handleRemove}
                    >
                        Remove
                    </button>
                </div>
            {/if}
        </div>

        {#if imageUrl}
            <div class="mb-10 flex justify-center">
                <img alt="" src={imageUrl} style="width: 500px;" />
            </div>
        {/if}

        {#if result}
            <div class="block mt-4 mb-8">
                <h4 class="font-bold text-2xl my-5">Result:</h4>
                <div class="max-w-[800px] mx-auto px-4">
                    <p class="text-justify break-words">{result}</p>
                </div>
            </div>
        {/if}
    </div>
</main>

<style lang="postcss">
    :global(html) {
        background-color: theme(colors.gray.300);
    }

    p {
        word-wrap: break-word; 
        overflow-wrap: break-word;
    }
</style>

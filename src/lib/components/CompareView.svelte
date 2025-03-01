<script lang="ts">
    import { marked } from "marked";
    import DOMPurify from "dompurify";

    let { workflow } = $props();

    let md_to_html = (md: string) =>
        DOMPurify.sanitize(marked.parse(md.replace(/^[\u200B\u200C\u200D\u200E\u200F\uFEFF]/, "")) as string);

    let workflow_yaml = $derived.by(() =>
        workflow ? md_to_html(workflow["answer"]) : "",
    );
    let generated_workflow_yaml = $derived.by(() =>
        workflow ? md_to_html(workflow["llm_response"]) : "",
    );
    let llm_judgement = $derived.by(() => workflow ? md_to_html(workflow["judgement"]) : "");
</script>

{#if workflow}
    <div class="px-12 py-8">
        {workflow["prompt"]}
    </div>
    <div class="grid grid-cols-2 gap-8 p-4">
        <div>
            <p>{@html workflow_yaml}</p>
        </div>
        <div>
            <p>{@html generated_workflow_yaml}</p>
        </div>
    </div>
    <div class="px-12 py-8">
        <p>{@html llm_judgement}</p>
    </div>
{/if}
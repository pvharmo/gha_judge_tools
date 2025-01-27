<script lang="ts">
    import { marked } from "marked";
    import DOMPurify from "dompurify";
    import { onDestroy } from "svelte";
    import { AppBar, FileUpload } from "@skeletonlabs/skeleton-svelte";
    import IconUpload from 'lucide-svelte/icons/upload';

    interface Workflow {
        id: string;
        level: string;
        judgement: string;
        answer: string;
        prompt: string;
        llm_response: string;
        judge_score: number;
        bleu_score: number;
    }

    let workflows: undefined | Workflow[] = $state(
        JSON.parse(localStorage.getItem("workflows") || "{}"),
    );

    let page = $state(0);
    let workflow: Workflow = $derived(
        workflows
            ? workflows[page]
            : {
                  id: "",
                  level: "",
                  judgement: "",
                  answer: "",
                  prompt: "",
                  llm_response: "",
              },
    );
    let workflow_yaml = $derived.by(() =>
        workflow
            ? DOMPurify.sanitize(
                  marked.parse(
                      workflow["answer"].replace(
                          /^[\u200B\u200C\u200D\u200E\u200F\uFEFF]/,
                          "",
                      ),
                  ) as string,
              )
            : "",
    );
    let generated_workflow_yaml = $derived.by(() =>
        workflow
            ? DOMPurify.sanitize(
                  marked.parse(
                      workflow["llm_response"].replace(
                          /^[\u200B\u200C\u200D\u200E\u200F\uFEFF]/,
                          "",
                      ),
                  ) as string,
              )
            : "",
    );

    let load_json = async (e: any) => {
        const file = e.acceptedFiles[0];
        const text = await file.text();
        const json = JSON.parse(text);

        json.sort(() => Math.random() - 0.5)

        localStorage.setItem("workflows", JSON.stringify(json.slice(0, 100)));

        workflows = json;
    };

    // Prompt 1
    let score = $state("");

    // Prompt 2
    let validity = $state("");
    let events = $state("");
    let jobs = $state("");
    let steps = $state("");
    let dependencies = $state("");
    // svelte-ignore state_referenced_locally
    let workflow_id = $state(workflow["id"]);

    const total: any = JSON.parse(localStorage.getItem("total") || "{}");

    function handleShortcut(event: KeyboardEvent) {
        const direction = event.key === "+" ? 1 : event.key === "-" ? -1 : 0;
        event.preventDefault();
        if (
            workflows &&
            page + direction < workflows.length &&
            page + direction >= 0 &&
            direction != 0
        ) {
            page += direction;
            score = total[workflow["id"]]?.score;
            validity = total[workflow["id"]]?.validity;
            events = total[workflow["id"]]?.events;
            jobs = total[workflow["id"]]?.jobs;
            steps = total[workflow["id"]]?.steps;
            dependencies = total[workflow["id"]]?.dependencies;
            workflow_id = workflow["id"]
        }
    }
    document.addEventListener("keyup", handleShortcut);

    onDestroy(() => {
        document.removeEventListener("keyup", handleShortcut);
    });

    let edit_id = $state(false);

    let find_id = () => {
        page = workflows?.findIndex((w) => w["id"] === workflow_id) || page;
        edit_id = false;
    };
</script>

<AppBar>
    {#snippet lead()}
    {#if edit_id}
        <input class="input" type="text" bind:value={workflow_id} />
        <button class="btn preset-filled" onclick={find_id}>
            Find
        </button>
    {:else}
        <button class="btn preset-tonal-surface" onclick={() => edit_id = true}>
            {workflow ? workflow["id"] : ""}
        </button>
    {/if}
    {/snippet}
    {workflow && workflow["level"]} -- BLEU score {workflow && workflow["bleu_score"].toFixed(2)} -- LLM Judge {workflow && workflow["judge_score"]}
    {#snippet trail()}
        <p>{page + 1} / {workflows?.length}</p>
        <FileUpload name="example-button" accept="application/json" onFileChange={load_json} maxFiles={1}>
          <button class="btn btn-icon">
            <IconUpload class="size-4" />
          </button>
        </FileUpload>
    {/snippet}
</AppBar>

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
{/if}

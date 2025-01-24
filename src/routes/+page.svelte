<script lang="ts">
    import { marked } from "marked";
    import DOMPurify from "dompurify";
    import { onDestroy } from "svelte";
    import { Segment, AppBar, FileUpload } from "@skeletonlabs/skeleton-svelte";
    import Prompt1 from "./prompt1.svelte";
    import Prompt2 from "./prompt2.svelte";
    import IconUpload from 'lucide-svelte/icons/upload';
    import IconDownload from 'lucide-svelte/icons/download';

    interface Workflow {
        id: string;
        level: string;
        judgement: string;
        answer: string;
        prompt: string;
        llm_response: string;
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

    const total: any = JSON.parse(localStorage.getItem("total") || "{}");

    const commit = () => {
        total[workflow["id"]] = {
            score: score,
            validity: validity,
            events: events,
            jobs: jobs,
            steps: steps,
            dependencies: dependencies,
        };
        localStorage.setItem("total", JSON.stringify(total));
    };

    function handleShortcut(event: KeyboardEvent) {
        const direction = event.key === "+" ? 1 : event.key === "-" ? -1 : 0;
        event.preventDefault();
        commit();
        if (
            workflows &&
            page + direction < workflows.length &&
            page + direction >= 0
        ) {
            page += direction;
            score = total[workflow["id"]]?.score;
            validity = total[workflow["id"]]?.validity;
            events = total[workflow["id"]]?.events;
            jobs = total[workflow["id"]]?.jobs;
            steps = total[workflow["id"]]?.steps;
            dependencies = total[workflow["id"]]?.dependencies;
        }
    }
    document.addEventListener("keyup", handleShortcut);

    onDestroy(() => {
        document.removeEventListener("keyup", handleShortcut);
    });

    const downloadScores = () => {
        const blob = new Blob([JSON.stringify(total)], {
            type: "application/json",
        });
        const url = window.URL.createObjectURL(blob);
        window.open(url);
        URL.revokeObjectURL(url);
    };
</script>

<AppBar>
    {#snippet lead()}
        {workflow["id"]}
    {/snippet}
    {workflow["level"]}
    {#snippet trail()}
        <p>{page + 1} / {workflows?.length}</p>
        <FileUpload name="example-button" accept="application/json" onFileChange={load_json} maxFiles={1}>
          <button class="btn btn-icon">
            <IconUpload class="size-4" />
          </button>
        </FileUpload>
        <button class="btn btn-icon " onclick={downloadScores}>
            <IconDownload class="size-4" />
        </button>
    {/snippet}
</AppBar>

{#if workflow}
    <form class="grid grid-cols-2 gap-8 p-4" oninput={commit}>
        <div>
            <p>Prompt:</p>
            <p>{workflow["prompt"]}</p>
        </div>
        <div>
            <p>Workflow:</p>
            <p>{@html workflow_yaml}</p>
        </div>
        <div class="p-4 card border-surface-200-800 border-[1px]">
            <Prompt1 />
        </div>
        <div class="flex items-center pt-8">
            <div class="mr-4">
                <p>Strongly Disagree</p>
            </div>
            <Segment name="align" bind:value={score}>
                <Segment.Item value="1">1</Segment.Item>
                <Segment.Item value="2">2</Segment.Item>
                <Segment.Item value="3">3</Segment.Item>
                <Segment.Item value="4">4</Segment.Item>
                <Segment.Item value="5">5</Segment.Item>
            </Segment>
            <div class="ml-4">
                <p>Strongly Agree</p>
            </div>
        </div>
        <div class="card border-surface-200-800 border-[1px]">
            <div class="m-4">
                <Prompt2 />
            </div>
        </div>
        <div>
            <div class="grid">
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Validity:</p>
                    </div>
                    <Segment name="align" bind:value={validity}>
                        <Segment.Item value="1">Valid</Segment.Item>
                        <Segment.Item value="2"
                            >Valid with minor errors</Segment.Item
                        >
                        <Segment.Item value="3"
                            >Mostly valid with major errors</Segment.Item
                        >
                        <Segment.Item value="4">Invalid</Segment.Item>
                    </Segment>
                </div>
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Requested events present:</p>
                    </div>
                    <Segment name="align" bind:value={events}>
                        <Segment.Item value="1">Yes</Segment.Item>
                        <Segment.Item value="0">No</Segment.Item>
                    </Segment>
                </div>
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Requested jobs present:</p>
                    </div>
                    <Segment name="align" bind:value={jobs}>
                        <Segment.Item value="1">Yes</Segment.Item>
                        <Segment.Item value="0">No</Segment.Item>
                    </Segment>
                </div>
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Requested steps present:</p>
                    </div>
                    <Segment name="align" bind:value={steps}>
                        <Segment.Item value="1">Yes</Segment.Item>
                        <Segment.Item value="0">No</Segment.Item>
                        <Segment.Item value="-">Not requested</Segment.Item>
                    </Segment>
                </div>
                <div class="flex flex-row items-center my-2">
                    <div class="mr-4">
                        <p>Requested dependencies present:</p>
                    </div>
                    <Segment name="align" bind:value={dependencies}>
                        <Segment.Item value="1">Yes</Segment.Item>
                        <Segment.Item value="0">No</Segment.Item>
                        <Segment.Item value="-">Not requested</Segment.Item>
                    </Segment>
                </div>
            </div>
        </div>
    </form>
{/if}

<p>
    You are an expert devops engineer. Judge wether the workflow is valid and
    follows the instructions in the prompt. Award points based on the following
    criteria:
</p>
<ul>
    <li>
        Three points if the workflow is valid. Two points if the workflow has
        some minor errors. One point if the workflow has some major errors but
        is mostly valid. Zero point if the workflow has many errors or is
        invalid.
    </li>
    <li>
        One point if every events requested in the prompt are present in the
        workflow.
    </li>
    <li>
        One point if every jobs requested in the prompt are present in the
        workflow.
    </li>
    <li>
        One point if all the steps and dependencies requested in the prompt are
        present in the workflow. If it was not requested in the prompt, award a
        point if the steps are valid.
    </li>
</ul>

<style>
    ul {
        list-style-type: initial;
    }
</style>

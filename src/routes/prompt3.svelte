<p>
    You are an expert devops engineer. Judge wether the workflow is valid and
    follows the instructions in the prompt. Answer the following questions:
</p>
<ol>
    <li>
        Is the workflow syntactically valid ? Answer with "Yes, the workflow is
        valid", "Yes but has a few minor errors", "No, but mostly valid", or
        "No, the workflow is invalid".
    </li>
    <li>
        Are every events requested in the prompt are present in the workflow ?
        Answer with "Yes, all events are present" or "No, not all events are
        present".
    </li>
    <li>
        Are every jobs requested in the prompt are present in the workflow ?
        Answer with "Yes, all jobs are present" or "No, not all jobs are
        present".
    </li>
    <li>
        Are all the steps requested in the prompt are present in the workflow ?
        Answer with "Yes, all steps are present" or "No, not all steps are
        present". If it was not requested in the prompt, answer "No steps were
        requested".
    </li>
    <li>
        Are all the dependencies requested in the prompt are present in the
        workflow? Answer with "Yes, all dependencies are present" or "No, not
        all dependencies are present". If it was not requested in the prompt,
        answer "No dependencies were requested".
    </li>
</ol>

<style>
    ol {
        list-style-type: upper-alpha;
    }
</style>

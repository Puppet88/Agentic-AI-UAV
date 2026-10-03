# Agentic-AI-UAV

Agentic AI for UAV-enabled smart manufacturing systems.

This repository provides the released notebook, prompts, and RAG knowledge-base files used to validate the proposed Agentic AI framework.

## Environment

The released notebook was executed with:

- Python 3.10.18
- Jupyter Notebook / JupyterLab

Install the required Python packages from the provided dependency manifest:

```bash
pip install -r requirements.txt
```

The `requirements.txt` file pins package versions when they were explicitly recorded by the released notebook. Packages whose exact versions were not recorded are listed without version constraints.

## Repository Structure

The main files required to run the released pipeline are:

```text
Agentic AI Framework.ipynb
requirements.txt
User_Prompt.txt
System_Prompt1.txt
System_Prompt2.txt
System_Prompt3.txt
database/
LICENSE
```

The `database/` directory contains the Markdown files used to construct the local RAG knowledge base.

## OpenAI API Key

The notebook expects a local file named:

```text
OPENAI_API_KEY.txt
```

Create this file in the repository root directory and place your OpenAI API key on a single line.

Example:

```text
YOUR_OPENAI_API_KEY
```

Do not commit, upload, or publicly share `OPENAI_API_KEY.txt`.

## Required Prompt Files

The following prompt files must remain in the repository root directory because the notebook loads them using relative paths:

```text
User_Prompt.txt
System_Prompt1.txt
System_Prompt2.txt
System_Prompt3.txt
```

The notebook also expects the RAG source files to be available in:

```text
./database
```

## Reproduction Steps

1. Clone or download this repository.

2. Open a terminal in the repository root directory.

3. Create and activate a Python 3.10.18 environment.

4. Install the required dependencies:

   ```bash
   pip install -r requirements.txt
   ```

5. Create `OPENAI_API_KEY.txt` in the repository root directory and place your OpenAI API key in this file.

6. Confirm that the following files and directory are present:

   ```text
   User_Prompt.txt
   System_Prompt1.txt
   System_Prompt2.txt
   System_Prompt3.txt
   database/
   ```

7. Launch JupyterLab:

   ```bash
   jupyter lab
   ```

8. Open:

   ```text
   Agentic AI Framework.ipynb
   ```

9. Run the notebook cells in order from the repository root directory.

## Pipeline Configuration

The released notebook uses:

- Planning model: `gpt-5.6-luna`
- Responder model: `gpt-5.6-luna`
- Verifier model: `gpt-5.6-luna`
- OpenAI embedding model: `text-embedding-ada-002`
- RAG source directory: `./database`
- RAG retrieval setting: similarity search with `k = 4`

The notebook builds the RAG vector database when a valid local index is not available. It otherwise loads the existing local index.

## Outputs

The notebook creates an `outputs/` directory for generated results and logs. Depending on the executed pipeline, the outputs include:

```text
outputs/
├── cot_step_results.json
├── cot_final_draft.md
├── verified_formulation_context.json
├── log.txt
├── debug_logs/
└── cost_latency_token/
```

Additional step-level responder and verifier outputs are also written to the `outputs/` directory during execution.

## Notes on Reproducibility

The pipeline makes calls to the OpenAI API. Therefore, reproduction requires a valid OpenAI API key and access to the model names configured in the released notebook.

Because the framework uses large language models, individual generated responses can vary across runs even when the same prompts and configuration are used. The released notebook records intermediate outputs and logs to support inspection of the pipeline execution.

## License

The software in this repository is released under the MIT License. See the `LICENSE` file for details.

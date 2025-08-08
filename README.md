# Consolidated Checkpoint Conversion Workflow

This script automates the conversion of Megatron-LM checkpoints to HuggingFace format using SLURM batch jobs and a containerized environment.

## Usage

Run the script with the required arguments:

```bash
python consolidated_conversion_workflow.py \
  --checkpoint_dir <path_to_checkpoint_dir> \
  --save_checkpoints_dir <path_to_save_converted_checkpoints> \
  --opensci_hf_path <path_to_Open-Sci-hf_repo> \
  --convert_logs_dir <path_to_conversion_logs> \
  --container_image <path_to_container_image.sif> \
  --venv_path <path_to_python_venv_activate> \
  --log_path <path_to_slurm_log_file> \
  --opensci_megatron_path <path_to_Megatron-LM-Open-Sci_repo>
```

### Example

```bash
python consolidated_conversion_workflow.py \
  --checkpoint_dir /leonardo_work/AIFAC_L01_028/hmahadik/checkpoint_backup/nemotron-cc-hq/open-sci-ref_model-0.13b_data-Nemotron-cc-2024-HQ-real-synth-mix_tokenizer-GPT-NeoX_samples-300B_global_bs-1008_context-4096_schedule-WSD_lr-4e-3_warmup-25000_machine-LEONARDO_14467405/checkpoints/torch/ \
  --save_checkpoints_dir /leonardo_work/AIFAC_L01_028/hmahadik/checkpoint_backup/nemotron-cc-hq/open-sci-ref_model-0.13b_data-Nemotron-cc-2024-HQ-real-synth-mix_tokenizer-GPT-NeoX_samples-300B_global_bs-1008_context-4096_schedule-WSD_lr-4e-3_warmup-25000_machine-LEONARDO_14467405/checkpoints/ \
  --opensci_hf_path /leonardo_work/AIFAC_L01_028/hmahadik/Open-Sci-hf \
  --convert_logs_dir /leonardo/home/userexternal/hmahadik/logs/convert_logs \
  --container_image /leonardo_work/AIFAC_L01_028/hmahadik/container_pretraining_megatron.sif \
  --venv_path /leonardo/home/userexternal/hmahadik/myenv/bin/activate \
  --log_path /leonardo/home/userexternal/hmahadik/logs/Smollm2_annealing/actual_run.out \
  --opensci_megatron_path /leonardo_work/AIFAC_L01_028/hmahadik/Megatron-LM-Open-Sci
```

## Arguments
- `--checkpoint_dir`: Directory containing Megatron-LM checkpoints (with `iter_XXXXXXX` subfolders).
- `--save_checkpoints_dir`: Directory to save converted HuggingFace checkpoints.
- `--opensci_hf_path`: Path to the Open-Sci-hf repository (for config/model/tokenizer files).
- `--convert_logs_dir`: Directory to store SLURM job logs for conversion.
- `--container_image`: Path to the container image (Singularity `.sif`).
- `--venv_path`: Path to the Python virtual environment activation script.
- `--log_path`: Path to the SLURM log file containing model training arguments.
- `--opensci_megatron_path`: Path to the Megatron-LM-Open-Sci repository.
- `--account`, `--partition`: (Optional) SLURM account and partition for job submission.
- `--tokenizer_cache_dir`: (Optional) Directory for HuggingFace tokenizer cache.

## Workflow Overview
1. **Argument Parsing**: The script parses all command-line arguments and validates paths.
2. **Checkpoint Discovery**: Scans the checkpoint directory for iteration subfolders (e.g., `iter_0002000`).
3. **Model Config Extraction**: Reads the SLURM log file to extract model hyperparameters and configuration.
4. **Conversion Filtering**: Checks which iterations have already been converted (by presence of `model.safetensors` in the output directory).
5. **SBATCH Script Generation**: For each unconverted iteration, generates a SLURM batch script using the provided template and arguments.
6. **Job Submission**: Submits the batch script to SLURM using `sbatch`, running the conversion inside the specified container and virtual environment.
7. **Output**: Converted HuggingFace checkpoints are saved in the specified output directory, with logs in the conversion logs directory.

## Notes
- The SBATCH template and conversion logic needs adjustment for different cluster configurations.

## Troubleshooting
- Check SLURM job logs in the `convert_logs_dir` for errors.
- Ensure the container image includes all required dependencies.
- If conversion fails for an iteration, check the SLURM log and model config extraction steps.

# LSTM Training Rules

## Execution
- Run training scripts with `nohup` + log file to avoid timeout
- Use log files to monitor training progress (do not poll interactively)

## Checkpoints
- Keep at most 5 checkpoints per training session to save disk space
- This prevents training from stopping due to low disk space

## Stopping
- Before stopping any training, check its status and report to user
- Let user decide whether to stop
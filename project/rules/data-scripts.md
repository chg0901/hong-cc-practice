# Data Processing & Script Rules

## File Output
- All generated files (png/pkl/csv) must include timestamp (YYYYMMDDHHmmss) in filename
- Group multiple output files for one task in a new subfolder

## Visualization
- Loss curves: use EMA smoothing with confidence interval (mean +/- std) bands
- One plot per metric; do not use subplot for training curves
- Do not use `ax.text()` in subplots; print summaries to terminal instead
- Chinese font: `plt.rcParams['font.sans-serif'] = ['SimHei', 'WenQuanYi Zen Hei']`
- Fix minus sign: `plt.rcParams['axes.unicode_minus'] = False`
- Backend: `matplotlib.use('Agg')` for server, `matplotlib.use('TkAgg')` for GUI

## Database
- SQLite database: `smart_energy.db`
- Use `check_db.py`, `check_db_structure.py`, `query_db.py` for database operations

## LSTM Model Configuration

**lstmpy6-3_m.py (Keras/PySide6)**:
- `filename`, `lablename`, `timesteps` (default 20), `model_name`, `creatModel` (True=train new)
- `single` (True=single target), `train_ratio` (0.75), `epochs` (40), `batch_size` (32)
- `hidden_units` (128), `predict_steps` (20)

**lightlstm1_m.py (PyTorch)**:
- `TIME_STEP` (6), `HIDDEN_SIZE` (16), `NUM_LAYERS` (3), `BATCH_SIZE` (8)
- `EPOCHS` (10), `LSTM_TRAIN_INTERVAL` (30s)

## LSTM Model Architecture (Keras)
- Input -> LSTM(hidden_units, return_sequences=True) -> LayerNormalization -> Attention -> Dropout -> LSTM(128) -> LayerNormalization -> Dense(output)
- L2 regularization, Adam (lr=0.0005), MSE loss
- EarlyStopping, ModelCheckpoint, ReduceLROnPlateau callbacks

## Time Series Data Preparation
- CSV loading: UTF-8 first, fallback gb2312
- Duplicate rows/columns removed; missing: random sampling (numeric), mode (categorical)
- Outliers: IQR method (Q1 - 1.5*IQR, Q3 + 1.5*IQR)
- MinMaxScaler normalization; sequence: X[i:i+timesteps], y[i+timesteps]
- Data length adjusted to multiple of timesteps

## Data Gap Filling

`scripts/fill_sensor_gap.py` detects and fills gaps in `sensor_data` + `energy_consumption`:

```bash
python scripts/fill_sensor_gap.py --dry-run  # Estimate gap size
python scripts/fill_sensor_gap.py            # Fill the gap
```

- Generates values using the same patterns as `generate_test_data()` (COP daily sine, power_meter random, etc.)
- Fills from last record timestamp to now, hourly for sensor_data, daily for energy_consumption
- Supports `--dry-run` for safe estimation without writing

## EDA Data Sources
Located in `lstm_eda/`:
- `dt11.csv` - Vibration data (16384 rows, single column)
- `golu22.csv` - Boiler data (369 rows, 10 columns)
- `scycgl1.csv` - Tobacco boiler (5659 rows, 23 columns)
- `ftt13-1.csv` - Frequency/phase data (6400 rows, 2 columns)
- `SP500-2.csv` - Stock data (16862 rows, OHLCV)

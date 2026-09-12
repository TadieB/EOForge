
---

## Requirements

- Python ≥ 3.10
- `parsl`, `zarr`, `xarray`, `rioxarray`, `numpy`, `pandas`, `webdataset`, `matplotlib`
- `gdal` (VRT-based reprojection)
- Optional: `prov[dot]` + Graphviz, for provenance DAG rendering
- A NASA Earthdata token (LAADS DAAC access)
- Access to an HPC cluster with SLURM and a Lustre (or equivalent) shared filesystem

```bash
conda env create -f environment.yml
conda activate eoforge
```

---

## Configuration

All parameters live in `pipeline_config.yaml`, controlled by a single `stage` key:

```yaml
stage: debug   # debug | benchmark | production_test | production
```

Each stage preset sets its own spatial/temporal scope and worker counts. Environment variables (`MAIN_WORKERS`, `EU_WORKERS`, `DTN_THREADS`, `BENCHMARK_BYTES`, `PROCESS_NUM_DAYS`) override any value per-submission without editing the YAML — used by the benchmark sweep scripts to vary configuration across runs.

---

## Usage

**Run the full pipeline** (download auto-chains process on completion):

```bash
sbatch download_stage.sbatch
```

**Run a specific processing subset** of already-downloaded days:

```bash
PROCESS_NUM_DAYS=5 sbatch process_stage.sbatch
```

**Reproduce the weak-scaling benchmark:**

```bash
./benchmark_weak_scaling_sweep.sh      # download sweep, 1–128 GB
./benchmark_weak_scaling_process.sh    # process sweep, 1–128 workers
python postprocessing/extract_benchmark_provenance.py > benchmark_final.csv
python postprocessing/compute_all_metrics.py
python postprocessing/plot_all_metrics.py
```

**Generate a provenance graph:**

```bash
python postprocessing/provenance_graph.py --input <provenance.json> --format svg
```

---

## Benchmark Results

Weak-scaling characterization on OLCF Andes (32-core AMD EPYC nodes), jointly scaling dataset size, compute workers, and I/O concurrency, 3 repetitions per configuration:

| Cores | Data (GB) | Throughput (MB/s) | Efficiency |
|---|---|---|---|
| 1   | 1   | 0.35  | 100% |
| 8   | 8   | 5.86  | 211% |
| 32  | 32  | 23.54 | 212% |
| 128 | 128 | 74.49 | 168% |

Efficiency exceeds ideal (100%) up to 32 workers due to fixed-overhead amortization, declining at 128 workers as non-parallelized downstream stages become proportionally dominant. Per-tile ARD profiling shows a stable, compute-dominant time-share (~54% compute, 35% I/O read, 11% write) invariant to scale. Full figures in `postprocessing/` output.

---

## Citation

If you use EOForge in your work, please cite:

```bibtex
@inproceedings{eoforge2026,
  title     = {EOForge: A Scalable, Efficient Dataset-as-Code Pipeline
               for Earth Observation and Climate Applications},
  author    = {[Author names]},
  booktitle = {[Venue]},
  year      = {2026}
}
```

---

## Acknowledgments

Developed at the Department of Information Engineering and Computer Science, University of Trento, using compute resources at the Oak Ridge Leadership Computing Facility (OLCF). Add advisor/collaborator credits as appropriate.

---

## License

[MIT](LICENSE) — placeholder; update to match your institution's or funding requirements.

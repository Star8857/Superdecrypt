# Summarize current artifacts: telemetry, runtime benchmarks, RL Q-table, package zips, benchmark CSVs.

import os, json, pandas as pd, pprint

from caas_jupyter_tools import display_dataframe_to_user



root = "/mnt/data"

artifacts = {}



# collect files of interest

candidates = [

    "superdecrypt_full_release.zip",

    "superdecrypt_ci_package.zip",

    "benchmark_results.csv",

    "superdecrypt_runtime/telemetry.json",

    "superdecrypt_runtime/runtime_benchmark_results.csv",

    "superdecrypt_runtime/rl_q_table.json"

]



for c in candidates:

    path = os.path.join(root, c)

    artifacts[c] = os.path.exists(path)



summary = {"artifact_exists": artifacts}



# Read telemetry if present

telemetry_path = os.path.join(root, "superdecrypt_runtime", "telemetry.json")

telemetry = []

if os.path.exists(telemetry_path):

    try:

        with open(telemetry_path, "r") as f:

            telemetry = json.load(f)

    except Exception as e:

        telemetry = [{"error": str(e)}]



# Read runtime benchmark CSV

runtime_csv = os.path.join(root, "superdecrypt_runtime", "runtime_benchmark_results.csv")

runtime_df = None

if os.path.exists(runtime_csv):

    runtime_df = pd.read_csv(runtime_csv)



# Read benchmark_results.csv

bench_csv = os.path.join(root, "benchmark_results.csv")

bench_df = None

if os.path.exists(bench_csv):

    try:

        bench_df = pd.read_csv(bench_csv)

    except Exception:

        bench_df = None



# Read RL q-table snapshot

qtable_path = os.path.join(root, "superdecrypt_runtime", "rl_q_table.json")

qtable = {}

if os.path.exists(qtable_path):

    try:

        with open(qtable_path, "r") as f:

            qtable = json.load(f)

    except Exception as e:

        qtable = {"error": str(e)}



# Prepare human-friendly summaries

telemetry_count = len(telemetry) if isinstance(telemetry, list) else 0

telemetry_sample = telemetry[-10:] if telemetry_count>0 else []



qtable_keys = list(qtable.keys())[:10] if isinstance(qtable, dict) else []



summary.update({

    "telemetry_count": telemetry_count,

    "telemetry_sample_last_10": telemetry_sample,

    "runtime_rows": len(runtime_df) if runtime_df is not None else 0,

    "benchmark_rows": len(bench_df) if bench_df is not None else 0,

    "qtable_keys_sample": qtable_keys

})



print("Artifacts presence:")

pprint.pprint(artifacts)

print("\nSummary:")

pprint.pprint(summary)



# Display dataframes to user if present

if runtime_df is not None and len(runtime_df)>0:

    display_dataframe_to_user("Runtime Benchmark Results", runtime_df)

if bench_df is not None and len(bench_df)>0:

    display_dataframe_to_user("Benchmark Results", bench_df)



# Save short report

report = {

    "artifacts": artifacts,

    "telemetry_count": telemetry_count,

    "runtime_rows": len(runtime_df) if runtime_df is not None else 0,

    "benchmark_rows": len(bench_df) if bench_df is not None else 0,

    "qtable_keys_sample": qtable_keys

}

report_path = os.path.join(root, "superdecrypt_status_report.json")

with open(report_path, "w") as f:

    json.dump(report, f, indent=2)



report_path


# NVOC-CLI-Stressor

> Language switch / 语言切换: [中文](#zh-cn) | [English](#en)
>
> License / 许可证: [Apache 2.0](LICENSE)
>
> 本仓库根目录的 `LICENSE` 适用于所有分支（包括 CUDA 和 opencl 分支）。

---

## 中文 <a id="zh-cn"></a>

这是一个基于 OpenCL 的 GPU 核心域稳定性压力测试工具。它通过时间驱动、随机化的通用矩阵乘法（GEMM）工作负载，结合旁路校验机制来对显卡进行压力测试，能够有效检测显卡的静默计算错误（Silent Data Corruption）或硬件稳定性问题。

> **提示**：当前仓库分支为 `opencl`，使用 `pyopencl` 驱动随机 GEMM 压力测试；如需 `CUDA` 版，请切换到对应分支。

### 功能特点

- **多精度支持**：支持测试 FP16、FP32、FP64；其中 FP16 需要设备支持 `cl_khr_fp16`，FP64 需要设备支持 `cl_khr_fp64` 或 `cl_amd_fp64`。
- **随机化工作负载**：动态改变矩阵尺寸（包含非对齐的尺寸），制造冷热交替的计算阶段，对显卡供电和内存分配器施加压力。
- **OpenCL 平台/设备选择**：可按平台与设备索引指定目标 GPU，也可以自动选择首个可用设备。
- **旁路数据校验**：周期性中断压力测试，并使用 CPU 上 FP64 参考算法进行确定性计算校验，捕获静默错误。
- **持续高压执行**：可自定义执行时长，对 GPU 持续平缓或剧烈施压。

### 环境要求

- 最低 Python 版本：`>=3.11`
- 可用的 OpenCL 平台/驱动/ICD，且目标设备具备足够的内存与算力。

### 安装说明

项目推荐使用 `uv` 进行虚拟环境和依赖的管理。

1. 克隆本仓库：

```bash
git clone https://github.com/your-username/NVOC-CLI-Stressor.git
cd NVOC-CLI-Stressor
```

2. 安装依赖并自动建立独立环境：

```bash
uv sync
```

如果你只想使用纯 `venv` + `pip`，可以直接安装仓库根目录提供的依赖文件：

```bash
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

如果需要生成 OpenCL 可执行文件，直接使用同一个 `requirements.txt` 即可：

```bash
pip install -r requirements.txt
```

然后使用：

```bash
pyinstaller NVOC-CLI-Stressor-opencl.spec
```

默认生成物会输出到 `dist/NVOC-CLI-Stressor-opencl/`。当前 spec 以仓库入口脚本为模板；如果你切换到独立的 OpenCL 启动文件，请在 spec 中修改 `entry_script`。

### 使用方法

借助 `uv` 可以直接运行测试环境并拉起测试：

```bash
uv run test.py [参数]
```

如果是在当前 Python 环境中：

```bash
python test.py [参数]
```

#### 可用参数

- `--platform-index`：OpenCL 平台索引；不指定时自动选择首个可用 GPU 平台。
- `--device-index`：平台内 GPU 索引；需与 `--platform-index` 配合使用，省略时默认 0。
- `--list-devices`：列出所有可见 OpenCL 平台和设备并退出。
- `--duration`（默认：90.0）：每个精度模式的压力持续时间（秒）。
- `--matrix-sizes`（默认：`2049, 4096, 4097, 8192, 8193, 16384`）：用于常规压力测试的随机矩阵尺寸列表，以逗号分隔。
- `--fp64-matrix-sizes`（默认：`2048, 4096`）：专门用于 FP64 模式的矩阵尺寸，避免双精度压力阶段过慢。
- `--precisions`（默认：`fp16,fp32`）：需要测试的精度列表。可选：`fp16`、`fp32`、`fp64`（按设备能力自动跳过）。
- `--warmup-iters`（默认：3）：每个工作负载窗口的预热轮数。
- `--burst-iters`（默认：6）：每个工作负载窗口的正式压力突发轮数。
- `--validate-interval`（默认：10）：旁路校验的间隔秒数。
- `--validate-size`（默认：768）：旁路校验所用的固定矩阵尺寸。
- `--transpose-prob`（默认：0.5）：随机转置 a/b 矩阵的概率，用于轻度扰动 kernel 执行路径。
- `--jitter-rate`（默认：0.3）：选择抖动尺寸的概率，其余情况使用首选矩阵尺寸。
- `--min-burst-ms`（默认：40.0）：每个工作负载窗口最少执行时长（毫秒），不足时自动追加 kernel 轮数。
- `--input-refresh-interval`（默认：1）：连续复用同一组输入的窗口数；`1` 表示每个窗口都刷新输入。
- `--seed`（默认：12345）：随机种子，保证结果一致性。

更多高级参数请以 `python test.py --help` 的输出为准。

### 输出日志概览

- 检测并输出基本的设备架构信息（架构版本、Compute Capability 等）。
- 实时持续显示迭代矩阵的尺寸、瞬间算力（TFLOPS）及验证进度。
- 在每个精度测试结束后，核心稳定性总结面板将给出是否出现报错的情况。

### 许可证

本项目采用 Apache License 2.0，详见根目录的 `LICENSE` 文件。

---

## English <a id="en"></a>

This is an OpenCL-based GPU core-stability stress tool. It drives randomized generalized matrix multiplication (GEMM) workloads over time, combined with a sidecar validation path, to stress the GPU and help detect silent data corruption or hardware stability issues.

> **Note**: This repository is currently on the `opencl` branch and uses `pyopencl` for the stress workload. Switch to the `CUDA` branch if you need the PyTorch-based variant.

### Features

- **Multiple precisions**: Supports FP16, FP32, and FP64. FP16 requires `cl_khr_fp16`; FP64 requires `cl_khr_fp64` or `cl_amd_fp64`.
- **Randomized workloads**: Dynamically changes matrix sizes, including misaligned sizes, to alternate hot and cold compute phases and stress power delivery plus memory allocators.
- **OpenCL platform/device selection**: You can target a specific platform and device, or let the tool auto-select the first available GPU platform.
- **Sidecar validation**: Periodically interrupts the stress loop and runs deterministic CPU-side FP64 reference checks to catch silent errors.
- **Sustained high load**: You can customize the runtime to apply steady or aggressive pressure to the GPU.

### Requirements

- Minimum Python version: `>=3.11`
- An available OpenCL platform, driver/ICD, and a target device with enough memory and compute capability.

### Installation

This project recommends `uv` for virtual environment and dependency management.

1. Clone the repository:

```bash
git clone https://github.com/your-username/NVOC-CLI-Stressor.git
cd NVOC-CLI-Stressor
```

2. Install dependencies and create the environment automatically:

```bash
uv sync
```

If you prefer a pure `venv` + `pip` workflow, install the root-level dependency file directly:

```bash
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

If you want to build a standalone executable, the same `requirements.txt` already includes PyInstaller:

```bash
pip install -r requirements.txt
```

Then run:

```bash
pyinstaller NVOC-CLI-Stressor-opencl.spec
```

The default output goes to `dist/NVOC-CLI-Stressor-opencl/`. The current spec uses the repository entry script as a template; if you switch to a dedicated OpenCL bootstrap file, update `entry_script` in the spec.

### Usage

Run the stress test directly with `uv`:

```bash
uv run test.py [arguments]
```

Or use the current Python environment:

```bash
python test.py [arguments]
```

#### Available arguments

- `--platform-index`: OpenCL platform index; auto-selects the first available GPU platform when omitted.
- `--device-index`: GPU index within the selected platform; requires `--platform-index`, defaults to `0`.
- `--list-devices`: List visible OpenCL platforms and devices, then exit.
- `--duration` (default: 90.0): Stress duration per precision mode, in seconds.
- `--matrix-sizes` (default: `2049, 4096, 4097, 8192, 8193, 16384`): Comma-separated list of random matrix sizes for the main stress workload.
- `--fp64-matrix-sizes` (default: `2048, 4096`): Matrix sizes dedicated to FP64 mode to avoid very slow double-precision phases.
- `--precisions` (default: `fp16,fp32`): Precision list to test. Supported values: `fp16`, `fp32`, `fp64` (skipped automatically when unsupported).
- `--warmup-iters` (default: 3): Warmup rounds for each workload window.
- `--burst-iters` (default: 6): Main stress rounds for each workload window.
- `--validate-interval` (default: 10): Interval, in seconds, for sidecar validation.
- `--validate-size` (default: 768): Fixed matrix size used by validation.
- `--transpose-prob` (default: 0.5): Probability of randomly transposing matrix A/B to perturb the kernel path.
- `--jitter-rate` (default: 0.3): Probability of selecting a jittered matrix size instead of the preferred one.
- `--min-burst-ms` (default: 40.0): Minimum runtime per workload window, in milliseconds.
- `--input-refresh-interval` (default: 1): Number of windows to reuse the same input tensors before refreshing them.
- `--seed` (default: 12345): Random seed for reproducible runs.

For the full, always-up-to-date argument list, run `python test.py --help`.

### Log overview

- Detects and prints basic device architecture information such as compute capability.
- Continuously reports matrix size, instantaneous throughput (TFLOPS), and validation progress.
- After each precision test, the final summary shows whether any runtime error occurred.

### License

This project is licensed under the Apache License 2.0. See the root `LICENSE` file for details.

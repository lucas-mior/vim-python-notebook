# notebook-python.vim

A small Jupyter-like Python notebook runner for Vim. It lets you split a normal
`.py` file into cells with `# %%`, run cells through a persistent stateful
Python kernel, and display generated stdout, expression results, errors,
timings, and Matplotlib figures directly in the buffer.


## Disclaimer
This is highly experimental and certainly has bugs.

## How to use it

The plugin is intentionally inactive by default. It only enables notebook
behavior for Python buffers that contain an activation comment near the top.

```python
# notebook-python: enable
```

or:

```python
# nb: enable
```

## Demonstration
https://github.com/user-attachments/assets/838b741f-2979-4533-9534-a4eed139d461

## Features

- Stateful per-buffer Python kernel.
- Run all cells from a fresh kernel.
- Run the current cell asynchronously without blocking Vim.
- Running/queued status blocks below cells.
- Output updates while cells finish.
- Editing the buffer while a cell is running is supported; generated output is
  anchored by status markers.
- Optional per-cell elapsed timings in `# nb-output` headers.
- Matplotlib figure capture and terminal/overlay rendering support.
- No default key mappings for notebook commands; add your own mappings in your
  vimrc.

## Requirements

### Required

- Vim with Vim9script support.
- Vim job/channel support: `job_start()`, `job_getchannel()`, `ch_sendraw()`,
  `ch_readraw()`.
- Python 3 available as `python3` in your `PATH`.

### Optional, depending on what you use

For Matplotlib figure generation:

- Python package: `matplotlib`

For figure sizing / image preparation:

- Python package: `Pillow`

For terminal image drawing, depending on `g:python_notebook_draw_engine`:

- `chafa` for the default `chafa` engine.
- ImageMagick's `magick` command for the `magick` engine.
- `ueberzugpp` for the `ueberzugpp` engine.

Your Python notebook code may also require whatever packages it imports, such as
`numpy`, `pandas`, or `scipy`.

## Install with vim-plug

Create a Git repository from this directory, or unpack/copy it somewhere local,
then add one of these to your vimrc.

For a local checkout:

```vim
Plug '~/path/to/vim-python-notebook'
```

For your own hosted repository:

```vim
Plug 'lucas-mior/vim-python-notebook'
```

Then run:

```vim
:PlugInstall
```

The repository layout is:

```text
notebook-python-vim/
├── README.md
└── plugin/
    ├── notebook-python.vim
    └── notebook-python-draw.py
```

The helper script is stored next to the Vim plugin file, which matches the
plugin's default helper lookup.

## Basic usage

Create a Python file with the activation comment near the top:

```python
# notebook-python: enable

# %%
x = 10

# %%
x + 5
```

Open the file in Vim. If the plugin does not enable automatically, run:

```vim
:PythonNotebookTryEnable
```

Then use the commands below.

## Commands

### Buffer/global commands

These commands will try to enable notebook mode for the current buffer if
needed:

```vim
:PythonNotebookTryEnable
:PythonNotebookRunAll
:PythonNotebookRunCell
:PythonNotebookRestartKernel
:PythonNotebookClearOutputs
```

### Buffer-local commands

Once notebook mode is active, the same main commands are also defined
buffer-locally for the notebook buffer:

```vim
:PythonNotebookRunAll
:PythonNotebookRunCell
:PythonNotebookRestartKernel
:PythonNotebookClearOutputs
:PythonNotebookDrawFigures
```

### Image/debug commands

```vim
:PythonNotebookStartUeberzugpp
:PythonNotebookStopUeberzugpp
:PythonNotebookStartImagePrepWorker
:PythonNotebookStopImagePrepWorker
```

## Suggested mappings

The plugin does not install default mappings. Add your preferred mappings to
your vimrc.

Example:

```vim
augroup MyPythonNotebookMaps
  autocmd!
  autocmd FileType python nnoremap <buffer> <leader>na :PythonNotebookRunAll<CR>
  autocmd FileType python nnoremap <buffer> <leader>nr :PythonNotebookRunCell<CR>
  autocmd FileType python nnoremap <buffer> <leader>nk :PythonNotebookRestartKernel<CR>
  autocmd FileType python nnoremap <buffer> <leader>nc :PythonNotebookClearOutputs<CR>
augroup END
```

A simple alternative is to use global mappings:

```vim
nnoremap <leader>na :PythonNotebookRunAll<CR>
nnoremap <leader>nr :PythonNotebookRunCell<CR>
nnoremap <leader>nk :PythonNotebookRestartKernel<CR>
nnoremap <leader>nc :PythonNotebookClearOutputs<CR>
```

## Cell syntax

Cells begin with:

```python
# %%
```

Example:

```python
# notebook-python: enable

# %%
import math

# %%
math.sqrt(81)
```

The last expression in a cell is evaluated and shown as a result, similar to a
notebook.

## Generated blocks

The plugin writes generated blocks into your Python file as comments.

Output:

```python
# nb-output: start [stdout, result]
# hello
# 15
# nb-output: end
```

Error:

```python
# nb-error: start
# Traceback ...
# nb-error: end
```

Running status:

```python
# nb-status: start [running] id=... cell=...
# running...
# nb-status: end
```

These blocks are removed by:

```vim
:PythonNotebookClearOutputs
```

## Stateful execution model

The current-cell command uses a persistent kernel for the buffer.

For example:

```python
# %%
x = 10

# %%
x + 5
```

If you run the first cell, then the second cell, the second cell can see `x`.

Use this to clear kernel state:

```vim
:PythonNotebookRestartKernel
```

`PythonNotebookRunAll` runs all cells from a fresh kernel and streams cell
outputs as each cell completes.

## Settings
```vim
" Show timings for each cell
let g:python_notebook_show_cell_timings = 1

" Helper script path
let g:python_notebook_helper = '/path/to/notebook-python-draw.py'
" Cache directory
let g:python_notebook_cache_dir = '~/.cache/notebook-python-vim'
" Stop run-all on first error
let g:python_notebook_stop_on_error = 1
" Show per-cell timings
let g:python_notebook_show_cell_timings = 1
" Activation scan range
let g:python_notebook_annotation_scan_lines = 120
" Figure display maximum height
let g:python_notebook_figure_lines = 40
" Figure drawing engine
let g:python_notebook_draw_engine = 'chafa'
let g:python_notebook_draw_engine = 'magick'
let g:python_notebook_draw_engine = 'ueberzugpp'
" Terminal cell pixel size
" (fallback in case your terminal does not respond to
the escape sequences)
let g:python_notebook_cell_width = 10
let g:python_notebook_cell_height = 20

" x11, wayland, sixel (don't use it), kitty, chafa (don't use it)
let g:python_notebook_ueberzugpp_output = 'x11'

let g:python_notebook_ueberzugpp_stdout_to_tty = 0

" Image prep worker settings
let g:python_notebook_image_prep_worker_timeout_ms = 2000
let g:python_notebook_image_prep_worker_cache_size = 16
```

## Matplotlib example

```python
# notebook-python: enable

# %%
import matplotlib.pyplot as plt

# %%
plt.plot([1, 2, 3], [1, 4, 9])
plt.title('Example')
```

The plugin saves generated figures under its cache directory and inserts `#
nb-figure:` references into the buffer.

## Troubleshooting

### Notebook mode does not activate

Check that:

- The filetype is `python`, or the file extension is `.py`.
- One of these comments is near the top of the file:

```python
# notebook-python: enable
# nb: enable
```

Then run:

```vim
:PythonNotebookTryEnable
```

### Python cannot run

Check:

```sh
python3 --version
```

### Figures do not display

For the default engine, check:

```sh
python3 -c 'import PIL'
chafa --version
```

For Matplotlib figures, check:

```sh
python3 -c 'import matplotlib'
```

### Kernel state is stale

Restart it:

```vim
:PythonNotebookRestartKernel
```

### Outputs are in the file

Generated output/status/error blocks are intentionally stored as Python comments
in the buffer. Remove them with:

```vim
:PythonNotebookClearOutputs
```

## Notes

- `input()` is not supported inside notebook cells.
- Normal `print()` output is captured. Low-level writes directly to file
  descriptor 1 may interfere with the JSON protocol used by the kernel worker.
- Because this is a stateful notebook runner, rerunning cells out of order can
  leave old variables in the kernel namespace, just like in Jupyter.

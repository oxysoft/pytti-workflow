# pytti-workflow

An upgraded workflow for PyTTI users.

## Instructions

You must re-make your notebook with each new version of PyTTI, with the following instructions:

1. Save PyTTI to your GDrive.
2. Enable auto-mount of GDrive.
3. delete the mounting, installation, parameters, and load json cells.
4. Move the instruction cells all the way to the bottom of the notebook or copy it to a separate instructions notebook.
5. Move the run cell to the top of the notebook.
6. Above the note cell, add the userscript cell:

```Python
# Inject our userscript api
%run -i '/content/drive/MyDrive/ai/pytti-userscript-api.ipy'

set_defaults()

# Path relative to gdrive root
# Specifies the root dir for resources (masks, init image, etc.)
resources_dir = "ai/tentacle-scene" 

setup(
  title = 'my-project',
  w     = 640,
  h     = 360
)

p(-1, 'greek letters', -0.75)
p(-1, 'latin alphabet letters text', -0.75)
p(-1, 'font text', -0.75)
p(-1, 'dictionary definitions stock photo', -0.75)

p(3.4, "Thick rolling clouds")
```

6. Add this at the very beginning of the run cell

```python
#############################################################################
# @markdown ---
import os

# Config -------------------------------------------------------
enable_nvidia_error_checking = False #@param{type: "boolean"}
restore  = False #@param{type: "boolean"}
restore_run =  0#@param{type:"raw"}
last_title = 0

if 'params' not in globals():
  set_defaults()

if enable_nvidia_error_checking:
  !nvidia-smi --query-gpu=gpu_name --format=csv | tail -1

# RUN THE USERSCRIPT
# These features work, but I found that it takes way too long for changes to
# propagate onto the servers when I save, using a mount on linux
# if script_path.endswith('.py'): # PYTHON USERSCRIPT
#   print("LOADING USERSCRIPT ...")
#   %run -i $script_path
# elif script_path.endswith('.json'): # JSON CONFIG
#   with open(script_path, 'r') as file:
#     print("LOADING JSON...")
#     text = file.read()
#     load_json(text)

# CELL USERSCRIPT
path = "/drive/MyDrive/Colab_Notebooks/Nuck-PyTTI.ipynb"
# if os.path.isfile(path):
#   print("LOADING CELL ...")
#   text = !cat $path
#   o = Bunch(json.loads(text[0]))
#   exec(''.join(o.cells[0]['source']))


# POSTPROCESSING -------------------------
params.save_every = params.steps_per_frame

if 'draft' in globals():
  params.width  = int(params.width / draft)
  params.height  = int(params.height / draft)
  params.display_scale = max(params.display_scale, draft)

params.display_every = display_every
params.clear_every   = display_every * 50 # images
params.save_every    = params.steps_per_frame

if solo_output:
  params.display_every = 1
  params.clear_every   = 1

# ERROR CHECKING

if params.border_mode not in ['wrap', 'mirror', 'smear', 'clamp', 'black']:
  raise Exception(f'ERROR: Invalid border_mode={params.border_mode}')

if params.infill_mode not in ['mirror', 'wrap', 'black', 'smear']:
  raise Exception(f'ERROR: Invalid infill_mode={params.infill_mode}')

if params.scenes == '':
  raise Exception('ERROR: No scenes.')
  
if params.animation_mode == '3D' and params.border_mode == 'mirror':
  raise Exception('ERROR: cannot use mirror with animation_mode == 3D.')


# REVISION  -------------------------
images_path = "/content/drive/MyDrive/pytti_test/images_out/"
revision = -1

if enable_auto_revision:
  revision = 1
  while True:
    dirpath = f'{images_path}{params.title} (rev-{revision})'
    print(dirpath)
    if not os.path.isdir(dirpath) and not os.path.exists(dirpath):
      break
    revision += 1

if restore:
  revision -= 1 # Take the latest rev instead so we can restore
    


# TITLE -------------------------
if enable_auto_revision and 'title' in params and revision > -1:
  params.file_namespace = f'{params.title} (rev-{revision})'
else:
  params.file_namespace = params.title
  print(params.file_namespace)



# PRINT -------------------------
print("")
print(f'Project: {params.file_namespace}' )
print(f'Draft: {draft}')
print(f'Seed: {params.seed}')
print(f'Dimensions: {params.width}x{params.height}')
# print(f'Prompts: /init')
print(f'Init: {params.init_image if params.init_image != "" else "N/A"}')
print(f'Json: {json.dumps(params)}')
print("")


######################################################################
```

7. Add this cell below or above the run cell (I prefer below since I rarely change them)
   This is the options cell.

```python
# @markdown ---
# @markdown ## Userscript

script_path = "/content/drive/MyDrive/ai/pytti-userscript.py" #@param{type: "string"}
enable_auto_revision = True #@param{type: "boolean"}

# @markdown ---
# @markdown ## Run


# @markdown ---
# @markdown ## Output

draft    = 1.0 #@param{type: "raw"}
solo_output  = False #@param{type: "boolean"}
display_every = 100#@param{type: "raw"}
```

## Usage

Upon first connecting to your VM, run the userscript and option cells. After that, you can simply alternate between run and userscript to iterate on your project quickly.

I use these keyboard shortcuts:

* `Ctrl-\` Interrupt Execution
* `Ctrl-Enter` Execute current cell
* `Ctrl-Alt-Enter` Restart runtime

When you are focused in the userscript text editor, press escape. This will move the focus to the cell and then you can simply navigate with the arrow keys. Press enter to snap back into the editor. With this workflow, we can work all from the keyboard.

## Features

* This adds a scripting API to facilitate reading and editing the userscript. Read through userscript-api to see the usable functions.
* Instead of always spitting your images in the same directory, each run is in a separate directory with a rev-# suffix appended to it. You can disable this in the option cell.

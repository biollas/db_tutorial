(Note: the Dreambooth images I uploaded were only trained with 300 epochs. They'd look better with more, but they look good enough for this purpose.

https://youtu.be/S1jFzeZA4aw

# Download Python
- https://www.python.org/downloads/release/python-3106/

# Download Automatic1111
-  Download the following commit: https://codeload.github.com/AUTOMATIC1111/stable-diffusion-webui/zip/5ef669de080814067961f28357256e8fe27544f4
-  Unzip it somewhere.
-  Open /requirements_versions.txt and add httpx==0.24.1 at the very end.
-  Run webui-user.bat, let the script run and install things.
-  Once it’s installed, it’ll pop up on your browser, but for now just close down the cmd window.


# Download sd_dreambooth_extension
- Download the following commit:
https://codeload.github.com/d8ahazard/sd_dreambooth_extension/zip/dc413a14379b165355502d9f65856c40a4bb5b6f
- Unzip the files into ./extensions/sd_dreambooth_extension
- Go to ./venv/Scripts and open a cmd, type the following:
  - activate
  - pip install https://github.com/jllllll/bitsandbytes-windows-webui/releases/download/wheels/bitsandbytes-0.41.1-py3-none-win_amd64.whl
  - pip uninstall bitsandbytes
- Open webui-user.bat and have 
  - set COMMANDLINE_ARGS= --xformers
- Run webui-user.bat again. Close it when it’s done.
- Optional: open webui-user.bat and have 
  - set COMMANDLINE_ARGS= --xformers --skip-install
which will enable “offline mode” and, more importantly, will prevent it from trying to update anything. In the world of Automatic1111, updates are bad.

# Preparing Photos
- Get photos
  - Roughly around 10 chest-up photos, 5 waist-up photos, 2-5 full body shots
    - Use different backgrounds
    - Use a handful of facial expressions, or otherwise use the expression you know you’ll want in your end result
    - Use different angles, or otherwise use the angle you know you’ll want in your end results
    - You can have more, of course, but I find the returns are rapidly diminishing at about 20 for most cases unless you really want a wide variety of facial expressions.
    - Crop and resize them to 512x512 png files
    - Give them the same name you’ll give your Dreambooth model, e.g. busey (1).png, busey (2).png, etc
  - In Automatic1111, click on the Train tab
    - In Preprocess images, Source directory is where your photos are. For example, mine are ./busey/before/
    - Destination directory is where your final photos will end up. Mine will be ./busey/processed/
    - Click Use CLIP for caption
    - Click the Preprocess button
    - In ./busey/processed/, edit the txt files for each png to make sure they make sense and are correct. 
    - Example: “a man with blonde hair and a jacket on smiling for a picture in front of a red curtain with a red curtain behind him” I’ll change to “a man with blonde hair and a jacket on smiling for a picture in front of a red curtain”
    - No need to be more specific than this, really; most important is to make sure the class token is in there – in this case, “man”. Use only one word for the class token. Also useful to get the word for the expression in there, e.g. “smiling” or “looking angry” or etc if you think you’ll want to re-create those expressions.

# Preparing Dreambooth
- Click on the Dreambooth tab, and under Model click Create.
  - Name your model. Mine will be “busey”
  - Choose the Source Checkpoint, i.e. the model on which you want to train. I’ve had good results with RealisticVision, but I’ll just use the default v1-5-pruned thing that comes with it.
  - Click 512x Model, nothing else checked, click Create Model
- Set parameters
  - Click on Performance Wizard (WIP) if you’d like.
  - In my cases the settings it chooses are pretty good. I’m on a 3060 with 12GB of VRAM to play around with. But regardless, here’s what I use:
  - Training Steps: 600. (Keep in mind if you want to do 300 now and 300 later, then you’ll set this to 300).
  - Save Model Frequency & Save Preview(s) Frequency: 0. This gave me errors a while back. Might work for you if you leave it on.
  - Batching: 1 for everything
  - Set Gradients to None while Zeroing: checked
  - Gradient Checkpointing: unchecked
  - Learning rates: 0.000001 
  - Learning Rate Scheduler: constant_with_warmup with 0 steps
  - 512 max resolution
  - Apply Horizonal Flip can be useful if you have relatively few photos. I always just leave it on for cheap additional learning variety.
  - Use EMA: unchecked
  - Optimizer: 8bit AdamW
  - Mixed Precision: bf16
  - Memory Attention: xformers
  - Cache Latents: unchecked
  - Train UNET: checked
  - Step ratio of Text Encoder Training: 0.75
  - Offset Noise: 0
  - Clip skip: 1
  - Weight Decay: 0.01
  - TENC Weight Decay: 0.01
  - TENC Gradient Clip Norm: 0
  - Pad Tokens: checked
  - Strict Tokens: unchecked
  - Shuffle Tags: checked
  - Max Token Length: 75
  - ScalePrior Loss: unchecked
  - Prior Loss Weight: 0.75
- Click on Concept 1
  - Dataset directory: ..\photos\busey\processed
  - Classification Dataset Directory: ..\photos\busey\classification
  - Instance token: ohwx. This is just the word the training subject will be associated with in the prompting. The idea is that ohwx is a nonsense term and therefore has little correlation with anything else already existing in the model, which makes it a clean choice for a new concept.
  - Class token: man
  - Instance Prompt: [filewords]
  - Class Prompt: [filewords]
  - What [filewords] does is look in the txt files you made to describe the photos, and it’ll insert “ohwx” in there where you wrote “man”, which is how it connects everything together
  - Classification Image Negative Prompt: ugly, disfigured, blurry, meme, cartoon, anime, black and white
  - Class Images Per Instance Image: 100
  - Classification CFG Scale: 7.5
  - Classification Steps: 40
  - What this does is creates pictures of “man” as described in your txt files so the model can retain some idea of what “man” (and the other descriptors) means beyond the new subject you’re injecting into the model
  - This could take a while, depending on how many photos you have.
  - You can try skipping this step; in my experience the end result looks a bit more plasticy if you don’t use any class images.

# Generate classification images
- Click Generate
  - Click Generate Class Images
  - This could take a while. I have 3 photos, so it’ll have to generate 300 classification photos. IMO worth it, but again, you can try with 0 classification images if you want a quicker proof of concept.

# Train
- Click Train
  - This could also take a while. With 5 or fewer photos, I might do up to 750 epochs total. Otherwise 600 seems like a good spot. If you do too many epochs, then the model will be overtrained and will become “rigid” insofar as it’ll basically just re-create the original photos. Sometimes you’ll see strange artifacts too, like hair growing on their forehead or something.
  - If you want to see how it evolves, could be worth doing 200, then another 200, then another 200, then another 100, and then see which you like best. There's always the trade-off between flexibility (less training) and specificity (more training).

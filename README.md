## Issue
The Dockerfile uses `nvcr.io/nvidia/cudagl:11.3.0-devel-ubuntu20.04` as base image, which provides great integration of CUDA and OpenGL. However, the absent of `GLIBCXX_3.4.30` in Ubuntu 20.04 making the testing example of OpenFMNav unable to be executed as expected (which is weird because the original repo has mentioned that the code has tested on Ubuntu 20.04). The returned error message:
```
OSError: Could not find/load shared object file: libllvmlite.so
Error was: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.30' not found (required by /opt/conda/envs/habitat/lib/python3.7/site-packages/llvmlite/binding/../../../../libLLVM-11.so)
```

Since the official nvidia/cudagl base image is no longer receiving updates and does not support Ubuntu 22.04, I have tried methods (listed in the Reference section) that integrate CUDA with OpenGL in Ubuntu 22.04, but none of them works as expected, receiving error message like: `Platform::WindowlessEglApplication::tryCreateContext(): unable to find EGL device for CUDA device 0`. 

### Reference
[CUDA+OpenGL Dockerイメージの作成方法](https://qiita.com/ntrlmt/items/c0a807aeff83eea7be24) \
[Docker Image with cuda and ROS2 on Ubuntu 22.04](https://stackoverflow.com/questions/76081863/docker-image-with-cuda-and-ros2-on-ubuntu-22-04)

## To reproduce
1. Build the container via Vscode Dev Containers\
`View` -> `Command Palette...` -> `Dev Containers: Rebuild and Reopen in Container`

2. Activate habitat-sim environment
```
source activate
conda activate habitat
```

3. Download dataset into /data
```
# Object goal data
wget https://dl.fbaipublicfiles.com/habitat/data/datasets/objectnav/hm3d/v1/objectnav_hm3d_v1.zip
unzip objectnav_hm3d_v1.zip -d data
mv data/objectnav_hm3d_v1 data/objectgoal_hm3d
rm objectnav_hm3d_v1.zip
# Scene data
python -m habitat_sim.utils.datasets_download --username <your token ID> --password <your token secret> --uids hm3d_val
```

4. Create apikey.txt and copy the OpenAI API key there
You can find (or generate) your API key(s) [here](https://platform.openai.com/api-keys)
```
touch apikey.txt
echo "<your api key>" > apikey.txt
```

5. Download the pretrained weights for [Grounded-SAM](https://github.com/IDEA-Research/Grounded-Segment-Anything)
```
cd Grounded_SAM
wget https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth
wget https://github.com/IDEA-Research/GroundingDINO/releases/download/v0.1.0-alpha/groundingdino_swint_ogc.pth
cd ..
```

6. Finally, run the example command provided by the original OpenFMNav repo
```
CUDA_VISIBLE_DEVICES=0 python main.py --split val --eval 1 --auto_gpu_config 0 --prompt_type scoring \
-n 1 --num_eval_episodes 100 --text_threshold 0.55 --boundary_coeff 12 --start_episode 0 --tag_freq 100 \
--use_gtsem 0 --num_local_steps 20 --print_images 1 --exp_name test
```
A fork of [yining043/VRP-DACT](https://github.com/yining043/VRP-DACT)

---

# VRP-DACT
DACT is a learning based improvement model for solving routing problems (e.g., TSP and CVRP), which explores dual-aspect representation, dual-aspect collaborative attention (DAC-Att) and cyclic positional encoding (CPE). It is trained by n-step Proximal Policy Optimization (PPO) with a curriculum learning (CL) strategy.

![](https://raw.githubusercontent.com/yining043/TSP-improve/master/outputs/ep_gif_0.gif)

# Paper
![architecture](./architecture.jpg)

This repo implements our paper: Yining Ma, Jingwen Li, Zhiguang Cao, Wen Song, Le Zhang, Zhenghua Chen, Jing Tang, “[Learning to iteratively solve routing problems with dual-aspect collaborative transformer](https://arxiv.org/abs/2110.02544),” in Advances in Neural Information Processing Systems, vol. 34, 2021. Please cite our paper if the code is useful for your project.
```
@inproceedings{ma2021learning,
 author = {Ma, Yining and Li, Jingwen and Cao, Zhiguang and Song, Wen and Zhang, Le and Chen, Zhenghua and Tang, Jing},
 booktitle = {Advances in Neural Information Processing Systems},
 editor = {M. Ranzato and A. Beygelzimer and Y. Dauphin and P.S. Liang and J. Wortman Vaughan},
 pages = {11096--11107},
 publisher = {Curran Associates, Inc.},
 title = {Learning to Iteratively Solve Routing Problems with Dual-Aspect Collaborative Transformer},
 url = {https://proceedings.neurips.cc/paper/2021/file/5c53292c032b6cb8510041c54274e65f-Paper.pdf},
 volume = {34},
 year = {2021}
}
```

# One more thing
You may also be interested in our new approach called [N2S](https://github.com/yining043/PDP-N2S) (IJCAI 2022) which makes DACT more efficient for solving pickup and delivery problems (PDPs).

Paper: Yining Ma, Jingwen Li, Zhiguang Cao, Wen Song, Hongliang Guo, Yuejiao Gong and Yeow Meng Chee, “[Efficient Neural Neighborhood Search for Pickup and Delivery Problems](https://arxiv.org/abs/2204.11399),” in the 31st International Joint Conference on Artificial Intelligence and the 25th European Conference on Artificial Intelligence (IJCAI-ECAI 22), 2022.

# Dependencies
* Python>=3.6
* PyTorch>=1.1
* numpy
* tensorboard_logger
* tqdm
* cv2
* matplotlib

### Note:
For the exception below from package tensorboard_logger,
```python
AttributeError: module 'scipy.misc' has no attribute 'toimage'
```
Please refer to [issue #27](https://github.com/TeamHG-Memex/tensorboard_logger/issues/27) to fix it.


# Usage
## Generating data
Training data is generated on the fly. Please follow repo [wouterkool/attention-learn-to-route](https://github.com/wouterkool/attention-learn-to-route) to generate validating or test data if needed. We also provide some randomly generated data in the  [datasets](./datasets) folder.

## Training
### TSP example
For training TSP instances with 50 nodes and GPU cards {0,1}:
```python
CUDA_VISIBLE_DEVICES=0,1 python run.py --problem tsp --graph_size 50 --step_method 2_opt --n_step 4 --T_train 200 --Xi_CL 2 --max_grad_norm 0.2 --val_m 1 --val_dataset  './datasets/tsp_50_10000.pkl' --run_name 'example_training_TSP50'
```

### CVRP example
For training CVRP instances with 20 nodes and GPU cards {0,1}:
```python
CUDA_VISIBLE_DEVICES=0,1 python run.py --problem vrp --graph_size 20 --dummy_rate 0.5 --step_method 2_opt --n_step 10 --T_train 500 --Xi_CL 1 --max_grad_norm 0.04 --val_m 1 --val_dataset  './datasets/cvrp_20_10000.pkl' --run_name 'example_training_CVRP20'
```

### Warm start
You can initialize a run using a pretrained model by adding the --load_path option:
```python
--load_path '{add model to load here}'
```
### Resume Traning
You can resume a training by adding the --resume option:
```python
--resume '{add last saved checkpoint(model) to resume here}'
```
The Tensorboard logs will be saved to folder "logs" and the trained model (checkpoint) will be saved to folder "outputs". Pretrained models are provided in the [pretrained](./pretrained) folders.

## Inference
Load the model and specify the iteration T for inference (using --val_m for data augments):

```python
--eval_only 
--load_path '{add model to load here}'
--T_max 10000 
--val_size 10000 
--val_dataset '{add dataset here}' 
--val_m 8
```

### Examples
For inference 10,000 TSP instances with 100 nodes and no data augment:
```python
CUDA_VISIBLE_DEVICES=0,1,2,3 python run.py --problem tsp --graph_size 100 --step_method 2_opt --eval_only --load_path 'pretrained/tsp100-epoch-195.pt' --T_max 10000 --val_size 10000 --val_dataset  './datasets/tsp_100_10000.pkl' --val_m 1 --no_saving --no_tb
```
For inference 512 CVRP instances with 100 nodes and 8 data augments:
```python
CUDA_VISIBLE_DEVICES=0,1,2,3 python run.py --problem vrp --graph_size 100 --dummy_rate 0.2 --step_method 2_opt --eval_only --load_path 'pretrained/cvrp100-epoch-193.pt' --T_max 10000 --val_size 512 --val_dataset  './datasets/cvrp_100_10000.pkl' --val_m 8 --no_saving --no_tb
```
See [options.py](./options.py) for detailed help on the meaning of each argument.

# Documentation
Please note that in our implementation, the VRP solution is stored in a linked list format. Let us consider a TSP-20 solution [ 6 -> 17 -> 3 -> 9 -> 16 -> 4 -> 12 -> 0 -> 1 -> 5 -> 13 -> 19 -> 11 -> 18 -> 8 -> 14 -> 15 -> 7 -> 2 -> 10 -> 6], we would store this solution as rec = torch.tensor([[ 1, 5, 10, 9, 12, 13, 17, 2, 14, 16, 6, 18, 0, 19, 15, 7, 4, 3,8, 11]]),. Here, if rec[i] = j, it means the node i is connected to node j, i.e., edge i-j is in the solution. For example, edge 0-1, edge 1-5, edge 2-10 are in the solution, so we have rec[0]=1, rec[1]=5 and rec[2]=10.

# Acknowledgements
The code and the framework are based on the repos [wouterkool/attention-learn-to-route](https://github.com/wouterkool/attention-learn-to-route) and [yining043/TSP-improve](https://github.com/yining043/TSP-improve).

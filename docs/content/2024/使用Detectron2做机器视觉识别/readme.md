---
title: 使用 Detectron 2 做机器视觉识别
id: 574579f8-10e2-41b1-8a43-bfe744edea92
createTimestamp: 2024-01-16T16:27:00+08:00
updateTimestamp: 2024-01-16T16:27:00+08:00
sortTimestamp: 2024-01-16T16:27:00+08:00
tags:
  - doc
  - tech
---

最近准备从 [MMdetection 框架](https://mmdetection.readthedocs.io/zh-cn/latest/overview.html) 迁移到 [Detectron2 框架](https://detectron2.readthedocs.io/en/latest/) 做机器视觉识别, 不得不说 Detectron2 的配置和使用方式是真的简单.

从安装开始, 基本上只需要几条指令:

```bash
# 创建虚拟环境
conda create -n detectron2 python python=3.8 -y
conda activate detectron2

# 安装 pytorch
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# 拉取 repo 然后安装
git clone https://github.com/facebookresearch/detectron2.git
python -m pip install -e detectron2

# 一般来说到这里就基本能用了
# 不过可能还会缺那么一点点依赖
# 分别手动 pip install 就行了
```

框架的数据集可以使用 COCO 数据集格式, 非常方便.

官方文档有 [提到](https://detectron2.readthedocs.io/en/latest/tutorials/datasets.html#register-a-coco-format-dataset), 如需使用自备　COCO　数据集, 需要先注册数据集.

下面是一个仿照 `tools/train_net.py` 扩展的 `custom_train_net.py`:

```python
from train_net import main, launch
from detectron2.engine.defaults import default_argument_parser

# 注册自己的数据集
from detectron2.data.datasets import register_coco_instances
register_coco_instances(
    "custom-dataset", # 数据集名称
    {},
    "datasets/coco-dataset/coco.json", # 标签数据文件
    "datasets/coco-dataset/images" # 图片文件夹
)

# 剩下的跟原文件一样
if __name__ == "__main__":
    args = default_argument_parser().parse_args()
    print("Command Line Args:", args)
    launch(
        main,
        args.num_gpus,
        num_machines=args.num_machines,
        machine_rank=args.machine_rank,
        dist_url=args.dist_url,
        args=(args,),
    )

```

然后用这条指令启动就行了:

```bash
python "tools/custom_train_net.py" --config-file "configs/COCO-InstanceSegmentation/mask_rcnn_R_101_FPN_3x.yaml" --num-gpus 1 "SOLVER.IMS_PER_BATCH" 2 "SOLVER.BASE_LR" 0.0025 "SOLVER.MOMENTUM" 0.9 "SOLVER.MAX_ITER" 20000 "SOLVER.STEPS" (6000,8000) "DATASETS.TRAIN" "('custom-dataset',)" "DATASETS.TEST" "('custom-dataset',)" "MODEL.WEIGHTS" "./model-weights/any_model.pth" "SOLVER.CHECKPOINT_PERIOD" 10000 "OUTPUT_DIR" "./output-train"
```

其中比较关键的超参数有:

* `--num-gpus 1` GPU 数量, 没什么好说的
* `"SOLVER.IMS_PER_BATCH" 2` 每个 GPU 在训练时能同时处理的图片数量, 提高这个值会消耗更多显存
* `"SOLVER.BASE_LR" 0.0025` 学习率. 刚开始学习可以用高一些的学习率, 比如 0.005
* `"SOLVER.MAX_ITER" 20000` 本次训练进行的 iteration 数量
* `"DATASETS.TRAIN" "('custom-dataset',)"`, `"DATASETS.TEST" "('custom-dataset',)"` 本次训练和测试所采用的数据集名称
* `"MODEL.WEIGHTS" "./model-weights/any_model.pth"` 本次预加载的模型权重文件, 如果不指定则会下载默认权重
* `"OUTPUT_DIR" "./output-train"` 本次训练产生的所有数据应该存放到什么目录
* `"SOLVER.CHECKPOINT_PERIOD" 10000` 每训练多少 iteration 会在 output 文件夹保存一份权重文件

其实这些超参数也可以直接在 Python 代码里加上, 不过看你心情了.

完整的超参数描述文档在 [这里](https://detectron2.readthedocs.io/en/latest/modules/config.html#yaml-config-references).

另外推理过程同上, 只需要改一下 Python 脚本, 先把自己的数据集注册到 Detectron2 里再进行后续步骤就行了.

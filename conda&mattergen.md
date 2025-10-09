---

title: conda install mattergen on shuguang

---
<H1 CENTER>conda&mattergen</H1>

* 首先创建并激活虚拟环境

```python
conda create -n mattergen python==3.10 -y
conda activate mattergen
```



* 安装uv（官方给出最简单的安装依赖方式）

```python
(mattergen) pip install uv
```



* 首先配置git lfs，否则在git clone源码时不会自动下载基础模型

```python
conda install -c conda-forge git-lfs
git lfs install
```



* 借助huggingface手动下载模型

```python
git lfs pull -I checkpoints/<model_name> --exclude="" 
```

`model_name`处可填：

- `mattergen_base`: unconditional base model trained on Alex-MP-20
  `mattergen_base` ：基于 Alex-MP-20 训练的无条件基础模型
- `mp_20_base`: unconditional base model trained on MP-20
  `mp_20_base` ：基于 MP-20 训练的无条件基础模型
- `chemical_system`: fine-tuned model conditioned on chemical system
  `chemical_system` ：基于化学体系条件微调的模型
- `space_group`: fine-tuned model conditioned on space group
  `space_group` : 基于空间群条件微调的模型
- `dft_mag_density`: fine-tuned model conditioned on magnetic density from DFT
  `dft_mag_density` : 基于 DFT 磁密度条件微调的模型
- `dft_band_gap`: fine-tuned model conditioned on band gap from DFT
  `dft_band_gap` : 基于 DFT 能带隙条件微调的模型
- `ml_bulk_modulus`: fine-tuned model conditioned on bulk modulus from ML predictor
  `ml_bulk_modulus` : 基于 ML 预测器体模量条件微调的模型
- `dft_mag_density_hhi_score`: fine-tuned model jointly conditioned on magnetic density from DFT and HHI score
  `dft_mag_density_hhi_score` ：基于 DFT 磁化密度和 HHI 分数联合调优的微调模型
- `chemical_system_energy_above_hull`: fine-tuned model jointly conditioned on chemical system and energy above hull from DFT
  `chemical_system_energy_above_hull` ：基于化学体系和 DFT 形成能联合调优的微调模型



* git clone源码(git clone失败可以通过SSH KEY的方式下载，见文末)

```python
git clone https://github.com/microsoft/mattergen.git
```

> [!note]
>
> 只有使用上述流程才能全部克隆仓库，否则没有toml文件无法下载mattergen



* 使用uv依赖安装mattergen

```python
uv pip install -e .
```



* 到这里Mattergen就已经可以使用了

下面是使用流程：

> 1. 进入mattergen仓库，找到checkpoints文件，hydra配置文件期望用户传入目录，目录内包含
>
> checkpoints/last.ckpt
>
> config.yaml
>
> 所以在MODEL_PATH处填写模型路径应该是`public/home/jxyuan/mattergen/checkpoints/chemical/system`

![tree_mattergen](C:\Users\HP\Desktop\typora笔记\tree_mattergen.png)

> 2. `export MODEL_PATH=public/home/jxyuan/mattergen/checkpoints/chemical/system`
>
> ​          `export RESULTS_PATH=./results`



> 3. 输入mattergen-generate指令开始生成结构

```python
mattergen-generate $RESULTS_PATH --model_path=$MODEL_PATH --batch_size=16 --num_batches=1
```

***

## git clone失效

git clone失败怎么办？可以通过SSH KEY完成

* 首先登陆官网页[Github Setting](https://github.com/settings/profile)

* 点击SSH and GPG keys

* 点击new ssh key

* 在集群终端输入：

```python
ssh-keygen -t ed25519 -C "your_email@example.com"
#上一步会自动创建.ssh目录其中包含id_ed25519.pub文件
cat ~/.ssh/id_ed25519.pub
```

* 之后会输出密钥：类似ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAf8H... your_email@example.com

* 复制粘贴到new ssh key中
* 点击Add ssh key
* 验证：`ssh -T git@github.com`

之后克隆仓库：

```python
git clone git@github.com:miscroft/mattergen.git
```

:smile:



记得运行时不要使用--pretrained_name而是--model_path，否则会通过huggingface拉取
---

title: packmol2

---
<h1> <center>Packmol </h1>

## 第一步、编写输入文件

* 全局设置

```python
tolerance 2.0  #这里建议直接默认为2.0
filetype pdb #根据自己所要添加的单分子的结构文件类型填入，支持pdb、tinker、xyz
             #vesta 将cif文件转为pdb可能会出现报错，在pdb文件不能用的时候建议使用xyz
output test.xyz #输出文件名称，这里的后缀应该不用与输入文件统一，没有测试过，建议统一
#以上是每个输入文件所必须的三行
--------------------------------------------------------------------------------
#下面的参数是可以选择添加的
seed -1 #可以使分子分布的结构每次都不一样，如果没有这行参数，那么结构会一直重复
add_box_sides #可以给输出的文件中的矩形盒子各个方向增加指定距离，避免边界原子和镜像盒子里的出现不合理接触（这个在生成结果里我没有使用，可以不用）
```



* 分子设置

> [!NOte]
>
> 在设置分子约束条件之前，要明确自己的需求：
>
> （1）**我要填充多少原子? **
>
> （2）**我要在多大的矩形盒子里填充分子？（x * y * z）**
>
> 这里的x,y,z决定了矩形盒子大小，简单来说就是<font color = red>**（0，0，0）坐标和（x,y,z）坐标决定的矩形**</font>
>
> 具体的决定方式可以参考上午的pdf文件

明确了自己的需求就可以开始编写input文件了：

```python
tolerance 2.0
filetype xyz
output test.xyz
seed -1

structure 2PACz.xyz #这里添加想要填充的分子
  number 240 #根据需求填写，比如2PACz:TTMB = 4:1总分子数300，那么此处填入240
  inside box 0. 0. 0. 100. 100. 15.
#inside box代表向（0，0，0）和(100,100,15)决定的矩形盒子内填充分子
end structure

structure TTMB.xyz
  number 60
  inside box 0. 0. 0. 100. 100. 15.
end structure
```

保存将**txt**后缀改为**inp**即可进入第二步



## 第二步、打开Julia，输入`using Packmol; run_packmol()`

> [!tip]
>
> 安装时注意选择将Packmol添加到环境变量，如果没有选择可以手动添加一下
>
> 并且每次使用packmol都要输入一次这个代码

（1）输入上述指令回车之后，会弹出输入文件选择框：

![选择框](C:\Users\HP\Desktop\选择框.png)

输入文件的选择是以inp为后缀的文本文件.

> [!IMPORTANT]
>
> 注意，   **输入的分子结构文件**    和    **输入文件**  在<font color = green>**同一路径**</font>下



（2）选择文件之后等待运行即可

![输入](C:\Users\HP\Desktop\输入.png)

![选择](C:\Users\HP\Desktop\选择.png)

![循环次数](C:\Users\HP\Desktop\循环次数.png)

等待收敛:smile_cat:

![完成](C:\Users\HP\Desktop\完成.png)

**收敛成功:smile:，将xyz拖到vesta观察：**



![1](C:\Users\HP\Desktop\1.png)

**不咋均匀:angry:，再跑一次**

![2](C:\Users\HP\Desktop\2.png)

还是不咋样，建议多跑几次或者增加分子数目，最终是批量化生成结构文件的思路。



## 三、批量化生产的思路

* 首先在linux里部署packmol软件，具体参见上午的pdf
* 然后将分子文件和input文件放在同一目录
* 之后借助ai辅助开发自动运行脚本，将结果全部存在在output文件夹内



ChatGpt 5 Thinking大模型给出的脚本（我没有安装linux版本，所以可能存在错误）

```python
#!/usr/bin/env julia
# batch_packmol.jl
# Batch-run Packmol for every .inp in a folder, saving each run's output into ./output
#
# Usage:
#   julia batch_packmol.jl [N] [DIR]
#     N   = number of structures to generate per .inp (default: 1)
#     DIR = directory to scan for .inp files (default: current directory)
#
# Example:
#   julia batch_packmol.jl 10             # run each .inp 10 times in ./
#   julia batch_packmol.jl 5 /data/jobs   # run each .inp 5 times in /data/jobs
#
# Notes:
# - This script rewrites the 'output ...' line of each .inp to point into ./output/
#   with unique filenames: <inpbase>_r001.xyz, <inpbase>_r002.xyz, ...
# - The .inp is *not* modified on disk; we write a temporary copy next to the original .inp.
# - Relative paths to 'structure ...' files are preserved because we keep the temp .inp
#   in the same directory as the original .inp.
# - Requires: Packmol.jl and the Packmol executable accessible in PATH.
#
using Packmol
using Printf

function parse_args(args::Vector{String})
    n = 1
    dir = pwd()
    if length(args) >= 1
        try
            n = parse(Int, args[1])
            if n < 1
                @warn "N must be ≥ 1; defaulting to 1"
                n = 1
            end
        catch
            @warn "First argument is not an integer; defaulting N=1"
            n = 1
        end
    end
    if length(args) >= 2
        dir = args[2]
    end
    return n, abspath(dir)
end

function read_text(path::AbstractString)
    open(path, "r") do io
        read(io, String)
    end
end

# Extract extension from the 'output ...' line; default to .xyz if absent
function output_ext_from_inp(text::String)
    m = match(r"(?im)^\s*output\s+(.+?)\s*(?:#.*)?$", text)
    if m === nothing
        return ".xyz"
    end
    outtok = strip(m.captures[1])
    if startswith(outtok, "\"") && endswith(outtok, "\"")
        outtok = outtok[2:end-1]
    end
    b = splitext(outtok)[2]
    return isempty(b) ? ".xyz" : b
end

# Replace the first 'output ...' line; if none exists, append one at the end
function patch_or_insert_output_line(text::String, outpath::AbstractString)
    out = replace(outpath, "\\" => "/")
    patched, n = replace(text, r"(?im)^\s*output\s+(.+?)\s*(?:#.*)?$" => "output $(out)"; count=1), 0
    if occursin(r"(?im)^\s*output\s+(.+?)\s*(?:#.*)?$", text)
        patched = replace(text, r"(?im)^\s*output\s+(.+?)\s*(?:#.*)?$" => "output $(out)"; count=1)
    else
        patched = text * "\noutput $(out)\n"
    end
    return patched
end

function main(args)
    n, dir = parse_args(args)
    println("Batch Packmol: dir=$(dir), replicates per .inp=$(n)")

    # Discover .inp files (non-recursive)
    inps = filter(f -> endswith(lowercase(f), ".inp"),
                  readdir(dir; join=true))
    if isempty(inps)
        @warn "No .inp files found in $(dir). Nothing to do."
        return
    end

    # Prepare output directory (inside dir)
    outdir = joinpath(dir, "output")
    isdir(outdir) || mkpath(outdir)

    for inp in sort(inps)
        name = splitext(basename(inp))[1]
        txt = read_text(inp)

        ext = output_ext_from_inp(txt)
        ext = isempty(ext) ? ".xyz" : ext

        for k in 1:n
            outpath = joinpath(outdir, @sprintf("%s_r%03d%s", name, k, ext))
            patched = patch_or_insert_output_line(txt, outpath)

            # temp .inp lives next to the original .inp so relative structure paths still work
            tmpinp = joinpath(dirname(inp), @sprintf("%s_r%03d.inp", name, k))
            open(tmpinp, "w") do io
                write(io, patched)
            end

            @info "Running Packmol" inp=basename(inp) replicate=k out=Base.relpath(outpath, dir)
            try
                run_packmol(tmpinp)
            catch e
                @error "Packmol failed" inp=basename(inp) replicate=k error=e
            finally
                # Clean up temp .inp to keep the folder tidy; comment this out if you want to keep them
                try
                    rm(tmpinp; force=true)
                catch
                end
            end
        end
    end

    println("Done. Results saved in: ", outdir)
end

main(ARGS)

```

使用指南：

```python
# 当前目录下每个 .inp 生成 10 个结构
julia batch_packmol.jl 10

# 指定目录 /data/jobs 下每个 .inp 生成 5 个结构
julia batch_packmol.jl 5 /data/jobs

# 不带参数：默认每个 .inp 生成 1 个结构
julia batch_packmol.jl

```

使用须知：

![须知](C:\Users\HP\Desktop\须知.png)
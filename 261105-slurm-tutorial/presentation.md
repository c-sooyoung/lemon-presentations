---
marp: true
theme: kiwi
math: katex
paginate: true
footer: 'SNU <a href="https://celestachang.com">🍋 LEMON lab</a>'
---

<!-- _class: title -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# SLURM tutorial

Last updated: 2026.11.05

정수영

---

## SLURM

SLURM: 서버 연산 자원 (CPU, GPU 등) 분배하는 역할

현재 LEMON 서버:
- CPU 48개 (AMD Ryzen Threadripper PRO 7965WX 24 cores * 2 threads)
- GPU 4개 (NVIDIA RTX 6000 Ada Generation * 4)
- RAM 124 GiB

---

## SLURM

[슬럼 그림]

---

## Checking SLURM queue

`squeue` 로 현재 queue 확인

```
[swim@lemon ~]$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
              7558 lemon_nod     BO-0   pky001  R    9:06:39      1 lemon
              7559 lemon_nod     BO-1   pky001  R    9:06:39      1 lemon
              7560 lemon_nod     BO-2   pky001  R    9:06:39      1 lemon
              7561 lemon_nod     BO-3   pky001  R    9:06:39      1 lemon
              7557 lemon_nod       BO   pky001  R    9:06:42      1 lemon
```

---

## Aside: checking CPU & GPU usage

```sh
source /opt/venvs/nvitop-env/bin/activate && nvitop && deactivate
```

가능하면 연산 돌리기 전에 `nvitop`으로 미리 확인!

---

예시:

```
╞═══════════════════════════════╪══════════════════════╪══════════════════════╪════════════════════╕
│   0  ..0 Ada Generation  Off  │ 00000000:01:00.0 Off │                  N/A │ MEM: ▍ 3.6%        │
│  0%   44C    P2    93W / 300W │   1781MiB / 47.99GiB │     25%      Default │ UTL: ██▎ 25%       │
├───────────────────────────────┼──────────────────────┼──────────────────────┼────────────────────┤
...

[ CPU: ███████▊ 15.6%                                         ]  ( Load Average:  6.13  6.29  6.23 )
[ MEM: ██████▌ 13.0%                                          ]  [ SWP: ███████▌ 34.3%             ]

╒══════════════════════════════════════════════════════════════════════════════════════════════════╕
│ Processes:                                                                            swim@lemon │
│ GPU     PID      USER  GPU-MEM %SM  %CPU  %MEM  TIME  COMMAND                                    │
╞══════════════════════════════════════════════════════════════════════════════════════════════════╡
│   0 2865260 C  pky001  1772MiB  26 183.7   2.5  4:52  /home/shared/MATLAB/R2021a/bin/glnxa64/M.. │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
...                                    ↑
                                       100% ≈ 1 CPU core
```

---

## Adding to SLURM queue

`sbatch`로 queue에 추가:

```
[swim@lemon ~]$ sbatch job.sub 
Submitted batch job 7562                   <- 추가된 job의 id
```

`squeue`로 확인:

```
[swim@lemon ~]$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
              7562 lemon_nod     Si2V     swim PD       0:00      1 (Resources)          <- 새로 추가
              7558 lemon_nod     BO-0   pky001  R    9:18:55      1 lemon
              ...
```

`(Resources)`가 뜨면 연산 자원이 부족해서 대기중이라는 뜻

---

## Removing from SLURM queue

Queue 에서 job 제거:
```
[swim@lemon ~]$ scancel <job_id>
```

<br/>

Queue에 있는 본인의 job들을 모두 제거하고 싶으면:
```
[swim@lemon ~]$ scancel -u $USER
```

---

## Writing a batch file

일반적인 `.sh` 스크립트 파일 쓰는거랑 거의 똑같음

위에 `#SBATCH` 명령어로 SLURM 설정

```
#SBATCH --job-name=Si2V
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8                                  <- CPU 개수 설정
#SBATCH --gres=gpu:rtx-6000ada:1                           <- GPU 개수 설정
#SBATCH --time=100:00:00
#SBATCH --output=/home/swim/slurm-logs/job_%j.log          <- output 파일 경로
```

필요한 변수 이름은 https://slurm.schedmd.com/sbatch.html#SECTION_OPTIONS 에서 확인

---

```bash
#!/bin/bash
#SBATCH --job-name=Si2V
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --gres=gpu:rtx-6000ada:1
#SBATCH --time=100:00:00
#SBATCH --output=/home/swim/slurm-logs/job_%j.log

pwd
hostname
date

export PATH=/home/shared/MATLAB/R2021a/bin:$PATH                      # MATLAB 경로 지정

export PATH=/usr/local/cuda-11.4/bin:$PATH                            # CUDA 경로 지정
export LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64:$LD_LIBRARY_PATH    

source /home/swim/bo-ptycho/venv/bin/activate

YAML="config.yaml"

cat $YAML
python -u main.py $YAML

date
echo "SLURM JOB FINISHED"
```

---

## Using SLURM environment variables

t.b.f.

---

## Managing SLURM

t.b.f.
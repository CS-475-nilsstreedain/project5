# project5
Note: The flip machines do not have GPU cards in them, so CUDA and OpenCL will not run there. (You can compile there, you just can't run.) If your own system has a GPU, you can try using that. You can also use rabbit or the DGX machine.

Also, CUDA is an NVIDIA-only product. It will not run on Intel or AMD GPUs.

Note: as there are two inputs to the tests you will run here (NUMTRIALS and BLOCKSIZE), this is a Pivot Table project.

## Introduction
Monte Carlo simulation is used to determine the range of outcomes for a series of parameters, each of which has a probability distribution showing how likely each option is to happen. In this project, you will take a scenario and develop a Monte Carlo simulation of it, determining how likely a particular output is to happen.

## The Scenario
![image](https://github.com/user-attachments/assets/e179e6e7-c5e9-4272-9b87-1a09db8867e0)

A castle sits on top of a cliff. An amateur band of merceneries is attempting to destroy it.

Normally this would be a pretty straightforward geometric calculation, but these are amateurs. What makes them amateurs you ask? It's because they are not very good at estimating distances, and not very good at aiming their cannon. They can only determine the 5 input parameters within certain ranges.

Your job is to figure out the probability that these doofuses will actually hit the castle. This is a job for multicore Monte Carlo simulation!

## Requirements:
1. The ranges are:
Variable | Meaning | Range
-|-|-
g | Ground distance to the cliff face | 20. - 30.
h | Height of the cliff face | 10. - 20.
d | Upper deck distance to the castle | 10. - 20.
v | Cannonball initial velocity | 10. - 30.
θ | Cannon firing angle in degrees | 70. - 80.

**These are different from Project #1! You will get a different probability.**

2. Run this for at least five different BLOCKSIZEs (i.e., the number of threads-per-block) of 8, 32, 64, 128, and 256, combined with NUMTRIALS sizes of at least 1024, 4096, 16384, 65536, 262144, 1048576, and 2097152. You can use more if you want.
3. Be sure each NUMTRIALS is a multiple of 1024. All of the ones above already are.
4. Record the timing for each combination. For performance, use some appropriate units like MegaTrials/Second.
5. For this one, use CUDA timing, not OpenMP timing. It's already in the sample code.
6. Produce a rectangular table and two graphs:
    1. Performance vs. NUMTRIALS with multiple colored curves of BLOCKSIZE
    2. Performance vs. BLOCKSIZE with multiple colored curves of NUMTRIALS
7. Like Project #1 before, fill the arrays ahead of time with the random values. Send them to the GPU where they can be used as look-up tables. Notice that the constant parameters are different here than in Project #1. Thus, your probability will be different.
8. Chosing one of the runs, tell me what you think the actual probability is.
9. Parallel Fraction and Maximum Speedup don't apply here, so don't compute them.

## Equations
Same as Project #1.

## Developing this Project in Linux
Get the skeleton code here: proj05.cu

Here are .h files you will need (right-click to download each one):
exception.h
helper_functions.h
helper_cuda.h
helper_image.h
helper_string.h
helper_timer.h

On rabbit, here is a working bash script:
```
#!/bin/bash
for t in 1024 4096 16384 65536 262144 1048576 2097152
do
        for b in 8 32 64 128 256
        do
                /usr/local/apps/cuda/cuda-10.1/bin/nvcc -DNUMTRIALS=$t -DBLOCKSIZE=$b -o proj05  proj05.cu
                ./proj05
        done
done
```

On the DGX system, here is a working sbatch script:
```
#!/bin/bash
#SBATCH  -J  MonteCarlo
#SBATCH  -A  cs475-575
#SBATCH  -p  classgputest
#SBATCH  --gres=gpu:1
#SBATCH  -o  montecarlo.out
#SBATCH  -e  montecarlo.err
#SBATCH  --mail-type=BEGIN,END,FAIL
#SBATCH  --mail-user=mjb@oregonstate.edu
for t in 2048 8192 131072 2097152
do
        for b in 8 16 32 64 128
        do
                /usr/local/apps/cuda/11.7/bin/nvcc -DNUMTRIALS=$t -DBLOCKSIZE=$b -o proj05  proj05.cu
                ./proj05
        done
done
```

You can (and should!) write scripts to run the benchmark combinations. If you want to pass in benchmark parameters, the way you did it in g++ (-DNUMTRIALS=$t) notation works fine in nvcc.

Before you use the DGX, do your preliminary development on the rabbit system. It is a lot friendlier because you don't have to run your program through a batch submission. If you have time, take your final performance numbers on the DGX! They will be wonderfully higher than what you got on rabbit.

You can also take your benchmark numbers on your own machine.

## numHits++ Doesn't Work Anymore
In Project #1, you counted the number of times the cannonball hit the castle by saying numHits++. In Project #5, there is no single numHits on the GPU. So, now you will keep an array of hits in GPU memory. Your kernel will put either a 0 or a 1 there for each trial, then send the array back to the CPU, then your CPU program will add them all up.

Remember that CUDA is an NVIDIA-only product. It will not run on Intel or AMD GPUs.

## Your PDF Commentary
Your commentary PDF should:
1. Tell what machine you ran this on
2. What do you think this new probability is?
3. Show the rectangular table and the two graphs
4. What patterns are you seeing in the performance curves?
5. Why do you think the patterns look this way?
6. Why is a BLOCKSIZE of 8 so much worse than the others?
7. How do these performance results compare with what you got in Project #1? Why?
8. What does this mean for what you can do with GPU parallel computing?

## Grading:
Feature | Points
-|-
Correct probability | 10
Performance table | 20
Graph of performance vs. NUMTRIALS with multiple curves of BLOCKSIZE | 20
Graph of performance vs. BLOCKSIZE with multiple curves of NUMTRIALS | 20
Commentary -- explain the trends of the curves | 30
Potential Total | 100

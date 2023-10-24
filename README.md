## A self-supervised workflow for particle picking in cryo-EM
Authors: Donal McSweeney donal.mcs15@gmail.com; Qun Liu qun.liu@gmail.com

## Referecne  
https://doi.org/10.1107/S2052252520007241 

## System requirement
1. NVIDIA GPU card
2. Relion2 or Relion3.0x (not 3.1) compiled with GPU and MPI support if you would like to run the workflow. Not needed for particle picking

## Instatllation
1. Install miniconda `curl https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -o Miniconda3-latest-Linux-x86_64.sh && bash Miniconda3-latest-Linux-x86_64.sh`
2. conda config --add channels conda-forge
3. conda create --name kpicker -c conda-forge python=3.9.13 cudatoolkit=11.2.2 cudnn=8.2.0.121
4. conda activate kpicker 
5. pip install tensorflow==2.10.0 mrcfile scikit-image==0.19.3 scikit-learn==1.1.3 

## Test tensorflow and cuda
python -c 'import tensorflow as tf; tf.test.is_gpu_available()'

## Quickstart
Download kpicker to a directory under the relion project structure as it uses relion for
2D class averaging. All commands should run from the root directory of Relion.
`git clone https://github.com/qun-liu/kpicker`

## Give an alias name for the directory containing motion corrected micrographs  
ln -s /path/to/motition-corrected/micrographs  aligned

## Do ctf correction, select ctf corrected micrographs for iterative training and pikcing, rename its star file as below
mv -f CtfFind/whatever_ctf.star CtfFind/micrographs_defocus_ctf.star

## Pre-processing. Pikcked particles will be under local/aligned/. Ideally, the script reads in defocus values and use them to setup defocus-based picking. 
1) Configure settings in `kpicker/config.ini`
2) Run `python kpicker/build_preprocess.py`. This will generate a bash script for initial (local) picking and 2D class averaging.
3) Run `sh preprocess.sh`
By default, the workflow uses 4 GPUs for 2D class average. Modify preprocess.sh if you use a different number of GPUs.

## Self-Supervised training/particle picking. Pikced particles will be under kpicker/aligned/
1) Configure settings in `kpicker/config.ini`
2) Run `python kpicker/build_workflow.py`. This will generate a bash file for iterative training/picking.
3) Run `sh workflow.sh`
By default, the workflow uses 4 GPUs for 2D class average. Modify workflow.sh if you use a different number of GPUs.

## Training from polished particles star file
python  kpicker/ktraining.py --train_good 'good.star' --particle_size 256   --logdir  'Logfile'   --bin_size 4 --model_save_file 'test_model.h5'


## Picking all micrigraphs: Create an empty directory of Kpicker/aligned if it does not exist, then run
python kpicker/kpicking.py  --input_dir 'linked'  --output_dir 'Kpicker/aligned'  --pre_trained_model 'test_model.h5' --star_file  'CtfFind/micrographs_ctf.star' --threshold 0.9  --threads 10  --particle_size  256 --coordinate_suffix '_kpicker' --bin_size 4


## using Localpicker for initial particle picking based on shapes
for mrcfile in `(ls */*.mrc)`; do ;
python -W ignore kpicker/localpicker.py  --mrc_file=${mrcfile} --particle_size=260 --bin_size=9  --threshold=0.0015 --max_sigma=10;
done


# examples for calculating the many resting state fmri properties. 
# change the input file name and output file for your own data.
# if the preprocessed images are not in the MNI space, you need to input your own brain mask which should be in the same space as the input rsfmri data.
# The same thing for the seed region.
# For matrix calculation using the program named by "conn", you will need a brain atlas such as BN_atlas_274.nii.gz


# tcm. d: segment lenght (should be longer than 20), r: correlation coefficient threshold
tcm -d 30 -r 0.5 -c 30 -m fsl_MNI152_T1_2mm_brain_mask.nii.gz -i rsfmri_inMNIspace.nii.gz -o tcm.nii.gz

# ben
ben -d 3 -r 0.6 -c 30 -m fsl_MNI152_T1_2mm_brain_mask.nii.gz -i rsfmri_inMNIspace.nii.gz -o ben.nii.gz

# crben using dmn pcc as the seed. r can be 0.3 since there will be more segment pairs to be compared than ben.
crben -d 3 -r 0.6 -c 30 -m fsl_MNI152_T1_2mm_brain_mask.nii.gz -roi dmnpcc.nii.gz -i rsfmri_inMNIspace.nii.gz -o crben_pcc.nii.gz

# ctcm using dmn pcc as the seed
ctcm -d 30 -r 0.5 -c 30 -m fsl_MNI152_T1_2mm_brain_mask.nii.gz -roi dmnpcc.nii.gz -i rsfmri_inMNIspace.nii.gz -o ctcm_pcc.nii.gz

# standard fc using dmn pcc as the seed
mvfc -c 30 -m fsl_MNI152_T1_2mm_brain_mask.nii.gz -roi dmnpcc.nii.gz -i rsfmri_inMNIspace.nii.gz -o fc_pcc.nii.gz

# transfer entropy density
pted -d 2 -c 30 -m fsl_MNI152_T1_2mm_brain_mask.nii.gz -i rsfmri_inMNIspace.nii.gz -o tedensity.nii.gz

#fc density
fcd -c 30 -m fsl_MNI152_T1_2mm_brain_mask.nii.gz -i rsfmri_inMNIspace.nii.gz -o fcdensity.nii.gz

# the following commands are to calculate connection matrix between different ROIs defined in the atlas. These are for within-subject calculation.
#crben matrix
crbenconn -d 3 -r 0.6 -c 30 -atlas BN_atlas_274.nii.gz -i rsfmri_inMNIspace.nii.gz -o cr_conn.dat

# ctcm matrix
ctcm_conn -d 30 -r 0.5 -c 30 -atlas BN_atlas_274.nii.gz -i rsfmri_inMNIspace.nii.gz -o ctcm_conn.dat

#correlation based connectome matrix
crbenconn -rfc -c 30 -atlas BN_atlas_274.nii.gz -i rsfmri_inMNIspace.nii.gz -o fc_conn.dat

#pte connectome matrix
PTE_conn -c 30 -atlas BN_atlas_274.nii.gz -i rsfmri_inMNIspace.nii.gz -o pte_conn.dat

#within subject MI based connectivity matrix calculation. 
# The current version contains three different MI algorithms: the traditional histogram based one, KDE, KSG.  the option -opt is to choose the one to be used.
# For using MI_hist with 20 bins:
    MI_conn -c 30 -nbins 20 -i rsfmri.nii.gz -atlas BN_atlas_274.nii.gz -m brainmask.nii.gz -o MI_hist_conn.data \n");  //opt is 2 by default
#  MI_KDE:
    MI_conn -c 30 -opt 0 -i rsfmri.nii.gz -atlas BN_atlas_274.nii.gz -m brainmask.nii.gz -o MI_KDE_conn.data;
#  MI_KSG with a kernel of 4:
  MI_conn -c 30 -opt 1 -d 4 -i rsfmri.nii.gz -atlas BN_atlas_274.nii.gz -m brainmask.nii.gz -o MI_KSG_conn.data;  
# MI_hist and MI_KDE are faster than MI_KSG.  nbins should be determined using 2*pow(N,1/3) where N is the time series length.
# For MI_KSG, -d 4 or -d 5 should be used.

# Between subject or between session connection calculation.
# cross subject or cross session entropy using cross-sample entropy 
csbenroi -d 3 -r 0.3 -atlas BN_atlas_274.nii.gz -i1 rsfmri1.nii.gz -i2 rsfmri2.nii.gz -o csben12.dat

# cross subject or cross session tcm
csctcmroi -d 30 -r 0.3 -itype 1 -atlas BN_atlas_274.nii.gz -i1 rsfmri1.nii.gz -i2 rsfmri2.nii.gz -o csctcm12.dat

# cross subject or cross session pte. d is for the length of the embedding vector. 
csteroi  -d 2 -c 40 -itype 2 -tetype 2 -atlas BN_atlas_274.nii.gz -i1 rsfmri1.nii.gz -i2 rsfmri2.nii.gz -o cspte.dat

# cross subject or cross session MI. d and r are not required
csmiroi  -c 40 -itype 4 -atlas BN_atlas_274.nii.gz -i1 rsfmri1.nii.gz -i2 rsfmri2.nii.gz -o csmi.dat

## connectome matrices are saved in a binary file with the following data format: 1 integer (4 byte) specifying the number of ROIs used in the atlas. Let's name this integer variable as NR. 
## The next section contains NR * NR * NMAPS float numbers (each with 4 bytes).  NMAPS is 1 for FC, CRBEN, MI; 3 for PTE, 3 or 8 for ctcm

## 

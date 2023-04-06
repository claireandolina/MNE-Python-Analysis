# MNE-Python-Analysis

Overview/Abstract
We attempted to replicate the magnetoencephalography analysis performed by Gramfort et al. with their time-frequency mixed-norm estimates (TF-MxNE) algorithm [1] using the MNE-Python software suite [2] to determine the brain regions most involved in processing images shown to the left eye. The source locations and their amplitudes were found, and these results were projected back into the MEG sensor space to determine how much of the data could be explained. Further analysis was performed with the SPM Faces dataset [3], which provides MEG data on the responses to being shown a normal or a scrambled face, in order to discover which regions may be involved in recognizing human faces. The dSPM [4], sLORETA [5], γ-MAP [6], and TF-MxNE solvers were used on the faces dataset to determine the robustness of these inverse techniques.

Full Paper: https://docs.google.com/document/d/15NtHK_71ab98F-WZmyJcLVUbvfW_o5FuVuSxiAeh_C4/edit?usp=sharing

References
[1] 	Gramfort, A., Strohmeier, D., Haueisen, J., Hämäläinen, M. S., & Kowalski, M. (2013). Time-frequency mixed-norm estimates: sparse M/EEG imaging with non-stationary source activations. NeuroImage, 70, 410–422. https://doi.org/10.1016/j.neuroimage.2012.12.051

[2] Gramfort, A., Luessi, M., Larson, E., Engemann, D. A., Strohmeier, D., Brodbeck, C., Goj, R., Jas, M., Brooks, T., Parkkonen, L., & Hämäläinen, M. (2013). MEG and EEG data analysis with MNE-Python. Frontiers in neuroscience, 7, 267. https://doi.org/10.3389/fnins.2013.00267

[3] Henson, R. (2009). Statistical Parametric Mapping :: Multi-modal Face Dataset. Wellcome Centre for Human Neuroimaging. Retrieved March 19, 2023, from https://www.fil.ion.ucl.ac.uk/spm/data/mmfaces/

[4] Dale, A. M., Liu, A. K., Fischl, B. R., Buckner, R. L., Belliveau, J. W., Lewine, J. D., & Halgren, E. (2000). Dynamic statistical parametric mapping: combining fMRI and MEG for high-resolution imaging of cortical activity. Neuron, 26(1), 55–67. https://doi.org/10.1016/s0896-6273(00)81138-1

[5] Pascual-Marqui R. D. (2002). Standardized low-resolution brain electromagnetic tomography (sLORETA): technical details. Methods and findings in experimental and clinical pharmacology, 24 Suppl D, 5–12.

[6] Wipf, D., & Nagarajan, S. (2009). A unified Bayesian framework for MEG/EEG source imaging. NeuroImage, 44(3), 947–966. https://doi.org/10.1016/j.neuroimage.2008.02.059

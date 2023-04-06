# MNE-Python-Analysis

Overview/Abstract
We attempted to replicate the magnetoencephalography analysis performed by Gramfort et al. with their time-frequency mixed-norm estimates (TF-MxNE) algorithm [1] using the MNE-Python software suite [2] to determine the brain regions most involved in processing images shown to the left eye. The source locations and their amplitudes were found, and these results were projected back into the MEG sensor space to determine how much of the data could be explained. Further analysis was performed with the SPM Faces dataset [3], which provides MEG data on the responses to being shown a normal or a scrambled face, in order to discover which regions may be involved in recognizing human faces. The dSPM [4], sLORETA [5], γ-MAP [6], and TF-MxNE solvers were used on the faces dataset to determine the robustness of these inverse techniques.
Introduction and Background
The classic problem in magneto- and electroencephalography (M/EEG) recordings is the inverse problem, where the electromagnetic measurements are attributed to originate from specific areas within the brain. The brain is represented as a collection of dipoles, known as the distributed source model, whose current-induced electromagnetic fields are projected onto the sensors. The number of dipoles in this model far exceeds the number of sensors placed on the scalp, making the inverse problem ill-posed and underdetermined. Linear approaches generally solve for the least-norm solution, using the simplest set of activations to explain the measurements [4-5]. However, these linear solvers tend to yield results that see activation throughout the brain, failing to highlight only the most relevant regions during a task [1]. To address this, nonlinear methods have emerged that induce sparsity spatially, including γ-MAP [6] and MxNE. However, these techniques have either been unable to guarantee temporal coherence (the parts of the brain that are firing remain the same across time) or assume stationarity (the signal’s frequency components remain the same throughout the recording).
In “Time-frequency mixed-norm estimates: Sparse M/EEG imaging with non-stationary source activations” [1], Gramfort et al. propose the time-frequency mixed-norm estimates (TF-MxNE) algorithm, which attempts to account for transient signals by using a Gabor transform (short-time Fourier transform with a Gaussian window) and obtaining the most relevant Gabor atoms, which makes noise removal during preprocessing unnecessary. The measurement  is decomposed into
,
where  is the forward operator that projects from the dipole space to the measurement space,  is the dipole source amplitudes, and  is the error.  is further decomposed into a product of the Gabor atoms  and its weights .  is to be estimated by minimizing the mean squared error with two regularization terms: one with the mixed  norm to induce sparsity and coherence in space and another with the  norm to induce sparsity in time. The maximum a posteriori estimate is then used to calculate the estimated source amplitudes.
TF-MxNE would thus be able to coherently highlight the dipoles most relevant in the data, which could aid neuroscientists in determining which regions are involved in certain activities, as well as revealing pathways or connections by seeing the temporal relationships between regional activations. In the paper, M/EEG data were used from a subject who was randomly presented with auditory (a tone) and visual stimuli (a checkerboard image) on one side of the body. The authors ran two analyses by extracting for the left auditory and left visual events and ran the TF-MxNE algorithm on MEG data to find the dipoles activated in both these stimuli. We attempt to replicate the results for the left visual stimuli: finding the dipoles, the current amplitudes, and how well it explains the MEG data.
Replication
	The algorithm paper itself did not include any code, so we relied on the MNE-Python package [2], which was created by its authors a year later, to do the analysis. The data were downloaded via MNE-Python, which included the raw data file, window-averaged data, the forward solution, and the noise covariance matrix. The main preprocessing required was to crop for the time window around the stimulus (such that the data focus on the 100 milliseconds before and 400 milliseconds after). Before the data were fed into the TF-MxNE algorithm, the linear solver dSPM was first used in order to calculate weights that would be used to initialize the weights for the Gabor atom library in TF-MxNE.
Once these weights were obtained, TF-MxNE could be used after specifying some parameters. The algorithm first performs a whitening transformation, which projects the data onto new variables in the same space such that the variables are independent of each other. The forward solution is also modified to account for depth; a depth compensation bias would help emphasize the sources on the cortex since deeper sources would be attenuated significantly before reaching the sensor. The dipole sources are also adjusted for loose orientations, meaning that each dipole is represented with three orthonormal sources, but these sources are biased toward one direction based on the loose orientation parameter so the source is not completely free to rotate. After these adjustments, the algorithm iterates to minimize the error function for  with
,
where  and  are the spatial and temporal regularization parameters for the  and  norms, respectively. There is also an optional coefficient debiasing step afterward which serves to counter the heavy penalization on the source amplitudes when using  LASSO regularization.
Interestingly, the code provided for TF-MxNE from the software package differed from what the authors did in the algorithm paper; we thus performed two iterations of replication: one to follow the tutorial code, and one to imitate the original paper. The main difference appeared to be the regularization parameters used, but this was unable to fully account for the figures presented in the algorithm paper. For the fully replicated plots from the tutorial, see the Jupyter Notebook PDF. Figure 1 shows the replication attempt for the paper.
Additional Dataset Analysis and Results
	In the same vein as the visual stimulus examined in Section III, the additional dataset we used was the Statistical Parametric Mapping (SPM) Faces dataset [3]. For this experiment, subjects were shown either a normal face or a scrambled image that had a face-like shape. The evoked responses were recorded on M/EEG, structural MRI, and fMRI. An EOG was also used to track eye movement. We focused on one subject and the corresponding MEG data. The goal was to find which brain regions would be differentially activated for a normal compared to a scrambled face—the answer to this question could shed some light on whether there is a center for recognizing other humans. This could be an evolutionarily interesting result since learning to recognize whether a face belongs to a predator or another animal of the same species would have consequences for survival. The TF-MxNE algorithm could be used to find the most relevant brain regions that are activated upon seeing another human versus a visual control.
	For this dataset, no forward solution, noise covariance matrix, or averaged dataset was provided; all of these had to be calculated. The preprocessing pipeline was as follows: the raw data were (1) passed through a bandpass filter from 1 to 30 Hz to remove high-frequency noise and low-frequency brain states, (2) split into normal- and scrambled-face stimuli categories, (3) cropped for 200 milliseconds before and 600 milliseconds after the stimulus, and (4) decomposed with independent component analysis and the EOG data were used to remove artifacts related to eye movement. The cleaned responses were then averaged within their respective face categories for each MEG sensor, and the difference between the normal and scrambled face responses was calculated. This differential response at each sensor would be the input for TF-MxNE later. The noise covariance matrix was also found from the cleaned responses. Lastly, the forward solution was calculated by supplying the library of dipole sources for the brain model and the solution to the boundary element method that tells how the magnetic signals attenuate when traveling through the brain.
	With all of the required components to run an inverse solver, four solvers were used: dSPM (a least-norm solver that normalizes by noise) [4], sLORETA (a least-norm solver that takes into account source signal variance as well as noise) [5], γ-MAP (an empirical Bayesian solver that computes a hyperparameter γ that defines the covariance matrix for the source amplitudes) [6], and TF-MxNE. Figure 2 compares the results from the linear solvers dSPM and sLORETA, and Figure 3 compares the sparsity-inducing nonlinear solvers γ-MAP and TF-MxNE. Both dSPM and sLORETA are least-norm algorithms and do not iterate. The nonlinear solvers successfully converged, although the extra debiasing step for TF-MxNE did not converge within 1000 iterations. Compared to the original visual stimuli analysis in the TF-MxNE paper, we found noticeably more dipoles activated in the differential activity for faces. However, less of the variance in the MEG measurements could be explained in this new dataset.
Feedback Rebuttal
Feedback from the proposal was addressed in the resubmission. Ground truth was not available for any of the datasets, so multiple inverse solvers were implemented in Section IV in order to compare robustness.
Discussion and Conclusions
	While we were unable to fully replicate the original paper’s results, we were able to come close despite not having their original source code. It is unclear what causes the difference, whether it stems from preprocessing or the parameters given to TF-MxNE. For the new dataset, across all solvers, we find that portions of the visual cortex (V1) are differentially activated when a subject is presented with a human face compared to a scrambled face. This provides some evidence that there may be an area in V1 that serves to specifically recognize humans. However, a significant limitation was that the results from the inverse solvers varied dramatically, both spatially and in estimated source power. TF-MxNE did well to explain much of the measured data with ten dipoles, but these were rather scattered and not reflected in the other solvers. For example, only TF-MxNE found a strong source near the diencephalon, which was unexpected due to its deep location; despite this, it may be interesting to explore whether a human recognition center exists here. During testing, we also found that TF-MxNE was strongly sensitive to regularization parameters; a slight adjustment could change the dipoles found from ten to 26. To improve future analysis, a more robust, neurobiologically sound way of selecting regularization values would be needed; otherwise, there is no clear reason for why one set of parameters would be better than another.
Figures
A)                                                                             C)  







B)                                                                             D) 
Figure 1: Comparison of replicated figures (left) and paper figures (right). A) Dorsal view with the dipoles marked. Three dipoles are found in both solutions, largely in the visual cortex, but the replication finds a source on the lateral posterior side. The paper’s sources are more medial. B) Estimated source amplitudes. Both analyses find similar waveforms, although the weakest dipole is noticeably stronger in the replication. For both, about 50 ms elapses from the onset of the stimulus before two waves of activity. The peak activity occurs around 150 ms after the stimulus. C) Raw MEG data. As expected, both plots are the same. D) Accounted MEG measurements. Remarkably, despite having different predicted dipoles, the explained data appear rather similar. The main difference appears to be MEG sensors who record a negative amplitude during the second wave of activity; there are a few that appear to have a greater amplitude explained in the replication run.


A)                                                                                  B) 
Figure 2: Comparison of dSPM and sLORETA. A) Activity visualization, ventral view (left, dSPM; right, sLORETA) at 150 milliseconds after stimulus, with the two dipoles with the greatest peak amplitude marked. Interestingly, both techniques agree on the blue dipole for the left visual cortex, but there is a slight difference for the right hemisphere, with sLORETA predicting a more posterior source. The source power predicted by sLORETA is also much smaller than that of dSPM. B) Source amplitudes of two strongest dipoles (top, dSPM; bottom, sLORETA). Both linear solvers predict rather “ringy” amplitudes likely influenced by noise, but sLORETA has more oscillations, suggesting that it may be more susceptible to noise. Both predict peaks of activity around 100 to 150 ms, but dSPM appears to peak slightly after. The two dipoles from sLORETA also track much closer together than the dipoles from dSPM.


A)                                                   C)                                                       D) 






B) 
Figure 3: Comparison of γ-MAP and TF-MxNE. A) Ventral view with dipoles marked (left, γ-MAP; right, TF-MxNE). γ-MAP predicts 9 dipoles, mostly clustered in the right inferior occipital gyrus, although there is a strong dipole on the medial side of the left visual cortex (orange). TF-MxNE predicts 10 dipoles, scattered much more diversely. Interestingly, one of the strongest dipoles occurs near the diencephalon (red). B) Estimated source amplitudes. While both inverse solvers still produce somewhat “ringy” waveforms, the extent is noticeably smaller than the linear methods. Despite γ-MAP predicting 9 dipoles, only 2 of them appear obviously nonzero, with the orange dipole dominating. TF-MxNE predicts a much more even spread of power, albeit with no coherent pattern across dipoles. TF-MxNE also predicts significantly higher dipole amplitudes. C) Differential response MEG data. D) Explained differential MEG response data (top, γ-MAP; bottom, TF-MxNE). γ-MAP does a poor job accounting for any of the variance, while TF-MxNE performs a much better job, especially for 50 ms to 300 ms after the stimulus.


References
[1] 	Gramfort, A., Strohmeier, D., Haueisen, J., Hämäläinen, M. S., & Kowalski, M. (2013). Time-frequency mixed-norm estimates: sparse M/EEG imaging with non-stationary source activations. NeuroImage, 70, 410–422. https://doi.org/10.1016/j.neuroimage.2012.12.051
[2] Gramfort, A., Luessi, M., Larson, E., Engemann, D. A., Strohmeier, D., Brodbeck, C., Goj, R., Jas, M., Brooks, T., Parkkonen, L., & Hämäläinen, M. (2013). MEG and EEG data analysis with MNE-Python. Frontiers in neuroscience, 7, 267. https://doi.org/10.3389/fnins.2013.00267
[3] Henson, R. (2009). Statistical Parametric Mapping :: Multi-modal Face Dataset. Wellcome Centre for Human Neuroimaging. Retrieved March 19, 2023, from https://www.fil.ion.ucl.ac.uk/spm/data/mmfaces/
[4] Dale, A. M., Liu, A. K., Fischl, B. R., Buckner, R. L., Belliveau, J. W., Lewine, J. D., & Halgren, E. (2000). Dynamic statistical parametric mapping: combining fMRI and MEG for high-resolution imaging of cortical activity. Neuron, 26(1), 55–67. https://doi.org/10.1016/s0896-6273(00)81138-1
[5] Pascual-Marqui R. D. (2002). Standardized low-resolution brain electromagnetic tomography (sLORETA): technical details. Methods and findings in experimental and clinical pharmacology, 24 Suppl D, 5–12.
[6] Wipf, D., & Nagarajan, S. (2009). A unified Bayesian framework for MEG/EEG source imaging. NeuroImage, 44(3), 947–966. https://doi.org/10.1016/j.neuroimage.2008.02.059

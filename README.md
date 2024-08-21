# simuPACT

simuPACT is a simulation tool that follows the methodology of Oezgurt and Schinitzler (2011) for studying phase-amplitude coupling analysis.

The tool allows users to set various parameters.

Please refer to the left panel of the GUI shown below.

<img src="images/mainGUI.jpg" alt="The main GUI." width="1200" />

1. **LFO freqs**: This parameter specifies the frequency range of the low-frequency phase. It consists of a pair of numbers in increasing order, indicating the -6db cutoff point. The band-pass filter used is a Hamming window FIR (-53dB/oct). The transition bandwidth for the low-freq phase is determined by min([lower edge of the low-freq]*2 [bandwidth of the low-freq range]). Similarly, that for the high-freq phase is determined by min([lower edge of the high-freq band]*0.1 [bandwidth of the high-freq range]).
2. **HFO freqs**: This parameter defines the the frequency range of the high-frequency amplitude. It is important to note that the lower edge of the high-freq range must belarger than (mean of the low-freq range) + (bandwidth of the low-freq range) + (bandwidth of the high-freq range)
3. **SNRs to explore**: This parameter represents the signal-to-noise ratio (SNR) between the variance of the high-freq signal (obtained through band-pass filtering white noise) and the variance of the pink noise. SNR is defined only based on the variances of high-freq signals. [-4:2:2] is the same as [-4 -2 0 2].
4. **Window size**: This parameter determines the size of the window in seconds. In generally, a longer window leads to better detection performance.
5. **Num windows per SNR**: This parameter specifies the numbers of windows for each SNR level. It determines the number of iterations for each condition. Higher values result in better Gaussian modeling.
6. **Var ratio of without PAC**: This parameter expresses the variance ratio of the 'without PAC' signals in decibel. A value of 0 (dB) indicates an exact match in variance within the specified high-freq range between the PAC Present and PAC absent. The simulator will show that higher values produce poorer performance of the standard MI but not others, which is one of the key learning points.
- **Plot the first 2-s of signal and noise**: When this option is checked, a separate window will open to display line plots. The blue line represents the simulated signal and the red line represents the simulated pink noise. The 'PAC present' data are created by summing these signal and noise components. Note that the SNR specified applies to the entire data, and the selected window sample may show local deviation and may not necessarily follow the expected trend.
- **Calculate Empirically Normalized MI (slow)**: When this option is checked, normalized MI is calculated using empirically estimated signal variance. It uses surrogate data obtained by computing MI from amplitude-randomized data. Oezkurt and Schnitzler (2011) reported that this approach has a good property, which can be confirmed by this simulator. However, one obvious drawback is the computational cost. It is slow due to the need for iterations to calculate surrogate data. The number of iteration is fixed at 200, as Oezkurt and Schnitzler (2011) mentions '(typically around 200 times)'. Indeed, one can confirm with this simulator that as the variance ratio [dB] increases, the performance of the standard MI becomes worse, but normMI is unaffected. 
    
<img src="images/SNR_2x2.jpg" alt="The main GUI." width="600" />

- **Plot the first 2-s of with and without PAC**: When checked, this option opens a separate window to display another figure with line plots that compare PAC Present (yellow) and PAC Absent (purple). One can make the variance of the PAC Absent signal larger or smaller by changing the **Var ratio of without PAC** above in dB.

<img src="images/PresentVsAbsent.jpg" alt="The main GUI." width="600" />

## Example 1: All default parameters, just pressing the 'Start' button.
### 1-1 Showing modulation index (MI) results.
The modulation index (MI) is defined by Canolty et al. (2006). Oezkurt and Schnitzler (2011) also suggest 'direct estimate MI' which is supposed to be as good as robust GLM, but this simulator always gives the near-identical results with the standard MI, so I only show standard MI's result here.
<img src="images/MI.jpg" alt="The main GUI." width="600" />

### 1-2 Showing Kullback-Leibler (KL) Divergence (or Distance) results.
KLD is a entropy-based measure proposed by Tort et al. (2010). It is good at separating PAC Present from PAC Absent, but the variance of the metric within the condition is much larger than other measures.
<img src="images/KLD.jpg" alt="The main GUI." width="600" />

### 1-3 Showing robust general linear model (GLM) results.
This approach uses the low-freq phase as a regressor to explain the temporal pattern of high-freq amplitude power changes. 'Robust' means it ignores the intercept which represents data of non-interest i.e. differences in variances between the two signals compared. See the code used in this simulator. Note that beta_coef(3) is not included in the equation.

```matlab
X = [cos(currentWinPhase) sin(currentWinPhase) ones(length(currentWinZ),1)];
[beta_coef, redundant_term, error_trms] = regress(currentWinAmp, X);
robustGLM = 0.5 * sqrt ( (beta_coef(1)^2 + beta_coef(2)^2) / sum(currentWinAmp.*currentWinAmp)  );
```
<img src="images/robustGLM.jpg" alt="The main GUI." width="600" />

## Example 2: Introducing variance imbalance (+20 dB in PAC Absent) between PAC Present and PAC Absent.
Note that the purple line (PAC Absent) has a variance that is 10 times larger than the yellow line (PAC Present).
<img src="images/PresentVsAbsent_20dB.jpg" alt="The main GUI." width="600" />

### 2-1 Showing MI results--it fails!
<img src="images/MI_20dB.jpg" alt="The main GUI." width="600" />

### 2-2 Showing normalized MI results--it holds! 
<img src="images/normMI_20dB.jpg" alt="The main GUI." width="600" />

### 2-3 Showing KLD--it holds!
<img src="images/KLD_20dB.jpg" alt="The main GUI." width="600" />

### 2-4 Showing robust GLM--it holds!
<img src="images/robustGLM_20dB.jpg" alt="The main GUI." width="600" />

## Conclusion and comments
1.The overall winner is robustGLM. It is also fast, unlike normalized MI.
2.Internally, all three variants of GLM studied by Oezkurt and Schnitzler (2011) are calculated. However, unless the pair of input vectors, PAC Present and PAC Absent, are exactly normalized to the same variance, standard GLM or equivalent-to-GLM keep failing. Even when PAC Present and PAC Absent are normalized as a whole data, the subsampled windows have local deviation from the overall variance, thus these GLM variants fail. When the inputs time series of PAC Present and Absent are exactly normalized, the results are identical to that of robust GLM. From here, we can learn what it means by excluding the intercept term i.e. beta_coeff(3) from the equasion.
